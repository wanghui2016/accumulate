本文，我将向大家解释什么是CIFS和SMB，它们如何工作和这些协议里一些共同的不安全隐患。
本文将会成为学习微软网络知识的一个有用资源。SMB协议是一个在LAN中非常通用的协议了。我为
大家提供了一个关于如何操作SMB例子的源代码。

你将会学习到在所有SMB密码都是加密的情况下，如何使用ARP毒药来获得清晰的SMB密码(不需要
粗鲁与暴力)。你将会理解SMB与NETBIOS之间的关系。你同样会学到什么是微软远程管理协议(RAP)，
以及如何使用它来扫描远程SMB服务器上的共享资源。

本文提供的程序和资料都只有一个教育性的目的。你将用它来做的任何事情与我无关。

  
--[ 2什么是SMB/CIFS ？

依照微软的意思，CIFS是为客户系统在网络上向服务器请求文件和打印服务的开放跨平台的运行
机制。它是建立在广泛应用于个人电脑和工作站等操作系统的标准服务器消息块(SMB)协议。

实际上，SMB是一个通过网络在共享文件，设备，命名管道和邮槽之间操作数据的协议。CIFS是
SMB的一个公共版本。

SMB客户端的可用系统：

for Microsoft :
Windows 95, Windows for workgroups 3.x, Windows NT,2000 and XP
  
for Linux :
Smblient from Samba，Smbfs for Linux
  
SMB服务器：

Samba
Microsoft Windows for Workgroups 3.x
Microsoft Windows 95
Microsoft Windows NT
The PATHWORKS family of servers from Digital
LAN Manager for OS/2,SCO,etc
VisionFS from SCO
TotalNET Advanced Server from Syntax
Advanced Serverfor UNIX from AT&T (NCR?)
LAN Server for OS/2 from IBM.
  
  
--[ 3 会话的建立

注意：SMB协议已经被发展成为可以运行于DOS操作系统，因此字节顺序将会和网络顺序相反。

SMB可以运行在TCP/IP，NetBEUI，DECnet和IPX/SPX协议之上。如果SMB执行于TCP/IP，DECnet
或则是NetBEUI之上，那么NETBIOS名字必须被使用。

我将会在第六章向大家介绍什么是NETBIOS。但是现在，你必须知道NETBIOS名字用来在网络上鉴
别一台计算机。

SMB技术的发展开始于八十年代，出现过很多版本的SMB协议。但是最通用的(在Windows 98,
Windows NT,Windows 2000 and XP)是NT LM 0.12版本。本文是基于NT LM 0.12版本之上的。

你必须知道一个SMB域名是用来鉴别一个SMB服务器上的一组资源的(用户，打印机，文件……)。

那么一个客户端是如何与一个服务器建立SMB会话的呢？

让我们假设一个这样的环境：一个客户端希望访问一台服务器上的特定资源。

1 - 开始于客户端向服务器请求一个NETBIOS会话。客户端发送它的已编码的NETBIOS名字到SMB
服务器(它们在139端口监听连接请求)。服务器接收到NETBIOS名字后回复一个NETBIOS会话数据报给
有效的会话连接。客户端在建立了连接之后才能进入访问。

2 - 客户端发送一个SMB negprot请求数据报(negprot是磋商协议“negotiate protocol”的
简写)。客户端列出了它所支持的所有SMB协议版本。

3 - 通过磋商之后，客户端进程向服务器发起一个用户或共享的认证。(下一章中我们将会看到
共享认证和用户认证之间的不同)。

这个过程是通过发送SesssetupX(SesssetupX是会话建立和X“Session setup and X”的简称)
请求数据报实现的。客户端发送一对登录名/密码或一个简单密码到服务器，然后服务器通过发送一个
SesssetupX应答数据报来允许或拒绝本次连接。

4 - 好了，在客户端完成了磋商和认证之后，它会发送一个TconX数据报并列出它想访问的特定
网络资源的名称，之后服务器会发送一个TconX应答数据报以表示此次连接是否接受或拒绝。

  
netbios session request
(netbios name)
[client] ---------------------------> [server]
1)
netbios session granted
[client] <-------------------------- [server]
  
  
SMB negprot request
[client] ---------------------------> [server]
2)
SMB negprot reply
[client] <-------------------------- [server]
  
  
SMB sesssetupX request
[client] ---------------------------> [server]
3)
SMB sesssetupX reply
[client] <-------------------------- [server]
  
  
SMB TconX request
[client] ---------------------------> [server]
4)
SMB TconX reply
[client] <-------------------------- [server]
  
  
关于每个数据报的详细描述在第六章里有详细的讲解。

  
--[ 4 - SMB的安全等级

在SMB中有两种安全模式：

第一种是“共享”级的安全模式。这种安全模式需要一个访问网络上共享资源的密码。用户通过
这个正确的密码来访问网络资源(IPC，磁盘，打印机)。他可以是网络上任何一个知道共享资源服务
器名字的用户。

第二种是“用户”级的安全模式。这是在第一种模式上的增强版。它坚持使用一对登录名/密码来
访问共享资源。所以如果一个用户想访问这种类型的共享资源，就必须同时提供登录名和密码。这种
模式对了解用户如何使用共享资源是很有帮助的。

--[ 5 - 密码

在SMB协议中，如果你想进行一次在服务器上的请求认证，你的密码可以以原码或加密后的形式发
送到服务器端。如果服务器支持加密属性，客户端必须发送一个应答信号。在negprot应答数据报中，
服务器会给客户端发送一个密钥。然后，客户端将密码加密并通过SesssetupX请求数据报发送到服务
器端。服务器将会核查密码的有效性，并由此允许或拒绝客户端的访问。

你必须知道一个SMB密码(未加密)的最大长度是14位。密钥的长度一般为8位，加密过后的口令长
度为24位。在ANSI密码中，密码中的所有位都转换成大写的形式然后再加密。

密码是以DEC编码方式进行加密的。

  
--[ 6 - 几种SMB数据报的描述

这一部分，我将会对SMB协议中涉及到的大多数重要的数据报类型进行分析。我知道这或许很烦，
不过这是了解SMB的工作机制和进行攻击的基础。我将会解释什么是数据报中最重要的类型。每种命
令都对应两种数据报类型，请求数据报和应答数据报。

----[ 6.1 - SMB数据报的常规特征

通常情况下，SMB运行于TCP/IP协议组之上。那么我们就假设SMB运行在TCP层之上。在TCP层上面，
你常常会发现NETBIOS(NBT)头部。在NBT上面，有SMB基础报文头部。在SMB基础报文头部之上，就是
另一种依赖于特定请求命令的头部。

----------------------
│TCP Header │
----------------------
│NETBIOS Header │
----------------------
│SMB Base Header │
----------------------
│SMB Command Header │
----------------------
│DATA │
----------------------
  
“SMB Base header”包含了几种信息，像接收缓冲区的长度，允许的最大连接数目……它也包
含了一个鉴别请求命令的数字。

“SMB Command header”包含了所有的请求命令的参数(像磋商协议版本命令……)。

“DATA”容纳了请求命令的数据。

我们把“SMB packet”看成：NETBIOS Header + SMB Base Header + SMB Command Header
+ DATA。

注意：我将使用这个定义：

typedef unsigned char UCHAR; // 8 unsigned bits
typedef unsigned short USHORT; // 16 unsigned bits
typedef unsigned long ULONG; // 32 unsigned bits
  
而STRING被定义为以空字符结束的ASCII字符串。

----[ 6.2 - NETBIOS与SMB
  
NETBIOS(NETwork Basic Input and Output System)在微软的网络系统中被广泛使用。它是
一个软件接口和命名系统。每台主机都有一个长度为15个字符的NETBIOS名字，且第十六个字符用来
标志主机的类型(域名服务器，工作站……)。

第十六个字符的选择：

0x00 基础电脑，工作站。
0x20 资源共享服务器。

这还有一些其他的值可选，不过我们对这两个最感兴趣。第一个(0x00)鉴定一台工作站，第二个
(0x20)鉴定服务器。

在一个SMB数据报中，NETBIOS头部对应NETBIOS会话头部。

定义如下：

UCHAR Type; // Type of the packet
UCHAR Flags; // Flags
USHORT Length; // Count of data bytes (netbios header not included)
  
“Flags”域的值总是被置为0。

“Type”域有几种可能的选择：

0x81 对应一个NETBIOS会话请求。这个代码在客户端发送它的NETBIOS名字到服务器是使用。

0x82 对应一个NETBIOS会话应答。这个代码在服务器向客户端批准NETBIOS会话时使用。

0x00 对应一个会话消息。这个代码总是在SMB会话中被使用。

“Length”域包含了数据字节的长度(NETBIOS头部没有被包含在内)。数据包含在NETBIOS头部
以上的所有部分(它可能是SMB Base Header + SMB Command Header + DATA 或 NETBIOS名字)。

NETBIOS名字与编码。

NETBIOS编码名字的长度为32字节。

NETBIOS名字总是以大写的形式存在的。

编码一个NETBIOS名字非常的简单。例如我的计算机的NETBIOS名字是“BILL”，它是一个工作
站，所以它的第十六个字符为“0x00”。

首先，如果一个NETBIOS名字比15字节短，就会在右边补填上空格。

“BILL “

十六进制为：0x42 0x49 0x4c 0x4c 0x20 0x20 ......0x00
  
每个字节都分裂为4位一组：

0x4 0x2 0x4 0x9 0x4 0xc 0x4 0xc 0x2 0x0 .......
  
而且每个4位都要添加ASCII码‘A’的值(0x41)。

0x4 + 0x41 = 0x45 -> ASCII value = E
0x2 + 0x41 = 0x43 -> ASCII value = C
……

最后NETBIOS名字被编码为32字节长。

注意：

SMB可以直接运行于TCP之上而无须NBT(在Windows 2k和XP上它们使用455端口)。此时，
NETBIOS名字没有被限制在15字符以内。

----[ 6.3 - SMB的基础报文头部

这个头部在所有的SMB数据报中都会使用，以下是它的定义：

  
UCHAR Protocol[4]; // Contains 0xFF,'SMB'
UCHAR Command; // Command code
union {
struct {
UCHAR ErrorClass; // Error class
UCHAR Reserved; // Reserved for future use
USHORT Error; // Error code
} DosError;
ULONG Status; // 32-bit error code
} Status;
UCHAR Flags; // Flags
USHORT Flags2; // More flags
union {
USHORT Pad[6]; // Ensure section is 12 bytes long
struct {
USHORT PidHigh; // High part of PID
ULONG Unused; // Not used
ULONG Unused2;
} Extra;
};
USHORT Tid; // Tree identifier
USHORT Pid; // Caller's process id
USHORT Uid; // Unauthenticated user id
USHORT Mid; // multiplex id
UCHAR WordCount; // Count of parameter words
USHORT ParameterWords[ WordCount ]; // The parameter words
USHORT ByteCount; // Count of bytes
UCHAR Buffer[ ByteCount ]; // The bytes
  
“Protocol”域包含协议(SMB)的名字，前面放了一个0xFF。

“Command”域包含请求命令的数据。例如0x72就是“磋商协议”命令。

“Tid”域在客户端成功和一台SMB服务器上的资源建立连接后被使用的。TID数字用来鉴别资源。

“Pid”域在客户端成功在服务器上创建一个进程是使用。PID数字用来鉴别进程。

“Uid”域在一个用户被成功通过验证后被使用。UID数字用来鉴别用户。

“Mid”域在客户端拥有几个请求(进程，线程，文件访问……)是和PID同时使用。

“Flags”域也很重要，如果第15位置1，则使用UNICODE编码。

----[ 6.4 - 重要SMB命令的描述

SMB 磋商协议(negprot)
  
磋商协议在SMB会话建立连接的第一步时使用。

在SMB基础报文头部中的“Command”域被填充为：0x72。

下面是negprot请求与应答数据报的定义：

Request header
  
UCHAR WordCount; //Count of parameter words = 0
USHORT ByteCount; //Count of data bytes
struct {
UCHAR BufferFormat; //0x02 -- Dialect
UCHAR DialectName[]; //ASCII null-terminated string
} Dialects[];
  
这个数据报包括客户端向服务器发送的它所支持SMB协议的所有版本信息。

有三件事要说，在这个报文中：
“WordCount”域总是被置为零；
“ByteCount”域等于结构“Dialects”的长度；
“BufferFormat”域总是等于0x02。

“DialectName”包含SMB协议支持的几种版本信息。

应答数据报头部：

UCHAR WordCount; Count of parameter words = 17
USHORT DialectIndex; Index of selected dialect
UCHAR SecurityMode; Security mode:
bit 0: 0 = share, 1 = user
bit 1: 1 = encrypt passwords
USHORT MaxMpxCount; Max pending multiplexed requests
USHORT MaxNumberVcs; Max VCs between client and server
ULONG MaxBufferSize; Max transmit buffer size
ULONG MaxRawSize; Maximum raw buffer size
ULONG SessionKey; Unique token identifying this session
ULONG Capabilities; Server capabilities
ULONG SystemTimeLow; System (UTC) time of the server (low).
ULONG SystemTimeHigh; System (UTC) time of the server (high).
USHORT ServerTimeZone; Time zone of server (min from UTC)
UCHAR EncryptionKeyLength; Length of encryption key.
USHORT ByteCount; Count of data bytes
UCHAR EncryptionKey[]; The challenge encryption key
UCHAR OemDomainName[]; The name of the domain (in OEM chars)
  
这个数据报由服务器发出，它包含SMB协议支持的版本列表，服务器的SMB域名，如果需要还要
包含密钥。

重要的：

第一个感兴趣的域是“SecurityMode”位。如果第0位被选中，那么我们就选择了用户安全模式；
如果没有，则拥有共享安全模式。如果第1位被选中，密码就使用DEC加密算法进行编码。

“SessionKey”域被用来鉴别会话。一个会话有单一的一个会话钥匙。

“Capabilities”域表明服务器是否支持UNICODE字符串，或NT LM 0.12特别的命令……

数据被放在报文头部的结束处。通过一个negprot应答，这些数据与“EncryptionKey”和
“OemDomainName”相对应。

“OemDomainName”域的长度等于（Bytecount - EncryptoinKeyLength)。

“OemDomainName”字符串包含服务器的SMB域名。

SesssetupX(Session setup and X)
  
SesssetupX数据报被用来处理用户鉴别，或在你访问资源时提供一个密码。

SesssetupX的命令代码是：0x73。

请求数据报头部：

UCHAR WordCount; Count of parameter words = 13
UCHAR AndXCommand; Secondary (X) command; 0xFF = none
UCHAR AndXReserved; Reserved (must be 0)
USHORT AndXOffset; Offset to next command WordCount
USHORT MaxBufferSize; Client's maximum buffer size
USHORT MaxMpxCount; Actual maximum multiplexed pending requests
USHORT VcNumber; 0=first (only),nonzero=additional VC number
ULONG SessionKey; Session key (valid iff VcNumber != 0)
USHORT CaseInsensitivePasswordLength; Account password size, ANSI
USHORT CaseSensitivePasswordLength; Account password size, Unicode
ULONG Reserved; must be 0
ULONG Capabilities; Client capabilities
USHORT ByteCount; Count of data bytes; min = 0
UCHAR CaseInsensitivePassword[]; Account Password, ANSI
UCHAR CaseSensitivePassword[]; Account Password, Unicode
STRING AccountName[]; Account Name, Unicode
STRING PrimaryDomain[]; Client's primary domain, Unicode
STRING NativeOS[]; Client's native operating system, Unicode
STRING NativeLanMan[]; Client's native LAN Manager type, Unicode
  
这个报文提供很多关于客户端系统的信息。

“MaxBufferSize”域非常的重要，它提供客户端可以接收数据报的最大长度。如果你将这个域
设置为零，那么你不会接收到任何类型的数据。

这个数据报内，有几个字符串。最总要的是“CaseSensitivePassword”(UNICODE编码的密码)
域和“CaseInsensitivePassword”(ANSI编码的密码)域。

这两个域中的一个被使用，它取决于服务器是否支持UNICODE编码(在磋商协议应答报文中描述)。
密码的长度保存在“CaseInsesitivePasswordLength”或“CaseSensitivePasswordLength”里。

其他字符串，请参见它后面的描述。数据的长度保存在“Bytecount”域中。

应答数据报头部：

UCHAR WordCount; Count of parameter words = 3
UCHAR AndXCommand; Secondary (X) command; 0xFF = none
UCHAR AndXReserved; Reserved (must be 0)
USHORT AndXOffset; Offset to next command WordCount
USHORT Action; Request mode: bit0 = logged in as GUEST
USHORT ByteCount; Count of data bytes
STRING NativeOS[]; Server's native operating system
STRING NativeLanMan[]; Server's native LAN Manager type
STRING PrimaryDomain[]; Server's primary domain
  
在这个数据报文里也包含了很多信息：操作系统类型，SMB服务器版本信息和服务器的域名。

如果连接失败，那么在NativeOS, NativeLanman
and PrimaryDomain 里面将没有任何东西。

我们已经讲完了“硬件”部分，下面我们来看看SMB协议。

----[ 6.5 我们如何才能将本应该加密的SMB密码清楚的还原 ？

在会话的建立过程中，密码是在SesssetupX阶段被发送到服务器的。而SMB negprot应答报文包
含了一个“SecurityMode”域，它判断是否允许使用加密属性。

如果你想在所有的密码都被加密后获得一个清晰的密码，你可能会面对两种情况。

第一种情况是得到密钥和加密过的口令，然后暴力破解。它会花费你很多的时间……

一些程序如LophtCrack (附SMBGrinder)，dsniff or readsmb2 会嗅探到SMB加密过的密码。

第二种方法是劫持连接，并使客户端确信密码不用加密后传送。

这种技术解释起来有点复杂，不过我会告诉你如何才能实现 ！

如果服务器配置成需要加密的密码，那么SMB negprot应答数据报就会选种“SecurityMode”的
第1位。但是，如果攻击者在服务器之前发送了一个negprot应答数据报，且相应的第一位被置为零就
可以了，这样密码就会以明文的形式出现在SesssetupX请求报文。

  
negprot request
[client] ----------------------------------> [server]
  
[ attacker ]
(攻击者等待一个negprot请求数据报)
  
[client] <-------------│[server]
│fake negprot reply
│
[ attacker ]
(攻击者发送一个虚假的neprot应答数据报)
  
  
real negprot reply
[client] <---------------------------------- [server]
  
  
[attacker (什么都不做)]
  
  
sessetupX request with the password in clear text
[client] ----------------------------------> [server]
  
(攻击者嗅探到明文形式的密码)
  
以上的图表明了在网络上进行一次直接数据报注射过程。大多数情况下，这种方法并不会发生，
因为虚假的negprot应答数据报会被处理。还有一个问题就是会话特性，有效的密码它不会在交换环
境中工作……
我们可以使用ARP毒药(ARP-Poisoning)来避免上面的所有问题。

我不会解释或描述什么是ARP毒药，你可以在互连网上找到很多这方面的文章。但是如果你不知道
这是什么，你只须知道这种攻击允许攻击者在客户端和服务器之间进行重定向并修改网络通信。

如果你考虑到这种情形，攻击者就在他们之间。

这就是man in the middle……

----[ 6.6 - Man in the middle 攻击

“Attack where your enemy is not expecting you“

Sun Tzu, 《The art of war 》

现在我就要为大家描述什么是man in the middle 攻击。这种攻击允许你回避交换机，可以避
免连接失败，进而获得密码的明文表达式。

现在我们认为客户端和服务器之间的通信被攻击者重定向了(由于ARP毒药)。客户端向服务器请
求一个SMB会话，它会向服务器的139端口发送一个数据报。这时攻击者接收到了这个数据报，但是攻
击者并没有将这个数据报重新发送到服务器。

所有接收到的发送给服务器SMB端口的数据报(现在发送到了攻击者的主机了)被重新定向到攻击者
主机的本地端口1139(使用NAT或iptables非常容易实现)。在攻击者的本地端口1139，有一个程序(一
个透明的代理程序)进行修改并重定向SMB数据报。

iptables/NAT的命令如下：

将接收到的数据报(139端口)重定向到本地端口(例如1139)。

#iptables -t nat -A PREROUTING -i eth0 -p tcp -s 192.168.1.3 --dport 139 -j REDIRECT --to-port 1139
  
192.168.1.3是客户端的IP地址。

重定向所有的网络通信：

#iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
  
需要做哪些修改呢？

攻击者修改negprot应答数据报使密码以明文的形式传送，攻击者同样要修复密钥。他将密钥的长
度设置为零，并将域名代替密钥。攻击者将“SecurityMode”位设置为零。这样，密码就不会被加密了。

此后，客户端将会在SesssetupX请求数据报中把密码以明文的的方式传送。在攻击者获得了密码
后，他会在把SesssetupX请求数据报发送给服务器之前用密钥将密码加密。

服务器发送一个SesssetupX应答报文以接受或拒绝会话。攻击者重定向SesssetupX应答报文和
以后的所有网络通信。

会话连接不会失败，而且没有人知道我们的man in the middle ！

描述：

ARP-P ARP-P
[client] <--------- [attacker] ---------> [server]
  
攻击者使用ARP毒药攻击，进而在两台主机之间重定性所有的网络数据报。

[client] <---------> [attacker] <---------> [server]
  
网络传输的重定向依靠NAT或iptables来实现。

port 139
[client] -----------------> [attacker] [server]
  
攻击者接收到第一个发送给SMB服务器的数据报。

[client] ----------------->[attacker 139] [server]
│
V
[attacker 1139]
  
攻击者将它重定向到1139端口。在1139端口上，我们的代理程序正在监听。

negprot request
[client] -----------------> [attacker] [server]
  
攻击者接收到了negprot请求数据报。

negprot request
[client] [attacker] -------------------> [server]
  
攻击者重定向negprot请求数据报到服务器。

negprot reply
[client] [attacker] <---------------------------- [server]
(加密属性位设置为要求对密码进行加密)
  
服务器发送了一个将加密属性设置为1的negprot应答数据报，要求客户端对密码进行加密传送。
攻击者并不会重定向发送这个数据报。他改变了要求加密的那一位，而告诉客户端不用对口令进行加
密。

  
negprot reply
[client] <----------------------------- [attacker] [server]
(加密属性位被设置为无须加密口令)
  
攻击者将修改过的无须加密口令的negprot应答报文发送到客户端。

SesssetupX request
[client] ------------------------> [attacker] [server]
(密码以明文形式传送)
  
客户端将密码以明文形式传送，这是攻击者得到了服务器的密码了 ！

SesssetupX request
[client] [attacker] ---------------------> [server]
(密码被加密过后传送)
  
攻击者将密码加密后通过SesssetupX请求数据报发送到服务器。

sesssetupX reply
[client] <------------- [attacker] <---------------- [server]
  
服务器发送SesssetupX应答报文，攻击者只是将它重定向到客户端。

[client] <------------> [attacker] <--------------> [server]
  
攻击者继续在服务器与客户端之间重定向数据报，直到SMB会话结束。

----[ 6.7 - 注意Windows 2k/XP里基于TCP的SMB操作

就想我先前说的，在Windows 2k/XP中SMB可以直接运行于TCP之上。SMB服务器在445端口上监听
所有的连接。但是这并不是如此的“直接”，实际上，我们使用一个4字节长的数据代替了NETBIOS头部。

描述：

│---------------│
│TCP │
│---------------│
│SPECIAL HEADER │
│---------------│
│SMB BASE HDR │
│---------------│
  
特殊头部的定义如下：

UCHAR Zero; // Set to zero
UCHAR Length[3]; // Count of data bytes
// the 4 bytes of the header are not included
  
这个特别的头部和NETBIOS头部并没有太大的区别，下面你就会知道为什么了。

下面是NETBIOS头部：

UCHAR Type; // Type of the packet
UCHAR Flags; // Flags
USHORT Length; // Count of data bytes (netbios header not included)
  
在SMB运行于TCP之上时，NETBIOS请求会话就不再需要了。

实际上，客户端和服务器的NETBIOS名字并不需要发送。所以，在NETBIOS里的“Type”域总是等
于零(如果“Type”域不等于零，在客户端发送它的NETBIOS的编码后的名字时这个域被置为0x81，如
果是服务器就填入0x82)。记住，在SMB的会话期间，“Type”域始终等于零。

第一个字节完全相同。

最后三个字节：

NETBIOS头部的“Flags”域始终置为零。只有数据报的长度会占据最后的两个字节。

我们可以得出这个结论：在NETBIOS没有使用时，NETBIOS头部和特殊头部没有什么区别。

Downgrade 攻击：

如果客户端(运行于Windows 2K 或XP之上)使用了NBT，它在连接时总是同时使用139和445端口。
如果客户端在445端口接收到一个应答报文，客户端就会在139端口发送一个RST数据报。如果客户端在
445端口没有接收到应答报文，他会试图使用139端口进行连接。如果在两个端口都没有回应，那么这
次会话就算失败了。

如果客户端没有使用NBT，它仅在445端口上进行连接。为了完成Downgrade 攻击，强迫客户端
不使用445端口而使用139端口，你必须是客户端确信445端口是关闭的。在使用透明代理攻击的情况下
是非常容易实现的。在使用iptables的情况下，只须在攻击者的主机上将接收的所有网络数据报从445
端口重定向到其他的端口。这样客户端就会使用139端口了。

以上假设是基于客户端使用了NBT的。

如果客户端没有使用，那么透明代理就会在445端口上进行数据报的流通。

好的，我们已经完成了还原密码的攻击了。现在我们要学习SMB的另一个重要部分。

--[ 7 - 事物处理子协议与RAP命令

在这部分我会解释一些SMB的命令：RAP命令。这些命令使用事物处理子协议。我同样会描述这些
子协议。

----[ 7.1 事物处理子协议

在大量的数据通过SMB会话传送时，或则有一个特别的操作请求，那么SMB协议就会包含一个事物
处理子协议。

事物处理子协议主要用在SMB远程过程调用(RPC)：RAP命令(RAP是Remote Administration
Protocal)，我将会在稍后解释。

事物处理子协议并不是来源与SMB协议。它只是为SMB提供的另外一个命令。所以事物处理子协议
是基于SMB基础报文头部的，它的命令代码是0x25。

像其他命令一样有发送和请求报文。

以下是事物处理请求报文头部：

UCHAR WordCount; Count of parameter words; value =
(14 + value of the “SetupCount“ field)
USHORT TotalParameterCount; Total parameter bytes being sent
USHORT TotalDataCount; Total data bytes being sent
USHORT MaxParameterCount; Max parameter bytes to return
USHORT MaxDataCount; Max data bytes to return
UCHAR MaxSetupCount; Max setup words to return
UCHAR Reserved;
USHORT Flags; Additional information:
bit 0 - also disconnect TID in TID
bit 1 - one-way transaction (no response)
ULONG Timeout;
USHORT Reserved2;
USHORT ParameterCount; Parameter bytes sent this buffer
USHORT ParameterOffset; Offset (from header start) to Parameters
USHORT DataCount; Data bytes sent this buffer
USHORT DataOffset; Offset (from header start) to data
UCHAR SetupCount; Count of setup words
UCHAR Reserved3; Reserved (pad above to word)
USHORT Setup[SetupCount]; Setup words (# = SetupWordCount)
USHORT ByteCount; Count of data bytes
STRING Name[]; Name of transaction
(NULL if SMB_COM_TRANSACTION2)
UCHAR Pad[]; Pad to SHORT or LONG
UCHAR Parameters UCHAR Pad1[]; Pad to SHORT or LONG
UCHAR Data[ DataCount ]; Data bytes (# = DataCount)
  
在大多数情况下，一个RAP命令通过事物处理子协议来发送时都需要几个事物处理数据报来发送参
数和数据。参数字节通常首先发送，数据字节紧跟在后。如果几个事物处理数据报必须相关联时，服
务器会送下面的这个小的数据报：

中间应答报文:
  
UCHAR WordCount; Count of parameter words = 0
USHORT ByteCount; Count of data bytes = 0
  
在事物处理请求报文头部，“TotalParameterCount”域描述发送的参数字节的长度，在
“TotalDataCount”域中也是这样的(发送的数据数目)。

从SMB基础报文头部的开始到参数字符的偏移量保存在“ParameterOffset”里，而数据字符的
偏移量保存在“DataOffset”域里。

“Parameter”域包含参数字符，而“Data”域则包含数据字符。

在“DataCount”域和“ParameterCount”域里分别保存了事物处理数据报里的数据字节和参数
字节的长度。

注意一下“WordCount”域，它包含了下面的值：14 + “SetupCount”域的值。但通常情况下，
“SetupCount”域都等于零。

发送的请求数据报和应答数据报之间并没有太多的差别。

事物处理应答报文:
  
UCHAR WordCount; Count of data bytes; value = 10 +
“Setupcount“ field.
USHORT TotalParameterCount; Total parameter bytes being sent
USHORT TotalDataCount; Total data bytes being sent
USHORT Reserved;
USHORT ParameterCount; Parameter bytes sent this buffer
USHORT ParameterOffset; Offset (from header start) to Parameters
USHORT ParameterDisplacement; Displacement of these Parameter bytes
USHORT DataCount; Data bytes sent this buffer
USHORT DataOffset; Offset (from header start) to data
USHORT DataDisplacement; Displacement of these data bytes
UCHAR SetupCount; Count of setup words
UCHAR Reserved2; Reserved (pad above to word)
USHORT Setup[SetupWordCount]; Setup words (# = SetupWordCount)
USHORT ByteCount; Count of data bytes
UCHAR Pad[]; Pad to SHORT or LONG
UCHAR Parameters UCHAR Pad1[]; Pad to SHORT or LONG
UCHAR Data[DataCount]; Data bytes (# = DataCount)
  
客户端必须使用“ParameterOffset”和“DataOffset”以了解参数和数据字节的偏移量(从SMB
基础报文头部开始)。

----[ 7.2 - RAP命令

RAP(Remote Administation Protocal)是SMB的RPC实现。

RAP 请求报文:
  
│---------------------------│
│TCP HDR │
│---------------------------│
│NETBIOS HDR │
│---------------------------│
│SMB BASE HDR │
│---------------------------│
│SMB TRANSACTION REQUEST HDR│
│---------------------------│
│RAP REQUEST PARAMETERS │
│---------------------------│
│RAP REQUEST DATAS │
│---------------------------│
  
RAP 应答报文:
  
│---------------------------│
│TCP HDR │
│---------------------------│
│NETBIOS HDR │
│---------------------------│
│SMB BASE HDR │
│---------------------------│
│SMB TRANSACTION REPLY HDR │
│---------------------------│
│RAP REPLY PARAMETERS │
│---------------------------│
│RAP REPLY DATAS │
│---------------------------│
  
在你使用RAP命令时，你总会在事物处理(请求和应答)的“Name”域中找到“/PIPE/LANMAN”
字符串。

这有几个RAP命令的例子：

- NETSHAREENUM : 获得服务器上所有共享资源的信息；

- NETSERVERENUM2 : 列举特定域里所有计算机的特定类型；

- NETSERVERGETINFO : 获得指定服务器的信息；

- NETSHAREGETINFO : 获得特定共享资源的信息；

- NETWKSTAUSERLOGON : 在SMB服务器上进行一次登录；

- NETWSTAUSERLOGOFF : 注销；

- NETUSERGETINFO : 获得指定用户的信息；

- NETWKSTAGETINFO : 获得指定工作站的信息；

- SAMOEMCHANGEPASSWORD : 更改远程SMB服务器上一个用户的密码。

我不会列举所有的命令，而只是举一个例子。

--[ 8 - 使用RAP命令列出服务器上所有可得的共享列表

这部分是上面的补充。我将会演示一个如何使用RAP命令的例子。

我们如何才能获得网络上的每人都可访问的SMB共享资源：

其实过程很简单。客户端必须在服务器上通过认证。这些在第三章里都有讲解。在服务器通过验
证后，客户端发送一个TconX请求数据报给服务器(在接收到SesssetupX应答数据报之后)。

TconX 的意思是Tree Connect and X。

TconX请求数据报用来访问一个共享资源。

----[ 8.1 - TconX数据报

TconX数据报是建立在SMB基础报文头部之上的，命令代码为0x75。

请求数据报头部：

UCHAR WordCount; Count of parameter words = 4
UCHAR AndXCommand; Secondary (X) command; 0xFF = none
UCHAR AndXReserved; Reserved (must be 0)
USHORT AndXOffset; Offset to next command WordCount
USHORT Flags; Additional information
  
USHORT PasswordLength; Length of Password[]
USHORT ByteCount; Count of data bytes; min = 3
UCHAR Password[]; Password
STRING Path[]; Server name and share name
STRING Service[]; Service name
  
密码在会话建立过程中被发送到了服务器。如果密码的长度为1，则密码就被置为空(0x00)。

“Path”域包含了你想要访问的共享资源的名字。它使用UNICODE编码。如过我想访问一台
“myserver”服务器上的“myshare”资源，“Path”字符串就该置为“//myserver/myshare”。

“Service”包含了请求资源的类型：

stringType of ressource
  
“A:” 磁盘共享.
“LPT1:” 打印机.
“IPC” 命名管道.
“COMM” 通信设备.
“?????” 任何设备类型.
  
如果你要扫描设备的类型，你必须在“Service”域里使用“?????”。

在你发送了一个TconX请求数据报到服务器后，它会给你发送一个TconX的应答报文。你还必须
修复“Tid”域(在SMB基础报文头部)，因为它是RAP命令里的事物处理请求。你还要告诉服务器你想
获得那些你有权得到的资源名字。当然，你可以通过“NETSHAREENUM”命令来获得它们。

----[ 8.2 - RAP命令“NetshareEnum”的解释

我们将要学习RAP命令里的“NetShareEnum”。

RAP命令中的“NetShareEnum”请求：

函数NetShareEnum的16位代码：0；

参数描述符： “WrLen”；

返回的数据描述符： “B13BWZ”；

一个16位的值为0x01的整数；

一个16位整数包含的获得数据的缓冲区长度；

在这个请求数据报中不需要任何数据，所以“DataCount”域和“TotalDataCount”域都置零。

│--------------------------------------------│
│NETBIOS HDR │---------> 4 bytes
│--------------------------------------------│
│SMB BASE HDR │---------> 32 Bytes
│--------------------------------------------│
│SMB TRANSACTION REQUEST HDR │
│--------------------------------------------│
  
事物处理请求的“Parameters”域获得RAP请求命令的参数：

│--------------│
│0x0000 │----------------------------------------> A
│--------------│--------------│--------------│
│W r │L e │h 0x00│-----------> B
│--------------│--------------│--------------│-------│
│B 1 │3 B │W Z │0x00 │---> C
│--------------│--------------│--------------│-------│
│0x0001 │0xffff │--------------------------> D
│--------------│--------------│
  
A: NetShareEnum函数代码: 0x00
B: 参数描述符
C: 数据描述符
D: 0x01(已定义值)和0xffff(获取数据报的最大长度)
  
服务器的应答：

“Parameters”域接收到事物处理应答报文头部信息：

一个16位整数包含了返回的状态值：

成功0
拒绝访问5
拒绝访问网络65
更多数据234
服务器已关闭2114
事物处理配置错误2141
  
一个16位的“转换字”，用来计算备注信息的偏移量。

一个16位获得返回的条目数= 返回SHARE_INFO数据结构的数目。

一个16位获得可利用的条目数。

在事物处理应答报文的“Data”域中包含了几组SHARE_INFO的数据结构。

SHARE_INFO数据结构包含了可利用资源的信息，定义如下：

struct SHARE_INFO {
char shi1_netname[13]; /*Name of the ressource*/
  
char shi1_pad; /*Pad to a word*/
  
unsigned short shi1_type; /*Code specifies the type of the shared resssource :
0 Disk Directory tree
1 Printer queue
2 Communications device
3 IPC*/
  
char *shi1_remark; /*Remark on the specified ressource*/
}
  
shi1_remark是一个32位的字符传指针。它包含了相应资源的备注信息。你必须取出它的后16位
到“converter word”域中，以知道这个备注字符串和RAP应答报文参数头部之间的偏移量。

实际上以ASCII编码是：

│--------------------------------------------│
│NETBIOS HDR │------------> 4 bytes
│--------------------------------------------│
│SMB BASE HDR │------------> 32 Bytes
│--------------------------------------------│
│SMB TRANS REPLY HDR │
│--------------------------------------------│
  
事物处理应答头部的“Parameters”域定义：
(对应于NetShareEnum函数返回的参数)
  
│--------------------------------------------│
│status code │-------------> 2 bytes
│--------------------------------------------│
│converted word │-------------> 2 bytes
│--------------------------------------------│
│number of entries returned │-------------> 2 bytes
│--------------------------------------------│
│number of entries available │-------------> 2 bytes
│--------------------------------------------│
  
事物处理应答报文的数据部分：
(对应于多个SHARE_INFO数据结构)
  
│--------------------------------------------│
│shi1_netname │-----------> 13 bytes
│--------------------------------------------│
│shi1_pad to pad to word │-----------> 1 byte
│--------------------------------------------│
│type of service │-----------> 2 bytes
│--------------------------------------------│
│pointer to remark string │-----------> 4 bytes
│--------------------------------------------│
.
其他的SHARE_INFO数据结构
.
│--------------------------------------------│
│remark string 1 │
│--------------------------------------------│
│another remarks strings │
│--------------------------------------------│
  
  
来源： http://www.jtyun.com.cn/forum.php?mod=viewthread&tid=103
