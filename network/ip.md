

###子网号 VS 网络地址 VS 主机号

子网划分并没有节约 IP 地址，实际导致可分配的 IP 地址数目减少。

证明：比如一个 C 类地址，不进行子网划分，实际可分配 IP 地址为 254 个。

现进行子网划分，假设借用 2 位主机号作为子网号，那么现在产生的子网为 01 和 10（全0全1子网号去掉），
每个子网的主机号为 6 位，则每个子网可分配的 IP 地址为 2 的 6 次方剪掉 2，即 62 台，
那么两个子网可分配的 IP 共 62*2=124 个， 那么减少的 IP 数目为：254-124=130个。

从上数据可以看出，减少了约一半的 IP 地址。既然这么浪费 IP 地址，为何我们还要使用子网划分呢？
我个人认为，这是利用子网来方便管理网络的一种措施。

很容易看出，减少这么多 IP 地址的主要原因是子网号为 00（全0）和 11（全1）的两个子网去掉了，
那为何要去掉“全0全1”的子网号呢？

不应该使用全 0 全 1 子网这个规定是源于 RFC950 标准，但后来 RFC950 在 RFC1878 中被废止了。

看看 RFC950 提到的原因：

假设我们有一个网络：192.168.0.0/24，我们现在需要两个子网，那么按照 RFC950，应该使用 /26 而不是 /25，
得到两个可以使用的子网 192.168.0.64 和 192.168.0.128

* 对于192.168.0.0/24，网络地址是192.168.0.0，广播地址是192.168.0.255
* 对于192.168.0.0/26，网络地址是192.168.0.0，广播地址是192.168.0.63
* 对于192.168.0.64/26，网络地址是192.168.0.64，广播地址是192.168.0.127
* 对于192.168.0.128/26，网络地址是192.168.0.128，广播地址是192.168.0.191
* 对于192.168.0.192/26，网络地址是192.168.0.192，广播地址是192.168.0.255

你可以看出来，对于第一个子网，网络地址和主网络的网络地址是重叠的，对于最后一个子网，广播地址和主网络
的广播地址也是重叠的。这样的重叠将导致极大的混乱。比如，一个发往 192.168.0.255 的广播是发给主网络的
还是子网的？这就是为什么在当时不建议使用全 0 和全 1 子网。

然而，人们认识到子网划分的 IP 地址浪费严重，后来 IETF 就研究出了其他一些技术，比如可变长子网掩码 VLSM，
该技术是在子网上进一步划分子网，可提高 IP 地址资源的利用率；后来在此基础上研究出了无类别域间路由CIDR，
即消除了传统的 A/B/C 等分类以及划分子网，才是采用网络前缀和主机号的方式来分配 IP 地址，这使得 IP 地址
的利用率更好。这两者的具体技术暂时不阐述。

就目前来说，现在可以使用全 0 和全 1 子网。但我们现在学习时，还强调子网划分时要去掉全 0 全 1，
这是何道理呢？我个人认为：

* 目前有些网络建设较早，设备也不更新，老设备可能不支持 CIDR，那么也就不支持全 0 全 1 的子网了。
* 我们建企业网（单位网络）时，一般是使用私有地址来分配内部主机，小企业使用 C 类的 192.168.0.0 网络，
中型企业使用172.16.0.0（私有部分）网络，如果还不够用，还有 10.0.0.0 网络。





 最近在和IP报文的优先级打交道，所以在网络上就乱搜一通，了解了一下这个东西，发现关于它的来龙去脉还有点多，像IP Precedence，TOS，DSCP，COS，QOS这几个玩意儿搞在一起，一时间会让你有点一头雾水，尤其是刚接触网络的。我在这里也刚接触一点点，所以就写下一点关于它们的概念这些东西，想了解的朋友可以看看，以后碰到的时候也好有个概念在脑海里，不至于说几次就会云里雾里的。当然我也只了解了一点点像DSCP也还没有怎么去了解，所以很多都说得不完全或甚至错误，还请谅解哟。

在TCP/IP网络中，IP协议称之为Internet Protocol英特网协议，IP协议是TCP/IP协议簇（协议簇我的理解就是“大家庭”哈）里面最重要的协议之一（之一是因为还有个TCP协议），因为正是由于有各个不同的IP网络（即不同IP网段的子网）组成了整个Internet，当然Internet也不仅仅是由IP网络构成哈，它还有一些其它类型的网络，这里不做研究。

