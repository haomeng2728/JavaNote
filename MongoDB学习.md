# 一、MongoDB是什么？

MongoDB 是由C++语言编写的，是一个基于分布式文件存储的开源数据库系统。 在高负载的情况下，添加更多的节点，可以保证服务器性能。 MongoDB 旨在为WEB应用提供可扩展的高性能数据存储解决方案。 MongoDB 将数据存储为一个文档，数据结构由键值(key=>value)对组成。MongoDB 文档类似于 JSON 对象。字段值可以包含其他文档，数组及文档数组。

# 二、MongoDB解决了什么问题？

（1）MongoDB提出的是文档、集合的概念，使用BSON（类JSON）作为其数据模型结构，其结构是面向对象的而不是二维表，存储一个用户在MongoDB中是这样子的。

``` json
{	
	username:'123',
	password:'123'
｝
```

使用这样的数据模型，使得MongoDB能在生产环境中提供高读写的能力，吞吐量较于mysql等SQL数据库大大增强。

（2）易伸缩，自动故障转移。易伸缩指的是提供了分片能力，能对数据集进行分片，数据的存储压力分摊给多台服务器。自动故障转移是副本集的概念，MongoDB能检测主节点是否存活，当失活时能自动提升从节点为主节点，达到故障转移。

（3）数据模型因为是面向对象的，所以可以表示丰富

# 三、MongoDB怎么用？

1、在pom中添加MongoDB依赖

2、在application.yml中添加MongoDB配置信息

3、添加文档对象，对象类上添加@Document，文档对象的ID域添加@Id注解，需要检索的字段添加@Indexed注解

4、添加MemberReadHistoryRepository接口用于操作Mongodb，继承于MongoRepository接口，这样就拥有了一些基本的Mongodb数据操作方法，同时定义了一个衍生查询方法

5、添加service serviceImpl controller方法

# 四、与其他类似工具对比？

|     类型      |                          代表                          |                             特点                             |
| :-----------: | :----------------------------------------------------: | :----------------------------------------------------------: |
|    列存储     |               Hbase/Cassandra/Hypertable               | 顾名思义，是按列存储数据的。最大的特点是方便存储结构化和半结构化数据，方便做数据压缩，对针对某一列或者某几列的查询有非常大的IO优势。 |
|   文档存储    |                    MongoD/ CouchDB                     | 文档存储一般用类似json的格式存储，存储的内容是文档型的。这样也就有机会对某些字段建立索引，实现关系数据库的某些功能。 |
| key-value存储 | Tokyo Cabinet / Tyrant /Berkeley DB / MemcacheDB/Redis | 可以通过key快速查询到其value。一般来说，存储不管value的格式，照单全收。（Redis包含了其他功能） |
|    图存储     |                    Neo4J / FlockDB                     | 图形关系的最佳存储。使用传统关系数据库来解决的话性能低下，而且设计使用不方便 |
|   对象存储    |                     db4o / Versant                     | 通过类似面向对象语言的语法操作数据库，通过对象的方式存取数据 |
|   xml数据库   |                Berkeley DB XML /Base X                 | 高效的存储XML数据，并支持XML的内部查询语法，比如XQuery,Xpath |

# 参考文章

https://blog.csdn.net/hayre/article/details/80628431

https://www.runoob.com/mongodb/nosql.html