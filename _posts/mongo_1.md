---
title: MongoDB基础篇（一）
date: 2017-12-22 00:46:21
toc: true
tags: NoSQL
categories: 数据库
---
## 概念与简介
### 简介：
MongoDB 是由C++语言编写的，是一个基于分布式文件存储的开源数据库系统。
在高负载的情况下，添加更多的节点，可以保证服务器性能。
MongoDB 旨在为WEB应用提供可扩展的高性能数据存储解决方案。
MongoDB 将数据存储为一个文档，数据结构由键值(key=>value)对组成。MongoDB 文档类似于 JSON 对象。字段值可以包含其他文档，数组及文档数组。
![存储文档举例](http://www.runoob.com/wp-content/uploads/2013/10/crud-annotated-document.png)
### 概念：
|SQL术语/概念|MongoDB术语/概念|解释/说明|
|:-------------:|:-------------:|:-------------:|
|database|database|数据库|
|table|collection|数据库表/集合|
|row|document|数据记录行/文档|
|column|field|数据字段/域|
|index|index|索引|
|table joins||表连接,MongoDB不支持|
|primary key|primary key|主键,MongoDB自动将_id字段设置为主键|
### 特点：
- 面向文档存储，操作简单容易
- 索引设置简单方便
- 可以通过本地或者网络创建数据镜像，扩展性强，支持分片的方式来增加负载处理能力
- 支持丰富的表达式查询，查询指令使用JSON
- 支持Map/reduce（js编写，通过db.runCommand或mapreduce命令来执行），主要对数据进行批量处理和聚合操作
- 内置GridFS，用于存放大量的小文件
- 允许服务端执行脚本

### 常用工具：
- Fang of Mongo – 网页式,由Django和jQuery所构成。
- Futon4Mongo – 一个CouchDB Futon web的mongodb山寨版。
- Mongo3 – Ruby写成。
- MongoHub – 适用于OSX的应用程序。
- Opricot – 一个基于浏览器的MongoDB控制台, 由PHP撰写而成。
- Database Master — Windows的mongodb管理工具
- RockMongo — 最好的PHP语言的MongoDB管理工具，轻量级, 支持多国语言.

## 安装及运行
### curl安装：
```
# 进入 /usr/local
cd /usr/local

# 下载
sudo curl -O https://fastdl.mongodb.org/osx/mongodb-osx-x86_64-3.4.2.tgz

# 解压
sudo tar -zxvf mongodb-osx-x86_64-3.4.2.tgz

# 重命名为 mongodb 目录
sudo mv mongodb-osx-x86_64-3.4.2 mongodb

# 安装完成后，把 MongoDB 的二进制命令文件目录（安装目录/bin）添加到 PATH 路径中：
export PATH=/usr/local/mongodb/bin:$PATH
```
### brew安装（推荐）：
```
sudo brew install mongodb

# 如果需要支持TLS/SSL
sudo brew install mongodb --with-openssl

#安装最新开发版本：
sudo brew install mongodb --devel
```

### 运行：
1.创建数据库存储目录（默认 /data/db）：

```
sudo mkdir -p /data/db
```

2.启动mongodb

```
sudo mongod

# 如果没有创建全局路径 PATH，需要进入以下目录
cd /usr/local/mongodb/bin
sudo ./mongod

指定db目录启动
sudo mongod --dbpath
```
## 基础操作
### 连接：
标准 URI 连接语法:

```
mongodb://[username:password@]host1[:port1][,host2[:port2],...[,hostN[:portN]]][/[database][?options]]
```
- mongodb:// 这是固定的格式，必须要指定。
- username:password 可选项，连接鉴权
- host1 必须的指定至少一个host,它指定了要连接服务器的地址。如果要连接复制集，需要指定多个主机地址。
- port 可选的指定端口，如果不填，默认为27017
- /database 如果指定username:password@，连接并验证登陆指定数据库。若不指定，默认打开 test 数据库。
- ?options 是连接选项。如果不使用/database，则前面需要加上/。所有连接选项都是键值对name=value,键值对之间通过&或;（分号）隔开

常用的一些options

|选项|描述|
|:-------------:|:-------------|
| replicaSet=name | 验证replica set的名称。 Impliesconnect=replicaSet. |
| slaveOk=true/false | `true`:在connect=direct模式下，驱动会连接第一台机器，即使这台服务器不是主。在connect=replicaSet模式下，驱动会发送所有的写请求到主并且把读取操作分布在其他从服务器。<br> `false`: 在 connect=direct模式下，驱动会自动找寻主服务器. 在connect=replicaSet 模式下，驱动仅仅连接主服务器，并且所有的读写命令都连接到主服务器。 |
| safe=true/false | `true`: 在执行更新操作之后，驱动都会发送getLastError命令来确保更新成功。(还要参考 wtimeoutMS). <br> `false`: 在每次更新之后，驱动不会发送getLastError来确保更新成功。|
| w=n | 驱动添加 { w : n } 到getLastError命令. 应用于safe=true。 |
| wtimeoutMS=ms | 驱动添加 { wtimeout : ms } 到 getlasterror 命令. 应用于 safe=true. |
| fsync=true/false | `true`: 驱动添加 { fsync : true } 到 getlasterror 命令. 应用于 safe=true. <br>  `false`: 驱动不会添加到getLastError命令中。 |
| journal=true/false | 如果设置为 true, 同步到 journal (在提交到数据库前写入到实体中). 应用于 safe=true | 
| connectTimeoutMS=ms | 可以打开连接的时间。 |
| socketTimeoutMS=ms | 发送和接受sockets的时间。 |
### 数据库：
#### 创建数据库：
MongoDB 创建数据库的语法格式如下：

```
# 如果数据库不存在则创建数据库，否则切换到指定数据库。
use DATABASE_NAME
```
#### 查看所有数据库：

```
show dbs
```
#### 删除数据库：

```
db.dropDatabase()
```
#### 查看所有集合：

```
show tables 或 show collections
```
#### 删除集合：

```
db.COLLECTION_NAME.drop()
```
### 文档：
#### 插入文档：
MongoDB 使用 **insert()** 或 **save()** 方法向集合中插入文档：

```
#如果该集合不在该数据库中， MongoDB 会自动创建该集合并插入文档。
db.COLLECTION_NAME.insert(document)
```
也可以将数据定义为一个变量：

```
> document=({title: 'MongoDB', 
    description: 'MongoDB 是一个 Nosql 数据库',
    myage:25,
    myName:'kevin'
});

> db.COLLECTION_NAME.insert(document)
```
也可以使用 db.COLLECTION_NAME.save(document) 命令。如果不指定 _id 字段 **save()** 方法类似于 **insert()** 方法。如果指定 _id 字段，则会更新该 _id 的数据。
#### 更新文档：
使用 **update()** 方法 更新已存在的文档：

```
db.COLLECTION_NAME.update(
   <query>,
   <update>,
   {
     upsert: <boolean>,
     multi: <boolean>,
     writeConcern: <document>
   }
)
```
参数说明：

- query : update的查询条件，类似sql update查询内where后面的。
- update : update的对象和一些更新的操作符（如$,$inc...）等，也可以理解为sql update查询内set后面的
- upsert : 可选，这个参数的意思是，如果不存在update的记录，是否插入objNew,true为插入，默认是false，不插入。
- multi : 可选，mongodb 默认是false,只更新找到的第一条记录，如果这个参数为true,就把按条件查出来多条记录全部更新。 
- writeConcern :可选，抛出异常的级别。

使用 **save()** 方法通过传入的文档替换已有文档：

```
db.COLLECTION_NAME.save(
   <document>,
   {
     writeConcern: <document>
   }
)
```
参数说明：

- document : 文档数据。
- writeConcern :可选，抛出异常的级别。

异常级别：

- WriteConcern.NONE:没有异常抛出
- WriteConcern.NORMAL:仅抛出网络错误异常，没有服务器错误异常
- WriteConcern.SAFE:抛出网络错误异常、服务器错误异常；并等待服务器完成写操作。
- WriteConcern.MAJORITY: 抛出网络错误异常、服务器错误异常；并等待一个主服务器完成写操作。
- WriteConcern.FSYNC_SAFE: 抛出网络错误异常、服务器错误异常；写操作等待服务器将数据刷新到磁盘。
- WriteConcern.JOURNAL_SAFE:抛出网络错误异常、服务器错误异常；写操作等待服务器提交到磁盘的日志文件。
- WriteConcern.REPLICAS_SAFE:抛出网络错误异常、服务器错误异常；等待至少2台服务器完成写操作。


ps:3.2版本开始，MongoDB提供了以下方法更新文档：

```
#向指定集合更新单个文档
db.COLLECTION_NAME.updateOne();
#向指定集合更新多个文档
db.COLLECTION_NAME.updateMany();
```
#### 删除文档：
使用 **remove()** 方法来删除文档：

```
db.COLLECTION_NAME.remove(
   <query>,
   <justOne>
)

#2.6版本以后对应如下语法格式
db.COLLECTION_NAME.remove(
   <query>,
   {
     justOne: <boolean>,
     writeConcern: <document>
   }
)
```
参数说明：

- query :（可选）删除的文档的条件。
- justOne : （可选）如果设为 true 或 1，则只删除一个文档。
- writeConcern :（可选）抛出异常的级别。

目前官方推荐使用 **deleteOne()** 和 **deleteMany()** 方法来删除文档。
#### 查询文档：
MongoDB使用 **find()** 方法来查询文档：

```
db.COLLECTION_NAME.find(query, projection)
```
具体参数：

- query ：可选，使用查询操作符指定查询条件
- projection ：可选，使用投影操作符指定返回的键。查询时返回文档中所有键值， 只需省略该参数即可（默认省略）。

除此之外还有 **findOne()** 方法，只返回一个文档，要以易读的方式来读取数据，可以使用 **pretty()** 方法。

如果熟悉常规的SQL数据，通过下表可以更好的理解 MongoDB 的条件语句查询：

|操作|格式|范例|RDBMS中的类似语句|
|:-------------:|:-------------|:-------------|:-------------|
|等于| {<key>:<value>} | db.col.find({"by":"菜鸟教程"}).pretty()|where by = '菜鸟教程'|
|  |  |  |  |
|  |  |  |  |
|  |  |  |  |
|  |  |  |  |