我现在要说的就是IP这个协议上的一个“字段”，关于“字段”呢，你可以想象一下一张二维表格，如下：

	姓名 	学号 	性别	年龄	专业

	张三	001	男	20	计算机

如上面所示，这张表格里张三就是表格的一条记录，这条记录记载了一个学生的四个属性，而属性就可以称之为一个“字段”，所有的“字段”就组成了这张表的基本结构。同理：

 			源IP	目的IP	TOS	……

	IP报文一	1.1.1.1	2.2.2.2	0001	……

应该能明白“字段”了吧，反正就是IP报文的一个属性就是了。好，切入正题，首先呢要谈IP Precedence、TOS和DSCP首先就要从IP协议的第一个标准文件RFC 791说起，好像在写小说一般?

RFC 791是IETF即Internet工程任务组（Internet Engineering Task Force）于1981年9月制定的一个关于IP协议的一个正式文件。IETF呢就是一群互联网领域的顶尖高手组成的一个专门制定互联网各种协议标准的组织。这个RFC 791里面就对IP协议的运行机制，报文格式做了详细的说明和规范，其中就包括了Type Of Service字段，简称TOS字段。

如下所示，TOS字段一共8个Bit（Bit就是一个二进制位），即8位。IETF规定了低位的0，1，2三位就用于IP Precedence即IP优先级，但是这里好像这里没做其它说明。中间的3，4，5用于Type Of Service，IETF规定这个字段用中间的三位，即3，4，5位用来表示这个IP报文期望得到的一种高质量的传输服务，也就是说中间这三位表明了这个IP包的服务类型（Type Of Service）。它用了三个参数来实现高质量服务：低延迟、高吞吐量、高可靠性。可以看到Bit 3第三位为0时代表正常延迟，为1为低延迟；第四位为0代表正常吞吐量，为1为高吞吐量；第五位为0代表正常可靠性，为1代表高可靠性。最后的Bit 6，7位保留不使用（为了将来使用而保留）：

      Bits 0-2:  Precedence.

      Bit    3:  0 = Normal Delay,      1 = Low Delay.

      Bits   4:  0 = Normal Throughput, 1 = High Throughput.

      Bits   5:  0 = Normal Relibility, 1 = High Relibility.

      Bit  6-7:  Reserved for Future Use.

 

         0     1     2     3     4     5     6     7

      +-----+-----+-----+-----+-----+-----+-----+-----+

      |                 |     |     |     |     |     |

      |   PRECEDENCE    |  D  |  T  |  R  |  0  |  0  |

      |                 |     |     |     |     |     |

      +-----+-----+-----+-----+-----+-----+-----+-----+

 

        Precedence

 

          111 - Network Control

          110 - Internetwork Control

          101 - CRITIC/ECP

          100 - Flash Override

          011 - Flash

          010 - Immediate

          001 - Priority

          000 – Routine

可以看到关于IP Precedence在这里只是做出了简单的规定，用三个Bit置不同位来实现不同的优先级。而关于中间的3，4，5位TOS，还做了一些其它的说明和规定。

但在1992年7月，IETF又制定了RFC 1349（好像中间关于TOS字段也有过修改说明），这个标准重新规定了有关TOS字段的意义：

 

                0     1     2     3     4     5     6     7

             +-----+-----+-----+-----+-----+-----+-----+-----+

             |                 |                       |     |

             |   PRECEDENCE |          TOS          | MBZ |

             |                 |                       |     |

             +-----+-----+-----+-----+-----+-----+-----+-----+

英文不好，在RFC 1349里用金山词霸取了部分重点词汇的含义，大概知道一点，它的大致意思就是说在RFC 791里用Bit 0，1，2表示IP优先级，用Bit 3，4，5实现IP报文渴望得到的服务即TOS，最后两位保留使用，而在RFC 1122中又规定了将Bit 6，7划入了TOS部分，而这里RFC 1349将重新把Bit 7划为保留位，中间的Bit 3,4,5,6四位来表明TOS（看来IETF也有出尔反尔啊，也许是网络变化太快，要随机应变）。当然Bit 0,1,2三位仍然为IP优先级位，只是RFC 1349也不做多余说明，基本没什么用啊。在W.Richard Stevens（他是个顶尖的高手，著作有《TCP/IP三卷》，《UNIX环境高级编程》可以去搜一搜）的TCP/IP详解（协议）书里也提到了IP Precedence这三位基本没有用，不过写书时间也很久了哈。

