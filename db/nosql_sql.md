本文导读：随着web2.0的快速发展，非关系型、分布式数据存储得到了快速的发展，它们不保证关系数据的ACID特性。NoSQL被我们用得最多的当数key-value存储，当然还有其他的文档型的、列存储、图型数据库、xml数据库等。那么，NoSQL的优缺点有哪些呢？我们为什么要选择NoSQL呢？
 
NoSQL 是非关系型数据存储的广义定义。NoSQL 数据存储不需要固定的表结构，通常也不存在连接操作。在大数据存取上具备关系型数据库无法比拟的性能优势。
 
当今的应用体系结构需要数据存储在横向伸缩性上能够满足需求。而 NoSQL 存储就是为了实现这个需求。Google 的BigTable与Amazon的Dynamo是非常成功的商业 NoSQL 实现。一些开源的 NoSQL 体系，如Facebook 的Cassandra， Apache 的HBase，也得到了广泛认同。从这些NoSQL项目的名字上看不出什么相同之处：Hadoop、Voldemort、Dynomite，还有其它很多。
 
 
一、NoSQL与关系型数据库设计理念比较
 
关系型数据库中的表都是存储一些格式化的数据结构，每个元组字段的组成都一样，即使不是每个元组都需要所有的字段，但数据库会为每个元组分配所有的字段，这样的结构可以便于表与表之间进行连接等操作，但从另一个角度来说它也是关系型数据库性能瓶颈的一个因素。而非关系型数据库以键值对存储，它的结构不固定，每一个元组可以有不一样的字段，每个元组可以根据需要增加一些自己的键值对，这样就不会局限于固定的结构，可以减少一些时间和空间的开销。
 
 
 
二、NOSQL的优势
 
 
易扩展
NoSQL数据库种类繁多，但是一个共同的特点都是去掉关系数据库的关系型特性。数据之间无关系，这样就非常容易扩展。也无形之间，在架构的层面上带来了可扩展的能力。
 
大数据量，高性能
NoSQL数据库都具有非常高的读写性能，尤其在大数据量下，同样表现优秀。这得益于它的无关系性，数据库的结构简单。一般MySQL使用Query Cache，每次表的更新Cache就失效，是一种大粒度的Cache，在针对web2.0的交互频繁的应用，Cache性能不高。而NoSQL的Cache是记录级的，是一种细粒度的Cache，所以NoSQL在这个层面上来说就要性能高很多了。
 
灵活的数据模型
NoSQL无需事先为要存储的数据建立字段，随时可以存储自定义的数据格式。而在关系数据库里，增删字段是一件非常麻烦的事情。如果是非常大数据量的表，增加字段简直就是一个噩梦。这点在大数据量的web2.0时代尤其明显。
 
高可用
NoSQL在不太影响性能的情况，就可以方便的实现高可用的架构。比如Cassandra，HBase模型，通过复制模型也能实现高可用。
 
三、NoSQL数据库还存在着很多的不足，常见主要有下面这几个
 
1. 不提供对SQL的支持：如果不支持SQL这样的工业标准，将会对用户产生一定的学习和应用迁移成本;
2. 支持的特性不够丰富：现有产品所提供的功能都比较有限，大多数NoSQL数据库都不支持事务，也不像MS SQL Server和Oracle那样能提供各种附加功能，比如BI和报表等;
3. 现有产品的不够成熟：大多数产品都还处于初创期，和关系型数据库几十年的完善不可同日而语;
 
四、NoSQL的分类
 
