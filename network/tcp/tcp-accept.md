
下面以一个不太精确却通俗易懂的图来说明之:

![tcp accept 流程图](tcp_accept.jpg)

研究过 backlog 含义的朋友都很容易理解上图. 这两个队列是内核实现的, 当服务器绑定, 监听了某个端口后,
这个端口的 SYN 队列和 ACCEPT 队列就建立好了. 客户端使用 connect 向服务器发起 TCP 连接, 当客户端的
SYN 包到达了服务器后, 内核会把这一信息放到 SYN 队列(即未完成握手队列)中, 同时回一个 SYN+ACK 包给客
户端. 一段时间后, 客户端再次发来了针对服务器 SYN 包的 ACK 网络分组时, 内核会把连接从 SYN 队列中取出,
再放到 ACCEPT 队列(即已完成握手队列)中. 而服务器调用 accept 时, 其实就是直接从 ACCEPT 队列中取出已
经建立成功的连接套接字而已.

现有我们可以来讨论应用层组件: 为何有的应用服务器进程中, 会单独使用 1 个线程，只调用 accept 方法来建
立连接, 例如 tomcat; 有的应用服务器进程中, 却用 1 个线程做所有的事, 包括 accept 获取新连接.

原因在于: 首先, SYN 队列和 ACCEPT 队列都不是无限长度的, 它们的长度限制与调用 listen 监听某个地址端口
时传递的 backlog 参数有关. 既然队列长度是一个值, 那么, 队列会满吗? 当然会, 如果上图中第 1 步执行的速
度大于第 2 步执行的速度, SYN 队列就会不断增大直到队列满; 如果第 2 步执行的速度远大于第 3 步执行的速度,
ACCEPT 队列同样会达到上限. 第1,2步不是应用程序可控的, 但第3步却是应用程序的行为, 假设进程中调用 accept
获取新连接的代码段长期得不到执行, 例如获取不到锁, IO阻塞等。

那么, 这两个队列满了后, 新的请求到达了又将发生什么?

若 SYN 队列满, 则会直接丢弃请求, 即新的 SYN 网络分组会被丢弃; 如果 ACCEPT 队列满, 则不会导致放弃连接,
也不会把连接从 SYN 列队中移出, 这会加剧 SYN 队列的增长. 所以, 对应用服务器来说, 如果 ACCEPT 队列中有
已经建立好的 TCP 连接, 却没有及时的把它取出来, 这样, 一旦导致两个队列满了后, 就会使客户端不能再建立新
连接, 引发严重问题。

所以, 如 TOMCAT 等服务器会使用独立的线程, 只做 accept 获取连接这一件事, 以防止不能及时的去 accept 获取连接.

那么, 为什么如 Nginx 等一些服务器, 在一个线程内做 accept 的同时, 还会做其他 IO 等操作呢?

这里就带出阻塞和非阻塞的概念. 应用程序可以把 listen 时设置的套接字设为非阻塞模式(默认为阻塞模式), 这两
种模式会导致 accept 方法有不同的行为. 对阻塞套接字, accept行为如下图:

![阻塞 accept](block_accept.jpg)

这幅图中可以看到, 阻塞套接字上使用 accept, 第一个阶段是等待 ACCEPT 队列不为空的阶段, 它耗时不定, 由客户端
是否向自己发起了 TCP 请求而定, 可能会耗时很长.

对非阻塞套接字, accept 会有两种返回, 如下图:

![非阻塞 accept](unblock_accept.jpg)

非阻塞套接字上的 accept, 不存在等待 ACCEPT 队列不为空的阶段, 它要么返回成功并拿到建立好的连接, 要么返回失败.

所以, 企业级的服务器进程中, 若某一线程既使用 accept 获取新连接, 又继续在这个连接上读, 写字符流, 那么, 这个连
接对应的套接字通常要设为非阻塞. 原因如上图, 调用 accept 时不会长期占用所属线程的 CPU 时间片, 使得线程能够及时
的做其他工作.