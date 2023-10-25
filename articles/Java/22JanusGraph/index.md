> 图库介绍 https://www.jianshu.com/nb/45863218



# 一、GraphDB

## 1. Introduction

图数据库就是用来存储图结构的数据库。一般的关系型数据库都可以存储图结构，但对于复杂的关系模型，比如微信用户关系网模型构建分析，关系型数据库就显的力不从心。图数据库就是为了解决这类复杂的关系问题而产生的。

NoSQL数据库分类：

- 键值对(key-value)数据库：如Memcache，Redis
- 列簇式数据库：如HBase
- 文档型数据库：如Mongodb
- 图数据库：如Neo4j，JanusGraph

JanusGraph是一个开源的、分布式的、以集群形式存储百亿节点的图数据库。特点：

* 后端数据存储多种选择：Hbase, Cassandra, Google Bigtable等
* 可利用外部如ES, Solr, Lucene等，实现全量文档的搜索功能
* 满足实时的复杂图遍历、满足ACID一致性的OLTP
* 基于spark + hadoop的OLAP
* 使用Gremlin语言查询，与Apache TinkerPop项目兼容

图查询和图计算都是对图的遍历。图数据库主要提供两种与遍历图的方式：OLTP （Online Transaction Processing）和 OLAP （Online Analytical Processing）。

* OLTP联机事务处理 / 图查询；实时返回，涉及少量数据，随机的数据访问，串行运行，用于查询，偏向深度优先的计算引擎，不需要太大的内存；和关系数据库一样，要保证ACID一致性。
* OLAP联机分析处理 / 图计算：长时间运行，涉及几乎整个图，串行地访问数据，并行运行，批量处理，偏向广度优先的计算引擎，需要更大的内存；可以和大数据技术看作一类。
* 图查询指支持对图数据模型的增、删、改、查（CRUD）方法，更关注 OLTP。有的图数据库也继承了少量的图计算能力，但真正的大型系统还是需要单独的计算框架。



## 2. GraphDB Comparison

|                    | Neo4j                    | JanusGraph                     | HugeGraph                                            |
| ------------------ | ------------------------ | ------------------------------ | ---------------------------------------------------- |
| 开源               | 社区版开源               | 开源，兼容Apache Tinkerpop生态 | 开源，兼容Apache Tinkerpop生态（时是百度的开源图库） |
| 图查询语言         | Cypher                   | Gremlin                        | Gremlin                                              |
| 支持数据规模       | 社区版十亿级             | 百亿以上                       | 千亿以上                                             |
| 大规模数据写入功能 | 在线导入慢，脱机导入快   | 较慢                           | 在线导入快                                           |
| 大规模数据查询性能 | 快                       | 较快，不稳定                   | 快，稳定                                             |
| Feature迭代速度    | 趋于完善，新功能上线较慢 | 较少迭代                       | 百度自研，2016年启动，更新快                         |
| 扩展性             | 无法扩展                 | 可扩展                         | 可扩展                                               |
| 内置常用图算法     | 提供了丰富的基本图算法   | 无                             | 内置提供了基本的图算法                               |
| 支持图计算平台集成 | 不支持                   | 支持Spark GraphX等             | 支持Spark GraphX                                     |



# 二、JanusGraph

## 1. JanusGraph & Tinkerpop & Gremlin

**Tinkerpop**是Apache基金会下的一个开源的图数据库与图计算框架（OLTP与OLAP）。

**Gremlin**是Tinkerpop的一个组件，它是一门路径导向语言，用于图操作和图遍历（也称查询语言）。Gremlin Console 和 Gremlin Server 分别提供了控制台和远程执行Gremlin查询语言的方式。Gremlin Server 在 JanusGraph 中被成为 JanusGraph Server。

**JanusGraph**是基于Tinkerpop这个框架来开发的，使用的查询语言也是Gremlin。



## 2. Download & Install

```
1. 从git下载 https://github.com/JanusGraph/janusgraph/releases
	janusgraph-full-0.5.2.zip
2. 解压即用
	unzip janusgraph-full-0.5.2.zip
```



## 3. Interactions

JanusGraph本身就是一组没有执行线程的jar文件。连接和使用JanusGraph数据库有两种基本模式(JanusGraph Embedded、JanusGraph Server)和交互式Shell（Gremlin Console）方式