现在用了Bit 3,4,5,6四位做TOS设置，那各Bit位的含义是什么呢？以下是RFC 1349中COPY下的：

                    1000   --   minimize delay

                    0100   --   maximize throughput

                    0010   --   maximize reliability

                    0001   --   minimize monetary cost

                    0000   --   normal service

如上所示，从低位到高位依次置1所代表的含义是：低延迟，高吞吐量，高可靠性，低花费（开销），正常服务。要注意的是这四个bit只能同时置1个位，不能同时置两个或多个位为1。

以上就是有关IP Precedence和TOS的我所了解的来龙去脉，总的来说呢有了这些TOS字段，那么含有这些字段信息的IP报文在送到某路由器上时，设备尊循标准就会按标准给这个IP报文一定的优先杈，给它什么优先权利就看TOS字段的设置了。另外IP Precedence好像没什么用一样，不过我接触到的设备里也有会对IP报文打上IP Precedence标签的路由器，不过会不会有什么优先可能还要看其它收到这个IP报文的设备有没有支持到这个特性。

最后说下DSCP，这个呢全称叫做差分服务代码点（Differentiated Services Code Point），它呢是IETF于1998年12月发布了Diff-Serv（Differentiated Service）标准，由RFC 2474定义。看RFC的编号就明白要比RFC 1349制定得晚，数字越大越晚嘛。这个标准完全重新定义了TOS这整个8 Bit，它使用以前的IP优先级和中间的TOS的三位，一共6位从0到63。它呢也是不同于TOS的另一个标准，不过目的也是为QOS(Quality Of Service)提供一个分类服务的标准。

简单说下QOS哈，这个概念是个很广，很宽泛的概念，以前我看到网络方面的书籍都提到了它，但就是不明白这个是干嘛用的，看一遍就蒙一遍，始终不明白它有什么用。这些前辈们老是弄些抽象的名词，让人理解起来很头疼。其实以我接触的一点点经验来说呢，QOS你就想在网络上有那么多种协议，那么多不同类型的数据它们总得有一个“等级的划分”吧，划分什么等级呢？当然就是为了区分在网络上传输时得到的“待遇”啊，不同级别的数据得到不同的待遇，这个好理解吧。像公交车上老，弱，病，残，孕就要得到优先照顾，其它人都要给这类人群让坐。这里对数据包根据一定标记划分不同等级而得到不同优先传输条件一系列活动抽象就叫QOS。我们对QOS的实现可以依据不同的标准，可根据User Priority、IP Precedence、TOS、DSCP等等。使用QOS也是为了对网络的可靠性、稳定性这些做个保障，让网络能更好的服务于大众。

好了，再拉回来到DSCP，关于这个不同于TOS标准，它对TOS那8 Bit做了许多的规定划分，我还没有仔细研究，RFC 2474也只草草看了一眼，所以DSCP暂时不做过多研究。

哦，差点忘记还有个COS（Class Of Service），等与DSCP一同研究研究吧，这次我只写IP Precedence与Type Of Service、QOS。

###

IP 数据报是通过封装为物理帧来传输的. 由于因特网是通过各种不同物理网络技术互连起来的, 在因特网的不同部分,
物理帧的大小(最大传输单元 MTU)可能各不相同. 为了最大程度的利用物理网络的能力, IP 模块以所在的物理网络的
MTU 做为依据，来确定 IP 数据报的大小. 当 IP 数据报在两个不同 MTU 的网络之间传输时, 就可能出现 IP 数据报
的分段与重组操作.

在 IP 头中控制分段和重组的 IP 头域有三个: 标识域, 标志域, 分段偏移域. 标识是源主机赋予 IP 数据报的标识符.
目的主机根据标识域来判断收到的 IP 数据报分段属于哪一个数据报, 以进行 IP 数据报重组. 标志域中的 DF 位标识
该 IP 数据报是否允许分段. 当需要对 IP 数据报进行分段时, 如果 DF 位置 1, 网关将会抛弃该 IP 数据报, 并向源
主机发送出错信息. 标志域中的 MF 位标识该 IP 数据报分段是否是最后一个分段. 分段偏移域记录了该 IP 数据报分
段在原 IP 数据报中的偏移量. 偏移量是 8 字节的整数倍. 分段偏移域被用来确定该 IP 数据报分段在 IP 数据报重组
时的顺序.

