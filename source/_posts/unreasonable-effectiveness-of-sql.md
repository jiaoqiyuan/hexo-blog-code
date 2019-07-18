---
title: 为什么 SQL 如此成功？
date: 2019-05-10 07:42:07
tags:
categories: 翻译
photos: https://github.com/jiaoqiyuan/pics/raw/master/blog/translate/effectiveness-sql.png
---

信息来源：[阮一峰每周分享](http://www.ruanyifeng.com/blog/2019/05/weekly-issue-55.html)

文章来源：[The Unreasonable Effectiveness of SQL](https://blog.couchbase.com/unreasonable-effectiveness-of-sql/)

---

Two score and five years ago two young IBM researchers brought forth on databases, a new language, conceived in relational, dedicated to the proposition that data can be manipulated declaratively and easily.   In the years since Don Chamberlin and Ramond Boyce published [SEQUEL: A Structured English Query Language][1] relational model and SQL have been extended and adapted to a significant number of technologies: OLTP, OLAP, object databases, object-relational databases, and even NoSQL. SQL has inspired query language design for non-relational databases: SQL for object databases, SQL for object-relational, SQL for XML, SQL for spatial, SQL for search, SQL for JSON, SQL for timeseries, SQL for streams and so on.  Every BI tool interacts with the data using variety of SQL. In fact, SQL is the most [successful 4th generation language][2].

45年前，两个 IBM 研究人员提出了数据库的概念，一种基于关系构建的新型语言，专注于数据易用和声明式操作。在 Don Chamberlin 和 Ramond Boyce 发表 [SEQUEL: A Structured English Query Language][1] 论文后的这些年间，关系模型和 SQL 已经被扩展并适用与大量技术： OLTP、OLAP、对象数据库、对象关系数据库以及 NoSQL。SQL 启发了非关系数据库的查询语言设计：对象数据库 SQL、对象关系数据库 SQL、XML SQL、空间 SQL、JSON SQL、时序 SQL，流式 SQL 等。每种与数据交互的 BI 工具都在适用各种各样的 SQL。实际上，SQL 是成功的四中通用语言之一。

<!--more-->

**“SQL is a device whose mystery is only exceeded by its power.”  Lukas Eder**

**"SQL 是一种只有它自己才能超越的神秘存在" Lukas Eder 说。**

As Don said recently, SQL was based on the foundation of [relational algebra][3] with the goal to make it easier by providing an English like query language with the following goals:

正如 Don 最近说的，SQL 基于 [关系代数][3]，目的是通过提供一种具有如下功能的类英语的查询语言来使得关系查询变得更加简单。

- A declarative language and processing (instead of procedural)

    一种声明性处理数据的语言（不是编程语言）。

- Make the language composable to help write complex queries easily

    语言可组合以帮助复杂查询的书写变得简单。

- Work with the relational model developed by [Edger F Codd][4].

    使用 [Edger F Codd][4] 开发的关系模型。

While the big data tried to compliment and replace relational systems for data warehousing, they tried to speak the same language: SQL.  Hive, Impala, drill, BigSQL all speak SQL inspired language, optimizer and execute similar to the MPP execution of SQL. They’re also adding new SQL features regularly. All this on every type of data store and model you think of. The separation of the data storage formats, data model and query processing in SQL has yielded significant benefits.  In the forty-five years since SQL was introduced, many databases have come and gone; many data processing have come and gone. Some in the NoSQL movement implied, even if inadvertently, the death of SQL and SQL databases. SQL camp has taken this in stride and [Don Chamberlin recently said][5]: “When a language is so well recognized that other languages start defining themselves as not that one, it must be doing pretty good.”

虽然大数据试图补充和替换数据仓库的关系系统，但他们还是使用了同样的语言：SQL。Hive，Impala，drill，BigSQL 这些使用 SQL 语言的工具启发了语言、优化器并执行类似于 MPP 执行的 SQL。他们还定期添加新的 SQL 功能。所有这些都在您想到的每种类型的数据存储和模型上。数据存储格式的分离、数据模型以及 SQL 中的查询处理已经产生了显著的效益。在 SQL 诞生的 45 年中，许多数据库诞生并陨落；许多数据处理方式来了又走。NoSQL运动中的一些人即使在不经意间暗示 SQL 和 SQL 数据库会死亡。SQL 阵营也不遑多让，而且 Don Chamberlin 最近还说“当一种语言如此被认可时，以至于其他语言也定义自己属于这种语言而不是其他语言，那么这种语言一定是相当不错的。”

Databases on the other hand simply went No-SQL.  While the current definition of this is “Not Only SQL”, the original approaches were to go without SQL and try alternative languages and frameworks like map-reduce.  A decade later, every popular NoSQL databases have a variation of SQL: N1QL in Couchbase, CQL in Cassandra, ElasticSearch SQL in Elastic. You say, “MongoDB doesn’t have SQL”.  I say, “[Squint][6]! You’ll see a very simplistic SQL implementation.” By using a simplistic, somewhat procedural and ad-hoc design in MongoDB, queries loose composability, optimizations and many of the innovations done with SQL.

另一方面，数据库就简单地朝着 No-SQL 发展。虽然目前定义的是 "Not Only SQL"，但是最初的设计是不使用 SQL 的，而是尝试使用像 map-reduce 一样的框架和语言。十年后，每个流行的NoSQL数据库都有SQL的变体：Couchbase 的 N1QL 、 Cassandra 的 CQL 、 Elastic 的 ElasticSearch SQL。你可能会说“ MongoDB 没有使用 SQL 啊”。我会说，“[Squint][6]! 你将看到一个非常简单的SQL实现。” 通过在MongoDB中使用简单、涉及一些编程和特殊的设计，可以查询松散的可组合性、优化以及使用SQL完成的许多创新。

While the relational model has been very successful, databases support a variety of data models: JSON, Graph, XML, Timeseries, spatial, wide-column, columnar, document and more.  Most, if not all of these databases have their version of SQL. N1QL is SQL for JSON; SQL/XML, SQL from InfluxDB, SQL/Spatial, CQL in Cassandra, etc. Even the NoSQL databases have implemented SQL and SQL inspired query languages.   Even in the new cool “data science” world, [SQL skills are highly recommended][7].  Lukas Eder makes this point in his must-see talks.  See the links for his talks in the reference section below.

虽然关系模型已经非常成功了，但数据库支持各种各样的数据模型：JSON, Graph, XML, Timeseries, spatial, wide-column, columnar, document 等等。大多数数据库都有他们自己的 SQL 版本。N1QL是JSON的SQL；SQL/XML, SQL来自于 InfluxDB，SQL/Spatial，Cassandra 中的 CQL等等。甚至数 NoSQL 数据库都实现了 SQL，而且 SQL 给查询语言带来了很大启发。即使是在新潮的 "数据科学" 的世界中， [强烈推荐掌握 SQL 技能][7]。Lukas Eder 在他必看的谈话中阐述了这一点。从下面的连接中可以看到他的讲话。

**Now, there are more SQL projects in NOSQL databases than SQL databases.**

**现在，NOSQL 数据库中的 SQL 项目比 SQL 数据库的还多。**

| Data models/formats |	SQL implementation |
|:-------------------:|:------------------:|
| JSON |	Couchbase N1QL: SQL for JSON |
| Wide | column	Cassandra CQL |
| Hadoop/Big | Data	Hive, Impala, Drill, BigSQL |
| Timeseries | Influxdb |
| Graph |	SQL Graph Database, Oracle Graph |
| NoSQL database |	Apache Phoenix
| Spatial |	Oracle Spatial
| Search |	Elastic SQL

## WHY IS SQL SO SUCCESSFUL? 为什么SQL如此成功？

1. Declarative:  You declare the output and the query engines figure out the optimal way to execute the query.  The optimizer, especially the cost based optimizer invented [Pat Selinger, et al][8] in 1979, has helped to continuously improve the performance.   This set a high bar for every new entrant to live up to. A [recent paper on Apache Hive][9] is an example of complexity and sophistication involved.

    声明：你声明输出，查询引擎找出执行查询的最佳方式。优化器，尤其是基于成本的优化器由 Pat Selinger 等人于 1979 年发明，一直在提高查询性能。这为每个新进入者提供了一个很高的标准。[最近 Apache Hive 上的一个论文][9]就是关于复杂和诡辩的一个例子。

2. SQL was used not just for “querying”, but to update the data, doing transactions.  Stored procedures, UDFs have extended the reach by combining procedural language with the declarative SQL.

    SQL 不仅仅用来 "查询"，而且用来跟新数据，做一些转换操作。存储过程，UDF 通过将过程语言与声明性 SQL 结合起来扩展了 SQL 边界。

3. SQL has been malleable. It has been standardized many times, each time adding a book full of features, a store full of syntaxes, and a dictionary full of keywords. For sure, not all SQL is the same.  Even the traditional SQL implementations on RDBMS are not exactly compatible unless you’re careful to write your SQL to be compatible.  Through all these, the original spirit of SQL has remained.  An example of SQL lending itself to evolution is [SQL++][10]. Don Chamberlin and Prof. Mike Carey [discuss the need for supporting complex data models][11], making the data in JSON easily accessible to users and developers. Don’s book [SQL++ For SQL Users][12]: A Tutorial teaches you the recent developments in SQL++, language designed for data processing on the flexible JSON data model, but in a SQL compatible way.

    SQL 具有可塑性。它已被标准化了很多次，每次都添加一整本数的特性，一个充满语法的商店，一个满是单词的字典。当然，并非所有SQL都是相同的。即使是RDBMS上的传统SQL实现也不完全兼容，除非您小心地将SQL编写为兼容。即使这样，SQL 的原始精神仍然存在。SQL 使自己进化的一个例子就是 [SQL++][10]。Don Chamberlin 和 Prof. Mike Carey [讨论是否需要支持复杂的数据模型][11]，让 JSON 中的数据能够易于被用户和开发人员获取。Don 的教你 SQL 最新发展的书 [SQL++ For SQL Users][12]，这是为灵活的 JSON 数据模型上的数据处理而设计的语言，但是以 SQL 兼容的方式。

4. SQL, like the English language it borrowed from, has been open to new ideas and extensions for new data types, access methods, use cases.

  SQL 就像它借用的英语一样，对新数据类型，访问方法和用例的新思想和扩展持开放态度。

5. SQL’s independence from the data representation has allowed itself to be used on non-relational data: CSV, JSON, and all the big data format.  Some folks conflate rigidity of relational model representation to the rigidity of SQL. In fact, on a given schema, SQL allows you to select-join-group-aggregate-project any of the data formats.

  SQL 与数据表示的独立性使其自身可以用于非关系数据：CSV，JSON和所有大数据格式。有些人将关系模型表示的死板与 SQL 的死板混为一谈。实际上，在给定的模式中，SQL 允许你 select-join-group-aggregate-project 任何数据格式。

## Evaluating SQL Support 评估 SQL 支持:

Now that SQL is everywhere you need to do the due diligence on the level of support.

既然 SQL 无处不在，你需要在支持级别上进行详细的考量。

1. Find out the workload characteristics and goal of each workload.  E.g. interactive applications or interactive analytics or batch analytics or BI workload, etc.

    找出每个工作负载的工作负载特征和目标。例如。交互式应用程序或交互式分析或批量分析或BI工作负载等。

2. Statements supported reflects operational capabilities.

    支持的声明反映了业务能力。

3. Language capabilities in terms of expressions (scalar, aggregate, boolean), joins (inner, left/right/full outer), subqueries, derived tables, sorting and pagination (LIMIT/OFFSET).

    表达式（标量，聚合，布尔），连接（内部，左/右/全外），子查询，派生表，排序和分页（LIMIT / OFFSET）方面的语言功能。

4. Indexing:  SQL without the right indexes is just a Turing machine prototype.

    索引：没有正确索引的SQL只是图灵机的原型。

5. Optimizer: Query rewrite, choosing the right access path, creating the optimal query execution path is what makes SQL a successful 4GL.  Some have a rule-based optimizer, some have a cost-based optimizer, some others have both.  Evaluating the quality of the optimizer is critical.  Typical benchmarks (TPC-C, TPC-DS, YCSB, YCSB-JSON) won’t help you here.

    优化器：查询重写，选择正确的访问路径，创建最佳查询执行路径是SQL成功成为 4GL 的原因。有些具有基于规则的优化器，有些具有基于成本的优化器，有些具有基于成本的优化器。评估优化器的质量至关重要。典型的基准测试（TPC-C，TPC-DS，YCSB，YCSB-JSON）对你没有帮助。

6. As the saying goes: ” There are three things important in databases: performance, performance, and performance”. It’s important to measure the performance of your workload.  YCSB and the extended YCSB-JSON will make this evaluation easier.

    俗话说：“数据库中有三件重要的东西：性能，性能和性能”。衡量工作负载的性能非常重要。YCSB 和扩展的 YCSB-JSON 将使评估更容易。

7. SDKs: Rich SDKs and language support speed up your development.

    SDK：丰富的SDK和语言支持可加快您的开发速度。

8. BI tool support: For large data analysis, support from BI tools usually via standard database connectivity drivers is important.

    BI工具支持：对于大数据分析，BI工具通常通过标准数据库连接驱动程序提供支持非常重要。

Gerald Sangudi, the creator of N1QL, once remarked SQL is successful because it represents the fundamental operations of data processing. SQL supports a rich set of operations select-join-nest-unnest-group-aggregate-having-window-order-paginate-set-ops.  Is this how we (or machines) think when specifying data operations?  While that remains to be seen, other languages like python and java are adding operators for these operations on data. Maybe, others will follow suit.  SQL has gone where the relational model didn’t.  It’s not an exaggeration to say:

N1QL 的创建者 Gerald Sangudi 曾经说过 SQL 是成功的，因为它代表了数据处理的基本操作。SQL支持一组丰富的操作select-join-nest-unnest-group-aggregate-having-window-order-paginate-set-ops。这是我们（或机器）在指定数据操作时的想法吗？虽然还有待观察，但其他语言如python和java正在为数据上的这些操作添加运算符。也许，其他人也会效仿。 SQL已经走到了关系模型没有的地方。可以毫不夸张地说：

## SQL is dead.  Long live SQL.  SQL已经死了。 SQL万岁。

## References 参考:

[SQL++ For SQL Users: A Tutorial](https://www.amazon.com/SQL-Users-Tutorial-Don-Chamberlin/dp/0692184503/)

[Lukas Eder – 2000 Lines of Java? Or 50 Lines of SQL? The Choice is Yours!](https://www.youtube.com/watch?v=S3JbAEd5u_c)

[Ten SQL Tricks that You Didn’t Think Were Possible](https://www.youtube.com/watch?v=mgipNdAgQ3o)

[How Modern SQL Databases Come up with Algorithms that You Would Have Never Dreamed Of by Lukas Eder](https://www.youtube.com/watch?v=wTPGW1PNy_Y)

[SQL History](https://youtu.be/LAlDe1w7wxc)

[Eugene Wigner: The unreasonable effectiveness of mathematics in the natural sciences](https://www.maths.ed.ac.uk/~v1ranick/papers/wigner.pdf)

[Alon Halevy, Peter Norvig, and Fernando Pereira: The Unreasonable Effectiveness of Data](https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/35179.pdf)

[Difference between 3GL and 4GL](https://www.reference.com/technology/difference-between-3gls-4gls-8826a78c1e34c2e)


[1]: https://researcher.watson.ibm.com/researcher/files/us-dchamber/sequel-1974.pdf
[2]: https://www.youtube.com/watch?v=wTPGW1PNy_Y
[3]: https://en.wikipedia.org/wiki/Relational_algebra
[4]: https://en.wikipedia.org/wiki/Edgar_F._Codd
[5]: https://youtu.be/-U_UjqnhMBI?t=3585
[6]: https://docs.mongodb.com/manual/reference/sql-comparison/
[7]: https://www.kdnuggets.com/2018/05/simplilearn-9-must-have-skills-data-scientist.html
[8]: https://people.eecs.berkeley.edu/~brewer/cs262/3-selinger79.pdf
[9]: https://arxiv.org/pdf/1903.10970.pdf
[10]: http://db.ucsd.edu/wp-content/uploads/pdfs/375.pdf
[11]: https://youtu.be/LAlDe1w7wxc
[12]: https://www.amazon.com/SQL-Users-Tutorial-Don-Chamberlin/dp/0692184503/ref=sr_1_2?keywords=n1ql&qid=1554186444&s=gateway&sr=8-2