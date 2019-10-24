



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

​	**nosql的有优点：**（常用的nosql底层都是基于集合的）

​		· 适用于半结构化数据（XML、JSON）

​		· 可水平扩展

​		· 无需定义数据库表结构

​		· 可以保证不同栏位内容的资料

​		· 出色的性能

​		· 经济

​		· 同时支持事务，能够保证更新ACID的机制，类似于其他的RDBM

​	**nosql的缺点：**

​		· 较松散的资料库交易

​		· 不支持join（只支持简单的join）

​		· 没有定义资料完整性规则

​		· 没有SQL或者只支持部分SQL

​		· 使用较高的存储空间（冗余、非正规化）

​		· 使用经验还不够成熟

图数据库主要是用来解决关系行数据库的join查询问题，当查询的深度很深时，MySQL的查询能力很明显不如图数据库，Neo4j是一个原生的图数据库，通过基于图数据结构存储数据，在深度为3时，Mysql的查询能力差不多是Neo4j的1/30，**同时Neo4j支持ETL工具**；

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

```cypher
SET 新添属性
DELETE 删除顶点
MERGE 创建节点并建立关系

MERGE| CREATE(c:Country {id: 668 ,name: "China"})
---------
MATCH｜MERGE(c:Country {id: 668 ,name: "China"}) ## 先匹配到,然后再添加属性或者删除node
DELETE c
SET c.ename = "zhongguo" 
-----------
同时可以对接EXCEL倒入数据到库中,并关联已经存在的 toInteger（） toFloat（） datatime（）
.csv文件必须在neo4j的指定目录.../import 下面
LOAD CSV WITH HEADERS FROM "file:///xxxx.csv" as row   ## row 类似于别名，一列一列的读取CSV中的数据
MATCH（c: Country {id: 668, name: "China", ename: "zhongguo"}）
WITH c,row
MERGE （d: City {id: row.id , name: row.name, longitude: toFloat(row.longitude) ,latitude: toFloat(row.latitude)}）
MERGE|CREATE (d) -[r:BELONG{created_by:"shufang"}]->(c)  ## 建立连接，尽量不用CREATE代替MERGE，能用MERGE代替CREATE
```

```cypher
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

##### · Neo4j整合项目

Neo4j得到合作伙伴，用户和社区贡献者提供的丰富的库，工具，驱动程序和指南生态系统的支持。我们想概述可用的内容并链接到原始资源。我们尝试将重点放在这里的免费解决方案上，并提供指向商业选项的链接。

以下地址涵盖了各种通过CSV文件导数据到NEO4J的使用方法，同时介绍了各种工具，以及Cypher方式的导入：

https://neo4j.com/developer/guide-import-csv/  

`Neo4j`除了支持`CSV`的资料汇入，通过`LOAD CSV WITH HEADERS  FROM  "" AS ROW`，同时还支持以下的项目整合：

- [Neo4j和Apache Spark](spark整合适用于更大的数据集dataSet)

  https://neo4j.com/developer/apache-spark/ ## neo4j(from) -> spark（GraphX） -> neo4j(to)

- [Neo4j和Elastic {Search}](https://neo4j.com/developer/integration/elastic-search)

- [Neo4j和MongoDB](https://neo4j.com/developer/integration/mongodb)

- [Neo4j和Cassandra](https://neo4j.com/developer/integration/cassandra)

- [Neo4j和Docker](https://neo4j.com/developer/integration/docker)



##### · Neo4j与关系型数据比较

```java
SQL 与 Cypher 的比较:
1.SQL需要频繁的join、而Cypher不需要，只需要做简单的MATCH操作WHERE操作就行，代码量更少
2.Cypher相对于SQL查询效率要更高
3.SQL与Cypther同时都支持JDBC的接口，Cypher的JDBC接口代码简单实现如下：
Connection con = DriverManager.getConnection("jdbc:neo4j://localhost:7474/");

String query =
    "MATCH (:Person {name:{1}})-[:EMPLOYEE]-(d:Department) RETURN d.name as dept";
try (PreparedStatement stmt = con.prepareStatement(QUERY)) {
    stmt.setString(1,"John");
    ResultSet rs = stmt.executeQuery();
    while(rs.next()) {
        String department = rs.getString("dept");
        ....
    }
}
```

**SQL Statement**

```SQL
SELECT name FROM Person
LEFT JOIN Person_Department
  ON Person.Id = Person_Department.PersonId
LEFT JOIN Department
  ON Department.Id = Person_Department.DepartmentId