类型	部分代表	特点
列存储	
Hbase
Cassandra
Hypertable
顾名思义，是按列存储数据的。最大的特点是方便存储结构化和半结构化数据，方便做数据压缩，对针对某一列或者某几列的查询有非常大的IO优势。
文档存储	
MongoDB
CouchDB
文档存储一般用类似json的格式存储，存储的内容是文档型的。这样也就有有机会对某些字段建立索引，实现关系数据库的某些功能。
key-value存储	
Tokyo Cabinet / Tyrant
Berkeley DB
MemcacheDB
Redis
可以通过key快速查询到其value。一般来说，存储不管value的格式，照单全收。（Redis包含了其他功能）
图存储	
Neo4J
FlockDB
图形关系的最佳存储。使用传统关系数据库来解决的话性能低下，而且设计使用不方便。
对象存储	
db4o
Versant
通过类似面向对象语言的语法操作数据库，通过对象的方式存取数据。
xml数据库	Berkeley DB XML	高效的存储XML数据，并支持XML的内部查询语法，比如XQuery,Xpath。
 
五、Cassandra、 Mongodb、CouchDB、Redis、Riak、 Membase、Neo4j和HBase的介绍
 
1. CouchDB
 
所用语言： Erlang
特点：DB一致性，易于使用
使用许可： Apache
协议： HTTP/REST
双向数据复制，
持续进行或临时处理，
处理时带冲突检查，
因此，采用的是master-master复制（见编注2）
MVCC – 写操作不阻塞读操作
可保存文件之前的版本
Crash-only（可靠的）设计
需要不时地进行数据压缩
视图：嵌入式 映射/减少
格式化视图：列表显示
支持进行服务器端文档验证
支持认证
根据变化实时更新
支持附件处理
因此， CouchApps（独立的 js应用程序）
需要 jQuery程序库
最佳应用场景：适用于数据变化较少，执行预定义查询，进行数据统计的应用程序。适用于需要提供数据版本支持的应用程序。
例如： CRM、CMS系统。 master-master复制对于多站点部署是非常有用的。
 
2. Redis
 
所用语言：C/C++
特点：运行异常快
使用许可： BSD
协议：类 Telnet
有硬盘存储支持的内存数据库，
但自2.0版本以后可以将数据交换到硬盘（注意， 2.4以后版本不支持该特性！）
Master-slave复制（见编注3）
虽然采用简单数据或以键值索引的哈希表，但也支持复杂操作，例如 ZREVRANGEBYSCORE。
INCR & co （适合计算极限值或统计数据）
支持 sets（同时也支持 union/diff/inter）
支持列表（同时也支持队列；阻塞式 pop操作）
支持哈希表（带有多个域的对象）
支持排序 sets（高得分表，适用于范围查询）
Redis支持事务
支持将数据设置成过期数据（类似快速缓冲区设计）
Pub/Sub允许用户实现消息机制
最佳应用场景：适用于数据变化快且数据库大小可遇见（适合内存容量）的应用程序。
例如：股票价格、数据分析、实时数据搜集、实时通讯。
 
3. MongoDB
 
所用语言：C++
特点：保留了SQL一些友好的特性（查询，索引）。
使用许可： AGPL（发起者： Apache）
协议： Custom, binary（ BSON）
Master/slave复制（支持自动错误恢复，使用 sets 复制）
内建分片机制
支持 javascript表达式查询
可在服务器端执行任意的 javascript函数
update-in-place支持比CouchDB更好
在数据存储时采用内存到文件映射
对性能的关注超过对功能的要求
建议最好打开日志功能（参数 –journal）
在32位操作系统上，数据库大小限制在约2.5Gb
空数据库大约占 192Mb
采用 GridFS存储大数据或元数据（不是真正的文件系统）
最佳应用场景：适用于需要动态查询支持；需要使用索引而不是 map/reduce功能；需要对大数据库有性能要求；需要使用 CouchDB但因为数据改变太频繁而占满内存的应用程序。
例如：你本打算采用 MySQL或 PostgreSQL，但因为它们本身自带的预定义栏让你望而却步。
 
4. Riak
 
