## mongoDb

- MongoDB 是由C++语言编写的，是一个基于分布式文件存储的开源数据库系统。

- 在高负载的情况下，添加更多的节点，可以保证服务器性能。

- MongoDB 旨在为WEB应用提供可扩展的高性能数据存储解决方案。

- MongoDB 将数据存储为一个文档，数据结构由键值(key=>value)对组成。MongoDB 文档类似于 JSON 对象。字段值可以包含其他文档，数组及文档数组。

- 主要特点

    - MongoDB 是一个面向文档存储的数据库，操作起来比较简单和容易。

    - 通过本地或者网络创建数据镜像，这使得MongoDB有更强的扩展性。

    - 可以在MongoDB记录中设置任何属性的索引 (如：FirstName="Sameer",Address="8 Gandhi Road")来实现更快的排序。

    - 支持丰富的查询表达式。查询指令使用JSON形式的标记，可轻易查询文档中内嵌的对象及数组。

    - Mongodb中的Map/reduce主要是用来对数据进行批量处理和聚合操作。

### MongoDB 应用案例
- 一些公司MongoDB的实际应用：
    - Craiglist上使用MongoDB的存档数十亿条记录。
    - FourSquare，基于位置的社交网站，在Amazon EC2的服务器上使用MongoDB分享数据。
    - Shutterfly，以互联网为基础的社会和个人出版服务，使用MongoDB的各种持久性数据存储的要求。
    - bit.ly, 一个基于Web的网址缩短服务，使用MongoDB的存储自己的数据。
    - spike.com，一个MTV网络的联营公司， spike.com使用MongoDB的。
    - Intuit公司，一个为小企业和个人的软件和服务提供商，为小型企业使用MongoDB的跟踪用户的数据。
    - sourceforge.net，资源网站查找，创建和发布开源软件免费，使用MongoDB的后端存储。
    - etsy.com ，一个购买和出售手工制作物品网站，使用MongoDB。
    - 纽约时报，领先的在线新闻门户网站之一，使用MongoDB。
    - CERN，著名的粒子物理研究所，欧洲核子研究中心大型强子对撞机的数据使用MongoDB。


### MongoDB 概念解析
SQL术语/概念|MongoDB术语/概念|解释/说明
-|-|-
database|database|数据库
table|collection|数据库表/集合
row|document|数据记录行/文档
column|field|数据字段/域
index|index|索引
table joins|	 |表连接,MongoDB不支持
primary key	|primary key|主键,MongoDB自动将_id字段设置为主键

### 数据库
- 一个mongodb中可以建立多个数据库。

- MongoDB的默认数据库为"db"，该数据库存储在data目录中。

- MongoDB的单个实例可以容纳多个独立的数据库，每一个都有自己的集合和权限，不同的数据库也放置在不同的文件中。

- "show dbs" 命令可以显示所有数据的列表。

- 有一些数据库名是保留的，可以直接访问这些有特殊作用的数据库。
    - admin： 从权限的角度来看，这是"root"数据库。要是将一个用户添加到这个数据库，这个用户自动继承所有数据库的权限。一些特定的服务器端命令也只能从这个数据库运行，比如列出所有的数据库或者关闭服务器。
    - local: 这个数据永远不会被复制，可以用来存储限于本地单台服务器的任意集合
    - config: 当Mongo用于分片设置时，config数据库在内部使用，用于保存分片的相关信息。

### 文档(Document)
文档是一组键值(key-value)对(即 BSON)。MongoDB 的文档不需要设置相同的字段，并且相同的字段不需要相同的数据类型，这与关系型数据库有很大的区别，也是 MongoDB 非常突出的特点。

- 需要注意的是：
    - 文档中的键/值对是有序的。
    - 文档中的值不仅可以是在双引号里面的字符串，还可以是其他几种数据类型（甚至可以是整个嵌入的文档)。
    - MongoDB区分类型和大小写。
    - MongoDB的文档不能有重复的键。
    - 文档的键是字符串。除了少数例外情况，键可以使用任意UTF-8字符。

- 文档键命名规范：
    - 键不能含有\0 (空字符)。这个字符用来表示键的结尾。
    - .和$有特别的意义，只有在特定环境下才能使用。
    - 以下划线"_"开头的键是保留的(不是严格要求的)。

### 集合
- 集合就是 MongoDB 文档组，类似于 RDBMS （关系数据库管理系统：Relational Database Management System)中的表格。

- 集合存在于数据库中，集合没有固定的结构，这意味着你在对集合可以插入不同格式和类型的数据，但通常情况下我们插入集合的数据都会有一定的关联性。

- 合法的集合名
    - 集合名不能是空字符串""。
    - 集合名不能含有\0字符（空字符)，这个字符表示集合名的结尾。
    - 集合名不能以"system."开头，这是为系统集合保留的前缀。
    - 用户创建的集合名字不能含有保留字符。有些驱动程序的确支持在集合名里面包含，这是因为某些系统生成的集合中包含该字符。除非你要访问这种系统创建的集合，否则千万不要在名字里出现$。　

#### capped collections
- Capped collections 就是固定大小的collection。

- 它有很高的性能以及队列过期的特性(过期按照插入的顺序). 有点和 "RRD" 概念类似。