WHERE Department.name = "IT Department"
```

**Cypher Statement**

```cypher
MATCH (p:Person)-[:WORKS_AT]->(d:Dept)   ## 关系，有点类似于scala中的模式匹配
WHERE d.name = "IT Department"    ## filter
RETURN p.name     ## 返回
```



##### 将NoSQL知识转换为图形

- 计算平均收入？询问**关系数据库**。
- 建立购物车？使用**键值存储数据库**。
- 是否存储结构化产品信息？存储为**文档数据库**。
- 描述用户如何从A点到达B点？按照**图数据库**。

下图显示了每种数据库类型如何在测量深度和大小的频谱上叠加。虽然键值存储可以处理大量数据，但它们是为数据的高级视图（低深度）而设计的。图形数据库保留最小化的大小，即使比其他类型的数据库具有更大的数据深度也是如此。其他类型的数据库介于这些范围之间。

![]()



##### Neo4j组件介绍

- Neo4j图形数据库–我们的核心图形数据库，用于存储和检索连接的数据。有[两个版本](https://neo4j.com/licensing/) -社区版和企业版。我们平台中的所有内容都与数据库中存储的数据进行交互。
- [Neo4j Desktop](https://neo4j.com/developer/neo4j-desktop/) –用于管理Neo4j本地实例的应用程序。免费下载包括Neo4j企业版许可证。
- [Neo4j浏览器](https://neo4j.com/developer/neo4j-browser/) –在线浏览器界面，用于查询和查看数据库中的数据。使用Cypher查询语言的基本可视化功能。
- [Neo4j Bloom](https://neo4j.com/bloom/) –业务用户的可视化工具，不需要任何代码或编程技能即可查看和分析数据。[文档](https://neo4j.com/docs/bloom-user-guide/current/)也是我们的文档部分找到。



##### Neo4j ETL 工具

许多人希望将其关系系统中的数据导入Neo4j。开发Neo4j ETL工具是为了使此初始导入变得简单。它从任何关系数据库中提取模式，并允许您将其转换为所需的图模式。然后，它将以批量或在线模式将数据导入图形中。您无需了解Cypher即可使用此工具，它可以处理所有繁重的工作。这使您可以浏览已经知道为图形的数据集。

**ETL-Tool Graph App**、

安装使用指南：https://neo4j.com/developer/neo4j-etl/

```
可用性和安装
ETL-Tool Graph App可以通过 https://install.graphapp.io 安装到Neo4j Desktop中。

