



# 图数据库的笔记

##### · 前言

**SQL**：（Structrue Query Language）结构化查询语言

目前除了主流的关系行数据库如：MySQL、ORACLE、SQL SERVER等，还有NoSQL数据库（资料库）

主流的Nosql实际上分4大类别：

| k，v类型的数据库 | redis   |
| ---------------- | ------- |
| 文件存储数据库   | mongoDB |
| 列式存储数据库   | Hbase   |
| 图数据库         | Neo4j等 |

​	**nosql的有优点：**

​		· 适用于半结构化数据（XML、JSON）

​		· 可水平扩展

​		· 无需定义数据库表结构

​		· 可以保证不同栏位内容的资料

​		· 出色的性能

​		· 经济

​	**nosql的缺点：**

​		· 较松散的资料库交易

​		· 不支持join（只支持简单的join）

​		· 没有定义资料完整性规则

​		· 没有SQL或者只支持部分SQL

​		· 使用较高的存储空间（冗余、非正规化）

​		· 使用经验还不够成熟

图数据库主要是用来解决关系行数据库的join查询问题，当查询的深度很深时，MySQL的查询能力很明显不如图数据库，Neo4j是一个原生的图数据库，通过基于图数据结构存储数据，在深度为3时，Mysql的查询能力差不多是Neo4j的1/30；

**初学者向导**

https://neo4j.com/download-center/?ref=web-product-database/#community ## 下载

```
1.Neo4j下载，google搜索neo4j download从社区下载免费版
2.安装，可以安装在windows、linux、macos
3.每天定时备份自己的资料库，作业脚本
关键字：Node
localhost：7474   username：neo4j /password：neo4j -> change pssword= new_pswd
```

**Neo4j的查询语言Cypher**

```SQL
SET 新添属性
DELETE 删除顶点
MERGE 创建节点并建立关系

MERGE| CREATE(c:Country {id: 668 ,name: "China"})
---------
MATCH｜MERGE(c:Country {id: 668 ,name: "China"}) ## 先匹配到,然后再添加属性或者删除node
DELETE c
SET c.ename = "zhongguo" 
-----------
同时可以对接EXCEL倒入数据到库中,并关联已经存在的
.csv文件必须在neo4j的指定目录.../import 下面
LOAD CSV WITH HEADERS FROM "file:///xxxx.csv" as row   ## row 类似于别名，一列一列的读取CSV中的数据
MATCH（c: Country {id: 668, name: "China", ename: "zhongguo"}）
WITH c,row
MERGE （d: City {id: row.id , name: row.name, longitude: toFloat(row.longitude) ,latitude: toFloat(row.latitude)}）
MERGE|CREATE (d) -[r:BELONG{created_by:"shufang"}]->(c)  ## 建立连接，尽量不用CREATE代替MERGE，能用MERGE代替CREATE
```

```sql
关键字： CREATE、MERGE、MATCH、 DELETE、 RETURN、 SET 、 LOAD、CSV、 WITH、HEADERS、 AS...
(:Person) -[:LIVES_IN]-> (:City) -[:PART_OF]-> (:Country)  ## 表示关系

-----------------
--代表无向关系
<- and ->代表有向关系
[...]用来添加详细关系信息 如 MERGE （a）-[r: Like]->(b)
---------------------------------------------------
(keanu:Person:Actor {name:  "Keanu Reeves"} )
-[role:ACTED_IN     {roles: ["Neo"] } ]-> 
(matrix:Movie       {title: "The Matrix"} )

这个模型所包含的意思是：“keanus”这个“人”是一个“演员”，他在电影“the Matrix”中饰演 “Neo”这个角色
---------------------------------------------------
为了减少重复增加模块性，可以用变量接受模块如下：
acted_in = (:Person)-[:ACTED_IN]->(:Movie)  顶点 -[ralation：关系标签 {动作：关系属性}]-> 顶点
其中 acted_in 是一个变量，接收了这个模块通过 MERGE可以创建一个模块
----------
MATCH(C:PERSON) WHERE C.NAME = "WORLD"
MERGE (C)-[]->(B) 
ON CREATE SET A.NAME= "HELLO" 
RETURN C.NAME,B
```