IP数据报在被传输过程中，一旦被分段，各段就作为独立的IP数据报进行传输，在到达目的主机之前有可能会被再次或多次分段。但是IP数据报分段的重组都只在目的主机进行。

###IP 分组

4.2 IP数据报的分段与重组
IP数据报是通过封装为物理帧来传输的。由于因特网是通过各种不 同物理网络技术互连起来的，在因特网的不同部分，物理帧的大小（最大传输单元MTU）可能各不相同。为了最大程度的利用物理网络的能力，IP模块以所在的 物理网络的MTU做为依据，来确定IP数据报的大小。当IP数据报在两个不同MTU的网络之间传输时，就可能出现IP数据报的分段与重组操作。

在IP头中控制分段和重组的IP头域有三个：标识域、标志域、分段偏移域。标识是源主机赋予IP数据报的标识符。目的主机根据标识域来判断收到的 IP数据报分段属于哪一个数据报，以进行IP数据报重组。标志域中的DF位标识该IP数据报是否允许分段。当需要对IP数据报进行分段时，如果DF位置 1，网关将会抛弃该IP数据报，并向源主机发送出错信息。标志域中的MF位标识该IP数据报分段是否是最后一个分段。分段偏移域记录了该IP数据报分段在 原IP数据报中的偏移量。偏移量是8字节的整数倍。分段偏移域被用来确定该IP数据报分段在IP数据报重组时的顺序。

IP数据报在被传输过程中，一旦被分段，各段就作为独立的IP数据报进行传输，在到达目的主机之前有可能会被再次或多次分段。但是IP数据报分段的重组都只在目的主机进行。

4.3 IP对输入数据报的处理
IP对输入数据报的处理分为两种，一种是主机对数据报的处理，一种是网关对数据报的处理。

当IP数据报到达主机时，如果IP数据报的目的地址与主机地址匹配，IP接收该数据报并将它传给高级协议软件处理；否则抛弃该IP数据报。

网关则不同，当IP数据报到达网关IP层后，网关首先判断本机是否是数据报到达的目的主机。如果是，网关将接收到的IP数据报上传给高级协议软件处理。如果不是，网关将对接收到的IP数据报进行寻径，并随后将其转发出去。

4.4 IP对输出数据报的处理
IP对输出数据报的处理也分为两种，一种是主机对数据报的处理，一种是网关对数据报的处理。

对于网关来说，IP接收到IP数据报后，经过寻径，找到该IP数据报的传输路径。该路径实际上是全路径中的下一个网关的IP地址。然后，该网关将该 IP数据报和寻径到的下一个网关的地址交给网络接口软件。网络接口软件收到IP数据报和下一个网关地址后，首先调用ARP完成下一个网关IP地址到物理地 址的映射，然后将IP数据报封装成帧，最后由子网完成数据报的物理传输。


6.2 接收IP数据报
当网络设 备从网络上接收到数据报时，它必须将接收到的数据转换为sk_buff数据结构，然后将该结构添加到backlog队列中排队。当backlog队列变得 很大时，接收到的sk_buff数据将会被丢弃。当新的sk_buff添加到backlog队列时，网络底层程序将被标志为就绪状态，从而可以让调度程序 调度底层程序进行处理。

调度程序最终会运行网络的底层处理程序。这时，网络底层处理程序将处理任何等待传输的数据报，但在这之前，底层处理程序首先会处理sk_buff结构的backlog队列。底层处理程序必须确定将接收到的数据报传递到哪个协议层。

在Linux进行网络层的初始化时，每个协议要在ptype_all链表或ptype_base哈希表中添加packet_type数据结构以进行 注册。Packet_type数据结构包含协议类型、指向网络设备的指针、指向协议的接收数据处理例程的指针等。Ptype_base是一个哈希表，其哈 希函数以协议标识符为参数，内核通常利用该哈希表判断应当接受传入的网络数据报的协议。通过检查ptype_all链表和ptype_base哈希表，网 络底层处理程序会复制新的sk_buff，最终，sk_buff会传递到一个或多个目标协议的处理例程。

6.3 发送IP 数据报
网络处理代码必须建立sk_buff来包含要传输的数据，并且在协议层之间传递数据时，需要添加不同的协议头和协议尾。

首先，IP协议需要决定要使用的网络设备，网络设备的选择依赖于数据报的最佳路由。对于只利用调制解调器和PPP协议连接的计算机来说，路由的选择比较容易，但是对于连接到以太网的计算机来说，路由的选择是比较复杂的。