所用语言：Erlang和C，以及一些Javascript
特点：具备容错能力
使用许可： Apache
协议： HTTP/REST或者 custom binary
可调节的分发及复制(N, R, W)
用 JavaScript or Erlang在操作前或操作后进行验证和安全支持。
使用JavaScript或Erlang进行 Map/reduce
连接及连接遍历：可作为图形数据库使用
索引：输入元数据进行搜索（1.0版本即将支持）
大数据对象支持（ Luwak）
提供“开源”和“企业”两个版本
全文本搜索，索引，通过 Riak搜索服务器查询（ beta版）
支持Masterless多站点复制及商业许可的 SNMP监控
最佳应用场景：适用于想使用类似 Cassandra（类似Dynamo）数据库但无法处理 bloat及复杂性的情况。适用于你打算做多站点复制，但又需要对单个站点的扩展性，可用性及出错处理有要求的情况。
例如：销售数据搜集，工厂控制系统；对宕机时间有严格要求；可以作为易于更新的 web服务器使用。
 
5. Membase
 
所用语言： Erlang和C
特点：兼容 Memcache，但同时兼具持久化和支持集群
使用许可： Apache 2.0
协议：分布式缓存及扩展
非常快速（200k+/秒），通过键值索引数据
可持久化存储到硬盘
所有节点都是唯一的（ master-master复制）
在内存中同样支持类似分布式缓存的缓存单元
写数据时通过去除重复数据来减少 IO
提供非常好的集群管理 web界面
更新软件时软无需停止数据库服务
支持连接池和多路复用的连接代理
最佳应用场景：适用于需要低延迟数据访问，高并发支持以及高可用性的应用程序
例如：低延迟数据访问比如以广告为目标的应用，高并发的 web 应用比如网络游戏（例如 Zynga）
 
6. Neo4j
 
所用语言： Java
特点：基于关系的图形数据库
使用许可： GPL，其中一些特性使用 AGPL/商业许可
协议： HTTP/REST（或嵌入在 Java中）
可独立使用或嵌入到 Java应用程序
图形的节点和边都可以带有元数据
很好的自带web管理功能
使用多种算法支持路径搜索
使用键值和关系进行索引
为读操作进行优化
支持事务（用 Java api）
使用 Gremlin图形遍历语言
支持 Groovy脚本
支持在线备份，高级监控及高可靠性支持使用 AGPL/商业许可
最佳应用场景：适用于图形一类数据。这是 Neo4j与其他nosql数据库的最显著区别
例如：社会关系，公共交通网络，地图及网络拓谱
 
7. Cassandra
 
所用语言： Java
特点：对大型表格和 Dynamo支持得最好
使用许可： Apache
协议： Custom, binary (节约型)
可调节的分发及复制(N, R, W)
支持以某个范围的键值通过列查询
类似大表格的功能：列，某个特性的列集合
写操作比读操作更快
基于 Apache分布式平台尽可能地 Map/reduce
我承认对 Cassandra有偏见，一部分是因为它本身的臃肿和复杂性，也因为 Java的问题（配置，出现异常，等等）
最佳应用场景：当使用写操作多过读操作（记录日志）如果每个系统组建都必须用 Java编写（没有人因为选用 Apache的软件被解雇）
例如：银行业，金融业（虽然对于金融交易不是必须的，但这些产业对数据库的要求会比它们更大）写比读更快，所以一个自然的特性就是实时数据分析
 
8. HBase
 
（配合 ghshephard使用）
所用语言： Java
特点：支持数十亿行X上百万列
使用许可： Apache
协议：HTTP/REST
在 BigTable之后建模
采用分布式架构 Map/reduce
对实时查询进行优化
高性能 Thrift网关
通过在server端扫描及过滤实现对查询操作预判
支持 XML, Protobuf, 和binary的HTTP
Cascading, hive, and pig source and sink modules
基于 Jruby（ JIRB）的shell
对配置改变和较小的升级都会重新回滚
不会出现单点故障
堪比MySQL的随机访问性能
最佳应用场景：适用于偏好BigTable:)并且需要对大数据进行随机、实时访问的场合。

来源： http://www.studyofnet.com/news/365.html