**JanusGraph Embedded**

```
写在程序里，当lib引入。JanusGraph 作为应用程序的一部分，执行Gremlin，查询同一JVM中的图库。查询执行、JanusGraph缓存和事务处理均发生于此JVM。存储后端，即数据的来源，可能是本地库或远程库。当数据模型大时，很容易OOM，并且耦合性太高，生产上一般不这么搞。
```

**JanusGraph Server**

```
机器上长期运行的服务器进程，允许远程客户端或运行在程序中的逻辑进行JanusGraph调用。提交Gremlin查询到服务器，与JanusGraph实例交互。JanusGraph原生支持Apache TinkerPop的Gremlin Server。

// 1. 编辑conf/gremlin-server/gremlin-server.yaml
graphs: {
  graph: conf/gremlin-server/janusgraph-berkeleyje-es.properties
}
// 2. 将配置配好conf/gremlin-server/janusgraph-berkeleyje-es.properties
gremlin.graph=org.janusgraph.core.JanusGraphFactory  #默认 指明gremlin服务器使用的图工厂实现
storage.backend=berkeleyje  #默认 后端存储BerkeleyDB
storage.directory=db/berkeley  #默认 后端存储数据目录
index.search.backend=elasticsearch  #默认 后端索引ES
index.search.hostname=192.168.200.101,192.168.200.102,192.168.200.103  #自己配 后端索引主机
// 3. 后台启动
nohup sh bin/gremlin-server.sh conf/gremlin-server/gremlin-server.yaml > /dev/null &
```

**Gremlin Console**

```
Gremlin控制台是一个REPL（即交互式shell），与JanusGraph一起打包，可以通过控制台进行JanusGraph 图库的创建（连接）、查询等操作。

本地数据库
    // 前提：已部署好es（看别人）
    // 1. 配置es
    vim conf/janusgraph-berkeleyje-es.properties
    // 2. 启动
    sh bin/gremlin.sh
    // 3. JanusGrphaFactory提供了一组静态方法，通过配置文件作为参数来获取graph实例
    graph = JanusGraphFactory.open('conf/janusgraph-berkeleyje-es.properties')
    // 4. 获取图的遍历对象
    g = graph.traversal()

远程数据库
	// 1. 修改配置conf/remote.yaml
	hosts: [localhost]
	port: 8182
	// 2. 启动
	sh bin/gremlin.sh
	// 3. 连接远程
	:remote connect tinkerpop.server conf/remote.yaml
	// 4. 查询 :> 表示操作的远程
	:> g.V().count()
```



## 3. Benefits

特点概述：

1. 分布式部署，支持集群。
2. 可以存储大图，比如包含数千亿Vertices和edges的图。
3. 支持数千用户实时、并发访问。
4. 集群节点可以线性扩展，以支持更大的图和更多的并发访问用户。
5. 数据分布式存储，支持各种后端存储系统。
7. 通过集成大数据平台Spark Hadoop等，支持全局图数据分析、报表
8. 通过集成ES Lucene等，支持全文搜索
9. 原生集成Apache Tinkerpop图技术栈。包括Gremlin graph query language, Gremlin graph server等。
10. 支持各种可视化工具。

**可扩展**

- 弹性和线性可扩展性，可用于不断增长的数据和用户群
- 数据分发和复制以提高性能和容错能力
- 多数据中心高可用性和热备份

**开源**

所有功能 都是完全免费的。无需购买商业许可证。JanusGraph在Apache 2许可下完全开放源代码。

**事务**

JanusGraph是一个事务数据库，可以支持数千个并发用户实时执行复杂的图遍历。支持ACID和最终的一致性。

**数据存储**

图数据可以存储在：

- Apache Cassandra - 注重在AP上
- Apache HBase - 注重在CP上
- Oracle BerkeleyDB - 一般用于单机本地验证

**检索**

全文搜索等高级搜索功能可以通过以下方式支持

- Elasticsearch - 使用最多
- Apache Solr
- Apache Lucene

**分析**

除了在线事务处理（OLTP）之外，JanusGraph的Apache Spark集成还支持全局图分析（OLAP）。

**TinkerPop**

与 Apache TinkerPop 图栈的本地集成：

- Gremlin graph query language
- Gremlin Server
- Gremlin Console

**适配器**

