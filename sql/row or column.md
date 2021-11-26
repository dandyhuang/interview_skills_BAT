# “行式存储”和“列式存储”的区别

### OLTP(on-line transaction processing)和OLAP(On-Line Analytical Processing)

- OLTP(联机事务处理) 是传统关系型数据库的主要应用,用来执行一些基本的、日常的事务处理
   比如数据库记录的增、删、改、查等等

- OLAP(联机分析处理 ) 则是分布式数据库的主要应用, 它对实时性要求不高，但处理的数据量大
   通常应用于复杂的动态报表系统上

### 行式存储和列式存储

> 传统的关系型数据库，如 Oracle、DB2、MySQL、SQL SERVER 等采用行式存储法(Row-based)，在基于行式存储的数据库中， 数据是按照行数据为基础逻辑存储单元进行存储的， 一行中的数据在存储介质中以连续存储形式存在。

> 列式存储(Column-based)是相对于行式存储来说的，新兴的 Hbase、HP Vertica、EMC Greenplum 等分布式数据库均采用列式存储。在基于列式存储的数据库中， 数据是按照列为基础逻辑存储单元进行存储的，一列中的数据在存储介质中以连续存储形式存在。

![image-20211125145826788](/Users/11126518/knowledge/interview_skills_BAT/img/image-row-column.png)