- Capped collections 是高性能自动的维护对象的插入顺序。它非常适合类似记录日志的功能和标准的 collection 不同，你必须要显式的创建一个capped collection，指定一个 collection 的大小，单位是字节。collection 的数据存储空间值提前分配的。

- Capped collections 可以按照文档的插入顺序保存到集合中，而且这些文档在磁盘上存放位置也是按照插入顺序来保存的，所以当我们更新Capped collections 中文档的时候，更新后的文档不可以超过之前文档的大小，这样话就可以确保所有文档在磁盘上的位置一直保持不变。

- 由于 Capped collection 是按照文档的插入顺序而不是使用索引确定插入位置，这样的话可以提高增添数据的效率。MongoDB 的操作日志文件 oplog.rs 就是利用 Capped Collection 来实现的。

>在 capped collection 中，你能添加新的对象。能进行更新，然而，对象不会增加存储空间。如果增加，更新就会失败 。

>使用 Capped Collection 不能删除一个文档，可以使用 drop() 方法删除 collection 所有的行。

>删除之后，你必须显式的重新创建这个 collection。在32bit机器中，capped collection 最大存储为 1e9( 1X109)个字节。

### 元数据
数据库的信息是存储在集合中。它们使用了系统的命名空间

>在MongoDB数据库中名字空间` <dbname>.system.* `是包含多种系统信息的特殊集合(Collection)，如下:

集合命名空间|描述
-|-
dbname.system.namespaces|列出所有名字空间。
dbname.system.indexes|列出所有索引。
dbname.system.profile|包含数据库概要(profile)信息。
dbname.system.users|列出所有可访问数据库的用户。
dbname.local.sources|包含复制对端（slave）的服务器信息和状态。

### MongoDB 数据类型

数据类型|描述
-|-
String|字符串。存储数据常用的数据类型。在 MongoDB 中，UTF-8 编码的字符串才是合法的。
Integer|整型数值。用于存储数值。根据你所采用的服务器，可分为 32 位或 64 位。
Boolean|布尔值。用于存储布尔值（真/假）。
Double|双精度浮点值。用于存储浮点值。
Min/Max keys|将一个值与 BSON（二进制的 JSON）元素的最低值和最高值相对比。
Array|用于将数组或列表或多个值存储为一个键。
Timestamp|时间戳。记录文档修改或添加的具体时间。
Object|用于内嵌文档。
Null|用于创建空值。
Symbol|符号。该数据类型基本上等同于字符串类型，但不同的是，它一般用于采用特殊符号类型的语言。
Date|日期时间。用 UNIX 时间格式来存储当前日期或时间。你可以指定自己的日期时间：创建 Date 对象，传入年月日信息。
Object ID|对象 ID。用于创建文档的 ID。
Binary Data|二进制数据。用于存储二进制数据。
Code|代码类型。用于在文档中存储 JavaScript 代码。
Regular expression|正则表达式类型。用于存储正则表达式。

### MongoDB - 连接

标准 URI 连接语法：
```
mongodb://[username:password@]host1[:port1][,host2[:port2],...[,hostN[:portN]]][/[database][?options]]
```

- mongodb:// 这是固定的格式，必须要指定。

- username:password@ 可选项，如果设置，在连接数据库服务器之后，驱动都会尝试登陆这个数据库

- host1 必须的指定至少一个host, host1 是这个URI唯一要填写的。它指定了要连接服务器的地址。如果要连接复制集，请指定多个主机地址。

- portX 可选的指定端口，如果不填，默认为27017

- /database 如果指定username:password@，连接并验证登陆指定数据库。若不指定，默认打开 test 数据库。

- ?options 是连接选项。如果不使用/database，则前面需要加上/。所有连接选项都是键值对name=value，键值对之间通过&或;（分号）隔开

#### 标准的连接格式包含了多个选项(options)
选项|描述
-|-
replicaSet=name|验证replica set的名称。 Impliesconnect=replicaSet.
slaveOk=true/false	|true:在connect=direct模式下，驱动会连接第一台机器，即使这台服务器不是主。在connect=replicaSet模式下，驱动会发送所有的写请求到主并且把读取操作分布在其他从服务器。false: 在 connect=direct模式下，驱动会自动找寻主服务器. 在connect=replicaSet 模式下，驱动仅仅连接主服务器，并且所有的读写命令都连接到主服务器。
safe=true/false|true: 在执行更新操作之后，驱动都会发送getLastError命令来确保更新成功。(还要参考 wtimeoutMS).false: 在每次更新之后，驱动不会发送getLastError来确保更新成功。
w=n	|驱动添加 { w : n } 到getLastError命令. 应用于safe=true。
wtimeoutMS=ms|驱动添加 { wtimeout : ms } 到 getlasterror 命令. 应用于 safe=true.
fsync=true/false	|true: 驱动添加 { fsync : true } 到 getlasterror 命令.应用于 safe=true.false: 驱动不会添加到getLastError命令中。
journal=true/false|如果设置为 true, 同步到 journal (在提交到数据库前写入到实体中). 应用于 safe=true
connectTimeoutMS=ms|可以打开连接的时间。
socketTimeoutMS=ms|发送和接受sockets的时间。