JanusGraph有不同的第三方存储适配器：

- Aerospike
- DynamoDB
- FoundationDB

**可视化**

JanusGraph支持各种可视化工具，例如Arcade Analytics、Cytoscape、Apache TinkerPop的Gephi插件、Graphexp、Cambridge Intelligence的Key Lines、Linkurious、Tom Sawyer。

# 三、Architectural

![img](https://upload-images.jianshu.io/upload_images/22933483-12f3b310be98fede.png?imageMogr2/auto-orient/strip|imageView2/2/w/1010/format/webp)



## Indices

> JanusGraph中有两种索引：图索引（Graph Index）和 以顶点为中心的索引（Vertex-centric Indexes）

图索引支持过滤索引，就是仅在满足特定条件的边或点建立索引。其中又包括复合索引（Composite Index）和混合索引（Mixed Index）

复合索引不依赖索引后端，仅依赖存储后端。复合索引只能等值查询，也就是说如果只包含部分键的查询都无法走复合索引。如Hbase，将索引的值做一个hash，然后分段存储在多个region server，这样就可以避免查询热点。

混合索引相当于关系型数据库中的常规索引，比复合索引更灵活，其依赖索引后端。混合索引支持全文索引，地理位置索引，范围查询等，默认创建的是全文索引类型。

## Transactional

JanusGraph中的事务是自动开启的，但不会自动commit或rollback。所有的读写操作都是在事务中执行。

JanusGraph采用乐观事务模型，在提交时才会进行冲突检测。

# 四、Gremlin Query Language

## 1. Query

开启本地JanusGraph

```
./gremlin.sh
graph = JanusGraphFactory.open('conf/janusgraph-berkeleyje-es.properties')
g = graph.traversal()
```

查询所有节点 / 查询边

```
g.V().limit(3)
g.E().limit(3)
```

根据id查询节点上的详细信息

```
g.V(4096).valueMap()
g.E('4cl-6co-9hx-39k').valueMap()
```

查询标签 / 属性

```
g.V(4096).label()
g.V(4096).properites()
g.E('4cl-6co-9hx-39k').label()
g.E('4cl-6co-9hx-39k').properites()
```

根据属性的key查询value (标签没有value)

```
g.V(4096).values('name')
g.V(4096).properties('name').value()
```

查询路径

```
// 从一个节点出发，查询出边、进边、所有边
g.V(4096).outE().limit(10)
g.V(4096).inE().limit(10)
g.V(4096).bothE().limit(10)

// 从一个节点出发，查询该节点出边的另一端节点 （path显示出节点、边）
g.V(4096).outE().otherV().valueMap().limit(3)
g.V(4096).outE().otherV().path().limit(3)

// 使用by()语句使查询结果更直观
g.V(4096).outE().otherV().path().by('name').by(label).by('name').limit(3)

// 使用simplePath去掉回路，使用cyclicPath筛选出回路
g.V(4096).outE().otherV().path().simplePath().limit(3)
```

过滤 / 条件查询

```
g.V().hasLabel('god').valueMap()
// 等值
```

终止操作

```
next(n)  // 展示n个
label()
values()
valueMap()
toList()
...
```



## 2. Create Schema

schema相当于关系型数据库中的表结构

开启本地JanusGraph

```
./gremlin.sh
graph = JanusGraphFactory.open('conf/janusgraph-berkeleyje-es.properties')
g = graph.traversal()
```

创建一个新的顶点标签 和 边标签

```
mgmt = graph.openManagement()
animal = mgmt.makeVertexLabel('animal').make()
friend = mgmt.makeEdgeLabel('friend').multiplicity(MULTI).make()
mgmt.commit()
```



## 3. Insert & Delete

开启本地JanusGraph

```
./gremlin.sh
graph = JanusGraphFactory.open('conf/janusgraph-berkeleyje-es.properties')
g = graph.traversal()
```

插入并查询

```
tom = graph.addVertex('human')
tom.property('name', 'Tom')
cat = graph.addVertex('human')
cat.property('name', 'Cat')
g.V().hasLabel('human').out('battled').path().by('name').by('name')
```

修改

```
g.V().has('human', 'name', 'Tom').property('age', '50')
g.V().has('human', 'name', 'Tom').valueMap()
```

删除

```
g.V().has('human', 'name', 'Tom').drop()
```