对每个要传输的IP数据报，IP利用路由表解析目标IP地址的路由。对每个可从路由表中找到路由的目标IP地址，路由表返回一个rtable数据结 构描述可使用的路由。这包括要使用的源地址、网络设备的device数据结构的地址以及预先建立的硬件头信息。该硬件头信息和网络设备相关，包含了源和目 标的物理地址以及其它的介质信息。

6.4 数据报的分段与重组
当传 输IP数据报时，IP从IP路由表中找到发送该IP数据报的网络设备，网络设备对应的device数据结构中包含由一个mtu字段，该字段描述最大的传输 单元。如果设备的mtu小于等待发送的IP数据报的大小，就需要将该IP数据报划分为小的片断。每个片断由一个sk_buff代表，其中的IP头标记为数 据报片断，以及该片断在IP数据报中的偏移。最后的数据报被标志为最后的IP片断。如果分段过程中IP不能分配sk_buff，则传输失败。

IP片断的接收较片断的发送更加复杂一些，因为IP片断可能以任意的顺序接收到，而在重组之前，必须接受到所有的片断。每次接收到IP数据报时， IP 要检查是否是一个分段数据报。当第一次接收到分段的消息时，IP建立一个新的ipq数据结构，并将它链接到由等待重组的IP片断形成的ipqueue链表 中。随着其他IP片断的接收，IP找到正确的ipq数据结构，同时建立新的ipfrag数据结构描述该片断。每个ipq数据结构中包含有其源和目标IP地 址、高层协议的标识符以及该IP帧的标识符，从而唯一描述了一个分段的IP接收帧。当所有的片断接收到之后，它们被组合成单一的sk_buff并传递到上 一级协议层处理。如果定时器在所有的片断到达之前到期，ipq数据结构和ipfrag被丢弃，并假定消息已经在传输中丢失，这时，高层协议需要请求源主机 重新发送丢失的信息。