创建项目和数据库实例后，需要转到Graph ApplicationsNeo4j Desktop中的选项卡，将https://r.neo4j.com/neo4j-etl-appNeo4j ETL工具的URL 复制并粘贴到安装框中，然后单击Install按钮。
```

ETL工具功能介绍：

- Neo4j Desktop中的Neo4j-ETL Graph App
- 管理多个RDBMS连接
- 从关系数据库中自动提取数据库元数据
- 推导图模型
- 直观地编辑标签，关系类型，属性名称和类型
- 将当前模型可视化为图形
- 坚持映射为json
- 从关系数据库中检索相关的CSV数据
- 运行批量或在线导入
- 将MySQL，PostgreSQL捆绑在一起，允许Neo4j Enterprise使用自定义JDBC驱动程序

![]()



##### APOC（awesome procedures on Cypher ),具体安装如下链接：

https://neo4j.com/developer/neo4j-apoc/

apoc可以用来导入各种数据.csv .json .xml .jdbc .xxx





##### //graph algorithms 图算法

//TODO



##### //graphQL 、Grandstack是一个平台，无缝集成了jscript

​	grand可以将带注释的graphsql转化成单个cypher执行

```java
type Movie {
    title: ID!
    released: Int
    tagline: String

    actors: [Person] @relation(name:"ACTED_IN", direction:IN)

    director: Person @relation(name:"DIRECTED", direction:IN)

    recommendation(first:Int = 3): [Movie]
      @cypher(statement:"MATCH (this)<-[r1:REVIEWED]-(:User)-[r2:REVIEWED]->(reco:Movie)
                         WHERE 3 <= r1.stars <= r2.stars
                         RETURN reco, sum(r2.stars) as rating ORDER BY rating DESC")
}

interface Person {
    name: ID!
    born: Int
}

type Actor extends Person {
    name: ID!
    born: Int

    movies: [Movie] @relation(name:"ACTED_IN")
}

type Director extends Person {
    name: ID!
    born: Int

    movies: [Movie] @relation(name:"DIRECTED")
}

type Mutations {
    directed(movie:ID! director:ID!) : String
      @cypher(statement:"MATCH (m:Movie {title: $movie}), (d:Person {name: $director})
                         MERGE (d)-[:DIRECTED]->(m)")
}
schema {
   mutations: Mutations
}

```

**GrandStack、GraphQL架构原理**：

![](/Users/shufang/Desktop/neo4j_myself/pics/grandstack_architecture.png)

##### Neo4j可视化工具和产品

可视化能够直观的带来价值，所以需要可视化

Neo4j可视化工具：

​	· Neo4j Browser（开发者）

​	· **Neo4j Bloom** （是一种商业许可产品，主要为非开发人员设计的，可以用自然语言进行查询）



##### Neo4j支持java开发

https://neo4j.com/docs/java-reference/3.5/javadocs/ ## javaAPI地址

##### Neo4j‘s javaDriver代码示意

```xml
maven依赖
<dependencies>
    <dependency>
        <groupId>org.neo4j.driver</groupId>
        <artifactId>neo4j-java-driver</artifactId>
        <version>1.5.2</version>
    </dependency>
</dependencies>
```

```java
import org.neo4j.driver.v1.*;

/**
 * @ ClassName HelloWorldExample
 * @ Author shufang
 * @ Descripetion
 * @ Date 2019/10/20 21:34
 * @ Version 1.0
 */

public class HelloWorldExample implements AutoCloseable {
    private final Driver driver;

    public HelloWorldExample(String uri, String user, String password) {
        //1.获取驱动对象
        driver = GraphDatabase.driver(uri, AuthTokens.basic(user, password));
    }

    @Override
    public void close() throws Exception {
        driver.close();
    }


    //创建会话以及事务
    public void printGreeting(final String message) {
        try (Session session = driver.session()) {
            String greeting = session.writeTransaction(new TransactionWork<String>() {
                @Override
                public String execute(Transaction tx) {
                    StatementResult result = tx.run("CREATE (a:Greeting) " +
                                    "SET a.message = $message " +
                                    "RETURN a.message + ', from node ' + id(a)",
                            Values.parameters("message", message));
                    return result.single().get(0).asString();
                }
            });
            System.out.println(greeting);
        }
    }

    public static void main(String... args) throws Exception {
        try (HelloWorldExample greeter = new HelloWorldExample("bolt://localhost:7687", "neo4j", "lanSHU19920725")) {
            greeter.printGreeting("hello, world");
        }
    }
}
```

| Cypher type     | Java type                           |
| :-------------- | :---------------------------------- |
| `String`        | `String`                            |
| `Integer`       | `Long`                              |
| `Float`         | `Double`                            |
| `Boolean`       | `Boolean`                           |
| `Point`         | `org.neo4j.graphdb.spatial.Point`   |
| `Date`          | `java.time.LocalDate`               |
| `Time`          | `java.time.OffsetTime`              |
| `LocalTime`     | `java.time.LocalTime`               |
| `DateTime`      | `java.time.ZonedDateTime`           |
| `LocalDateTime` | `java.time.LocalDateTime`           |
| `Duration`      | `java.time.temporal.TemporalAmount` |
| `Node`          | `org.neo4j.graphdb.Node`            |
| `Relationship`  | `org.neo4j.graphdb.Relationship`    |
| `Path`          | `org.neo4j.graphdb.Path`            |

##### neo4j性能调优

https://neo4j.com/docs/operations-manual/current/performance/

##### Neo4j拓展知识

https://neo4j.com/docs/developer-manual/3.4/extending-neo4j/procedures/

//TODO！



##### Neo4j与spark的GraphX图分析系统的整合

**GraphX**特点：离线计算、批量处理，基于同步的BSP模型（Bulk Sychronous Parallel Computing model，整体同步并行计算模型），这样能保证提升数据处理的吞吐量和规模，但是在速度方面会稍微逊色一筹，目前大规模的图处理框架还有基于**MPI**模型的异步图计算模型**GraphLab** 和同样基于BSP模型的**Giraph**等

除了Neo4j可以与GraphX组合使用外，还有另外的Janusgraph分布式图数据库可以与GraphX进行组合使用，但是janusGraph不是原生的图数据库，基于Hbase存储，这2个数据库都能作为GraphX的持久层。可以存储大规模的图数据。

**Graph**内部提供了三种RDD来对一个有向多重图的属性进行描述：

​	· VertextRDD extends RDD

​	· EdgeEDD[ED,VD] extends RDD

​	· Triplet