clone from [here](http://www.wildpackets.com/resources/compendium/tcp_ip/ip_tos)

The 'Type Of Service' Byte In The IP Header

RFC 791, (Internet Protocol - DARPA Internet Program Protocol Specification, September 1981), defined a field within the IP header called the Type Of Service (TOS) byte. This Byte is used to specify the quality of service desired for the datagram and is an amalgamation of several factors. These factors include several fields such as Precedence, Speed, Throughput and Reliability as identified below. In normal conversations you would not use any special alternatives, so the Type of Service byte typically would be set to zero. However, with the advent of Internet multimedia transmission and the emergence of protocols such as Session Initiation Protocol (SIP), this field is coming into use.

(A general note regarding the use of the IP TOS Byte is that in the course of normal network operations, not including Internet Multimedia, if you ever see the Type Of Service byte set to anything other than zero you should find out who's doing it, and why.)

The IP Type of Service Byte:

Bits 0-2: Precedence.
Bit 3: Delay (0 = Normal Delay, 1 = Low Delay)
Bit 4: Throughput (0 = Normal Throughput, 1 = High Throughput)
Bit 5: Reliability (0 = Normal Reliability, 1 = High Reliability)
Bits 6-7: Reserved for Future Use.
0 	1 	2 	3 	4 	5 	6 	7
PRECEDENCE 	D 	T 	R 	0 	0

The three bit Precedence field is further defined as follows:

111 - Network Control
110 - Internetwork Control
101 - CRITIC/ECP
100 - Flash Override
011 - Flash
010 - Immediate
001 - Priority
000 - Routine

So what exactly is the Precedence field, and what is the difference between these various classifications such as Priority and Immediate? Recall that ARPAnet originated under the authority of the DOD, and that a number of the early performance parameters were patterned after existing DOD communications models. It is from these already established models of communications that the concept of the Precedence Field emerged. The answer for the priorities (Routine - CRITIC/ECP) is defined in Department of Defense (DOD) communications message handling directives and in RFC 791. The remaining two classifications (Internetwork Control and Network Control) are defined in RFC 791.

The following is a synopsis of these classifications as specified by DOD Communications Directives and RFC 791:

A. DOD DD173 Precedence/Priority Filed Explanations (Lowest-Highest):

    Routine: (R) "…is used for all messages that justify transmission by electrical means unless the message delivery is of sufficient urgency to require higher precedence."
    Priority: (P) "…is used for all messages that require expeditious action by the addressee(s) and/or furnish essential information for the conduct of ongoing operations."
    Immediate (O) "…is reserved for messages relating to situations that gravely affect the security of National/Allied forces or populace."
    Flash (Z) "…is reserved for initial enemy contact messages or operational combat messages of extreme urgency."
    Flash Override (X) "… is reserved for messages relating to the outbreak of hostilities and/or detonation of nuclear devices."
    CRITIC/ECP "…stands for "Critical and Emergency Call Processing" and should only be used for authorized emergency communications, for example in the United States Government Emergency Telecommunications Service (GETS), the United Kingdom Government Telephone Preference Scheme (GTPS) and similar government emergency preparedness or reactionary implementations elsewhere." 

B. RFC 971 Specific Classifications:

According to RFC 791, the functions of the classifications: Network Control and Internetwork Control are defined as follows:

1. Network Control "…is intended to be used within a network only. The actual use and control of that designation is up to each network."

2. Internetwork Control "…is intended for use by gateway control originators only."

RFC 791 further addresses the use of these two classifications by noting that if used, "… it is the responsibility of that network to control the access to, and use of, those precedence designations."

It should now be apparent why the Precedence field is no longer used in traditional networking applications. Therefore, if you ever see these bits in use, you should find out who's doing it, and why. The implication of using the priority bits is completely vendor dependent. Consider the following example of IP TOS in a contemporary network environment:

Let's assume that you have a router that shows three different routes between Honolulu (on the island of Oahu) and Hilo (on the big island of Hawaii). You have a leased copper T1 line (which runs in an undersea cable), a fiber optic link (also in the cable), and a T1 satellite link. You decide that there is the least data loss in the fiber optic cable and you select it to be your link of greatest reliability. You know that most traffic is going across this link (because you have load-balancing routers) and that the satellite link and the leased T1 are almost unused. You decide that the leased T1 will be your link of least delay and that the satellite link will be your link of maximum throughput. These decisions are totally arbitrary; let's hope you made good decisions.

Let's further assume that your routers support the speed, throughput, and reliability types of service. You configure the Router and identify the links according to your decisions. Let's now assume that your workstation has the need, ability, and application software that will try to utilize different types of service for different activities. Perhaps your file transfer utility will select the link of greatest bandwidth while your accounting application selects the link of greatest reliability. Meanwhile, your terminal access software selects the link of highest speed.

Now consider the following criteria: Does your router support alternative types of service? Do your workstations support these options? A failure of any device in the datagram path will result in the traffic not being properly routed to its destination. This is why you need to determine who is responsible if you ever see the Type Of Service byte set to anything other than zero.

If a misconfiguration exists or an error occurs that relates to the IP Type Of Service, a protocol called ICMP (Internet Control Message Protocol) will report the error back to the station sending the failed frame. RFC 1349 discusses the relationship between IP Type Of Service and ICMP messages. For more information on ICMP or to read the text of RFC 1349, please refer to the ICMP Section.

Future Employment of the TOS/Precedence Byte:

Until fairly recently, it was a safe assumption that the IP TOS Byte was essentially obsolete and would be ignored in day-to-day networking situations. However, with the emergence of Internet multimedia transmission and the emergence of protocols such as Session Initiation Protocol (SIP), this field is returning to use.

RFC 2543 (SIP: Session Initiation Protocol, March 1999) specifies a protocol known as Session Initiation Protocol (SIP). This protocol is an application-layer control protocol used for creating, modifying and terminating sessions with one or more participants. Some examples of such activities include Internet multimedia conferences, Internet telephone calls and multimedia distribution. SIP is intended to support communications using Multicast, a mesh of Unicast relations, or a combination of both.

Contained within this protocol specification is the reliance upon the traditional RFC 791 Precedence classifications as identified by "Priority" in the following extract:

"…The resource value is formatted as "namespace"".""Priority value". The namespace and priority value are assigned by IANA (see IANA Considerations). An initial namespace, "dsn" (Defense Switched Network), contains the priority values, "critic-ecp", "flash-override", "flash", "immediate", "priority", "routine", where "flash-override" is the highest priority and "routine" is the lowest.

As a response header, the value indicates the actual priority selected by the recipient. This priority value may be lower or higher than the request header value. If the header field is missing, the SIP request is treated as if it had the Resource-Priority value of "routine"..." For further information regarding the specifications of SIP, consult RFC 2543 (SIP: Session Initiation Protocol, March 1999)



[more](http://www.study-area.org/network/network_ip_addr.htm)

