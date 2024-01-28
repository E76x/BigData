# Hive

## 1.简介

Apache Hive**是一个建立在Hadoop之上的数据仓库系统**，提供了类似于SQL的查询语言（称为HiveQL）来进行大规模数据的分析。它是Hadoop生态系统的一部分，旨在简化数据处理、查询和分析任务。

## 2.优势&特点

- 提供简单和优化的模型，编码量少于mapreduce(只需要写SQL，底层会自动转化为MapReduce)
- 支持在不同计算框架上运行
- 支持在Hdfs&Hbase上进行临时的查询
- 支持自定义函数，脚本，格式
- 成熟的JDBC&ODBC用于ETL和BI工具
- 适合做数据的批处理，离线处理

## 3.Hive元数据管理

Hive元数据管理是指Hive对其数据仓库中的元数据进行管理和维护的过程。元数据是描述数据的数据，它包含了关于存储在Hive中的表、分区、列、数据类型等信息的描述。以下是Hive元数据管理的一些关键方面：

1. **元数据存储：** **Hive的元数据通常存储在关系型数据库中**，例如MySQL或Derby。这些数据库包含了有关Hive中各种对象（表、分区等）及其属性的信息。
2. **元数据信息：** Hive元数据包括但不限于以下信息：
   - 表定义：包括表名、列名、列的数据类型等。
   - 分区信息：对于分区表，元数据包括分区的键、值等。
   - 表的存储格式：描述表数据的存储格式，如文本、Parquet等。
   - 表的位置：指定表数据在HDFS中的存储位置。

## 4.Hive体系架构

![image-20240125163901445](C:\Users\Q27\AppData\Roaming\Typora\typora-user-images\image-20240125163901445.png)

## 5.Hive Tables

### 5.1外部表和内部表

|                      | 外部表                                                       | 内部表                                                       |
| -------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 数据管理             | 外部表的数据不由Hive管理，而是由外部存储系统（通常是Hadoop分布式文件系统，HDFS）管理。 | 内部表的数据由Hive完全管理，包括元数据和实际数据。           |
| 元数据和数据生命周期 | 删除外部表时，只会删除Hive中的元数据，而不会删除实际存储在外部存储系统中的数据。数据的生命周期不受Hive的控制。 | 内部表的元数据和数据生命周期由Hive完全控制。删除内部表时，会删除元数据以及存储在HDFS中的实际数据。 |
| 表定义               | 外部表的定义与内部表类似，但使用 `EXTERNAL` 关键字来声明。**数据保存在location关键字指定的HDFS路径中**。 | 内部表的定义不需要使用 `EXTERNAL` 关键字。使用 `Managed` 关键字创建（可以省略） |
| 使用场景             | 当**数据需要被多个系统或工具使用时**，使用外部表，以便多个系统可以共享相同的数据。当需要保留数据，即使Hive中的元数据被删除时，使用外部表。 | 当数据仅用于Hive，不需要共享给其他系统时，使用内部表。 当希望Hive完全管理数据的生命周期、备份和恢复时，使用内部表。 |

```hive

-- 外部表
CREATE EXTERNAL TABLE external_table (
  id INT,
  name STRING
)
LOCATION '/path/to/external_data';

-- 内部表
CREATE TABLE internal_table (
  id INT,
  name STRING
);

```

### 5.2分区表

#### 1.为什么要分区？

- **提高查询性能：** 分区可以大幅提高查询性能，特别是在处理大量数据时。通过在查询中仅扫描特定分区，可以减少需要读取的数据量，从而提高查询效率。
- **简化数据管理：** 分区使得数据的组织更加清晰，可以按照某个逻辑上的分类对数据进行组织。这样一来，可以更轻松地进行数据的管理、维护和备份。
- **降低查询成本：** 对于只关心特定分区的查询，分区可以降低查询成本，提高系统的整体性能。尤其在数据量庞大的情况下，只查询关心的分区可以减少资源消耗。
- **增强数据过滤：** 分区提供了更精确的数据过滤。通过在查询中指定特定分区，可以迅速缩小检索范围，加速查询速度。
- **优化数据加载：** 分区可以帮助优化数据加载过程。在使用静态或动态分区时，可以只加载新数据而无需重新加载整个表，从而提高加载效率。
- **支持并行处理：** 分区可以支持并行处理，因为不同分区的数据可以在不同的计算节点上并行处理，提高了计算效率。
- **改善数据仓库性能：** 对于大型数据仓库系统，分区可以使其更具扩展性，支持更多的数据和更多的查询请求。
- **方便数据维护：** 在静态分区的情况下，可以更方便地进行数据维护，例如，按照时间维度进行分区，可以轻松地管理历史数据。

#### 2.静态分区

**定义分区：** 静态分区是在创建表时明确指定分区值的方式，通过在`PARTITIONED BY`子句中列出分区键。

```sql
CREATE TABLE static_partitioned_table (
  id INT,
  name STRING
)
PARTITIONED BY (partition_col INT);
```

**管理分区**： **静态分区需要手动管理**，包括创建、删除和维护分区。

**插入数据时指定分区：** 在向静态分区表中插入数据时，需要明确指定插入数据的分区。

```sql
INSERT INTO static_partitioned_table PARTITION (partition_col=1) VALUES (1, 'John');
```

**加载数据时指定分区**

```sql
load data local inpath /root/hdp/hive_stage/data.txt into table static_partitioned_table 
partition(partition_col=1);
```

**删除数据时指定分区**

```sql
alter table static_partitioned_table drop if exists partition (partition_col=1);
```

**手动增加分区**

```sql
alter table static_partitioned_table add partition(partition_col=2);
```

#### 3.动态分区

**插入数据时自动确定分区：** 动态分区是在插入数据时自动确定分区的方式。Hive会根据插入语句中的分区键值动态创建和管理分区。

```sql
INSERT INTO TABLE dynamic_partitioned_table PARTITION (partition_col) VALUES (1, 'John');
```

**开启动态分区：** 需要在Hive中启用动态分区，可以通过设置属性 `hive.exec.dynamic.partition` 和 `hive.exec.dynamic.partition.mode` 来实现。

```sql
SET hive.exec.dynamic.partition=true;
SET hive.exec.dynamic.partition.mode=nonstrict;
```

**分区列的位置：** 动态分区的分区列通常位于插入语句的最后。

#### 4.如何选择动静态

**静态分区：**

- 适用于分区值相对稳定，不经常变化的情况。
- 对于分区值较少，且在创建表时已知的情况，静态分区是一种较好的选择。

**动态分区：**

- 适用于分区值动态变化的情况，无需手动管理分区。
- 当分区键的取值范围较大或经常变动时，动态分区提供了更灵活的管理方式

> **在原始数据比较多的情况下，没有被分区这个时候需要使用动态分区，后面生成的数据使用静**
> **态分区就可以了**

### 5.3桶表

> **这个用的不是很多，但是面试会问到。**

1.**桶的概念：** 桶是表的分区的一种形式，它将表的数据分成固定数量的桶，每个桶包含一部分数据。桶的目的是将数据划分成更小的块，以便提高查询性能。

2.**创建桶表：** 在创建桶表时，需要指定表的桶数和桶的列。桶数表示表将被分成的桶的数量，桶的列是用于确定桶的列。

```sql
CREATE TABLE bucketed_table (
  id INT,
  name STRING
)
CLUSTERED BY (id) INTO 4 BUCKETS;
```

上述例子中，`CLUSTERED BY (id) INTO 4 BUCKETS` 指定了根据列 `id` 进行分桶，分成 4 个桶。

3.**插入数据到桶表：** 插入数据时，**Hive会自动将数据分散到不同的桶**中，**基于桶列的散列值**。

```sql
INSERT INTO TABLE bucketed_table VALUES (1, 'John'), (2, 'Alice');
```

4.**查询桶表：** 查询桶表时，Hive可以利用桶的结构，只扫描涉及的桶，提高查询效率。

```sql
SELECT * FROM bucketed_table WHERE id = 1;
```

**注意事项：**

- 桶表通常适用于那些**经常根据某列进行等值查询**的场景。
- 桶表的桶数应该谨慎选择，通常与集群中的核数相匹配，以确保更好的并行性。

**桶表的优势：**

- **查询性能提升：** 桶表可以显著提高查询性能，特别是在等值查询时，因为Hive只需扫描涉及的桶，而不是整个表。
- **更好的并行性：** 桶表的桶数可以与集群中的核数相匹配，提高并行性，加速查询。

**使用场景：**

- 桶表适用于大型数据集，并且查询中经常使用等值条件过滤的情况。
- 适用于需要优化查询性能并提高并行性的场景。

## 6.严格模式

Hive 严格模式是一种配置选项，它对 Hive 查询语句的解析和执行进行了一些限制，以提高查询的质量和避免一些潜在的问题。

**严格模式的启用：** 通过设置 Hive 配置选项 `hive.mapred.mode` 为 `strict` 来启用严格模式。

```sql
SET hive.mapred.mode=strict;
```

1.查询分区表时，必须在WHERE语句中包含分区字段的过滤条件，以限制数据范围，防止扫描所有分区。这是因为分区表通常包含大量数据，而**没有分区过滤条件的查询可能会消耗大量资源**。 示例：

```sql
SELECT DISTINCT(planner_id) FROM fracture_ins WHERE planner_id=5;
报错:
FAILED: Error in semantic analysis: No Partition Predicate Found for Alias "fracture_ins" Table "fracture_ins"
```

通过在WHERE语句中增加分区过滤条件，可以解决这个问题，例如：

```sql
SELECT DISTINCT(planner_id) FROM fracture_ins WHERE planner_id=5 AND hit_date=20120101;
```

2.对于带有ORDER BY的查询，必须添加LIMIT语句以防止在排序过程中消耗大量资源。 示例：

```sql
SELECT * FROM fracture_ins WHERE hit_date>2012 ORDER BY planner_id;
报错:
FAILED: Error in semantic analysis: line 1:56 In strict mode, limit must be specified if ORDER BY is present planner_id
```

通过添加LIMIT语句，可以解决这个问题，例如：

```sql
SELECT * FROM fracture_ins WHERE hit_date > 2012 ORDER BY planner_id LIMIT 10;
```

> **因为orderby为了执行排序**
> **过程会将所有的结果分发到同一个reducer中进行处理，用户增加这个limit语句可以防止**
> **reducer额外执行很长一段时间。**

3.在执行JOIN查询时，应使用ON语句而不是WHERE语句，以防止产生笛卡尔积。如果表足够大，笛卡尔积可能导致不可控的情况。 示例：

```sql
SELECT * FROM fracture_act JOIN fracture_ads;
报错:
Error in semantic analysis: In strict mode, cartesian product is not allowed. If you really want to perform the operation, +set hive.mapred.mode=nonstrict+
```

正确的使用JOIN和ON语句的查询示例：

```sql
 SELECT * FROM fracture_act JOIN fracture_ads ON (fracture_act.planner_id = fracture_ads.planner_id);
```

> Hive无法像关系型数据库那样**对不使用`ON`语句而是使用`WHERE`语句的笛卡尔积查询进行高效的优化**。这可能导致在大表上执行时出现不可控的性能问题。

## 7.CTAS&CTE

在 Hive 中，有两个高阶语句与建表相关：CTAS（Create Table As Select）和 CTE（Common Table Expressions）。

**CTAS (Create Table As Select):**

- **定义：** CTAS 是一种语句，允许用户在创建表的同时执行 SELECT 查询，并将查询结果插入到新创建的表中。
- **示例：**

- ```sql
  CREATE TABLE new_table AS
  SELECT column1, column2
  FROM existing_table
  WHERE condition;
  ```

- **作用：** CTAS 语句用于方便地创建新表并将某个查询的结果存储到新表中。它可以包含条件、聚合等查询操作。

**CTE (Common Table Expressions):**

- **定义：** CTE 是一种在查询中定义临时表达式的语法结构。它使用 WITH 子句来定义一个临时查询结果，并在查询中引用这个临时查询结果。
- **示例：**

```sql
WITH cte_name AS (
  SELECT column1, column2
  FROM existing_table
  WHERE condition
)
SELECT * FROM cte_name;
```

**作用：** CTE 主要用于提高查询的可读性和可维护性，将复杂的查询逻辑分解成更简单的部分，并在查询中引用这些部分。

## 8.Hive视图

Hive视图提供了一种逻辑结构，通过隐藏虚拟表中的子查询、连接和函数来简化查询。然而，**它们不存储数据，也不能物化**。一旦创建，**视图的架构立即冻结。如果基础表被删除或更改，查询视图将失败**。**视图是只读**的，不能用作LOAD、INSERT或ALTER操作的目标。

1.**创建视图：**

```sql
CREATE VIEW view_name AS
SELECT column1, column2
FROM existing_table
WHERE condition;
```

使用 `CREATE VIEW` 语句可以创建一个视图，指定视图的名称、选择查询的列以及查询的条件。

2.**查询视图：**

```sql
SELECT * FROM view_name;
```

查询视图时，可以像查询表一样使用 `SELECT` 语句，从而检索视图中的数据。

3.**更新视图：**

Hive 中的视图是虚拟的，不能直接进行插入、更新或删除等修改操作。如果需要更新视图，通常需要重新创建视图。

4.**删除视图：**

```sql
DROP VIEW IF EXISTS view_name;
```

使用 `DROP VIEW` 语句可以删除视图。`IF EXISTS` 用于在删除视图不存在时不报错。

5.**视图的优势：**

- **简化复杂查询：** 视图可以用于封装复杂的查询逻辑，简化用户对数据的查询操作。
- **提高可维护性：** 将复杂查询逻辑封装在视图中，提高了代码的可读性和可维护性。
- **控制数据访问：** 视图可以用于限制用户对数据的访问，只暴露部分列或行。

6.**注意事项：**

- 视图中的查询结果是实时计算的，每次查询都会重新执行视图的查询逻辑。
- Hive 视图与传统数据库中的视图概念相似，但由于 Hive 是基于 MapReduce 的，其实现方式略有不同。

## 9.Hive Lateral View

在 Hive 中，LATERAL VIEW 是一种用于处理复杂数据类型的语法结构。它通常与 explode 函数一起使用，以展开嵌套数据结构中的**数组或映射**。

> **LATERAL VIEW 语句用于处理嵌套数据结构，其中 `explode` 函数用于展开数组或映射。**

假设有一个表 `user_ratings` 包含以下数据：

```sql
user_id  |  movie_ratings
-------------------------
1        |  [4, 5, 3]
2        |  [2, 5]
3        |  [4, 2, 1, 5]
```

如果我们想展开 `movie_ratings` 数组，可以使用 LATERAL VIEW 和 explode 函数。查询模拟结果如下：

```sql
SELECT user_id, movie_rating
FROM user_ratings
LATERAL VIEW explode(movie_ratings) AS movie_rating;
```

模拟结果：

```sql
user_id  |  movie_rating
-------------------------
1        |  4
1        |  5
1        |  3
2        |  2
2        |  5
3        |  4
3        |  2
3        |  1
3        |  5
```

上述查询通过 LATERAL VIEW 和 explode 将数组 `movie_ratings` 展开为单独的行，每行包含一个 `user_id` 和一个 `movie_rating`。

## 10.Hive Join

Hive的JOIN语句类似于数据库中的JOIN，用于将两个或多个表中的行组合在一起。支持INNER JOIN、OUTER JOIN（RIGHT  JOIN、LEFT JOIN、FULL OUTER JOIN）和CROSS JOIN。这些JOIN操作发生在WHERE子句之前。

```
Area C = Circle1 JOIN Circle2
Area A = Circle1 LEFT OUTER JOIN Circle2
Area B = Circle1 RIGHT OUTER JOIN Circle2
AUBUC = Circle1 FULL OUTER JOIN Circle2
```

![image-20240125181535761](C:\Users\Q27\AppData\Roaming\Typora\typora-user-images\image-20240125181535761.png)

#### MAPJOIN

在 Hive 中，MAPJOIN 是一种优化技术，用于处理小表与大表的连接操作。通过 MAPJOIN，Hive 将小表加载到内存中，并在 Map 阶段完成连接操作，避免了大量的数据传输和 Reduce 阶段的开销。以下是关于 Hive 中 MAPJOIN 的一些关键知识点：

1. **MAPJOIN 的使用场景：**
   - 当连接的两个表中，有一个表很小，可以完全加载到内存中时，MAPJOIN 是一个有效的优化手段。
   - 适用于小表与大表的连接操作，能够显著提高查询性能。
2. **MAPJOIN 的语法：**

```sql
-- 使用 MAPJOIN
SET hive.auto.convert.join=true;
SET hive.mapjoin.smalltable.filesize=25; -- 阈值，小表的大小
SELECT /*+ MAPJOIN(small_table) */ *
FROM large_table
JOIN small_table ON large_table.id = small_table.id;
```

上述语法中，`hive.auto.convert.join` 用于启用自动连接优化，`hive.mapjoin.smalltable.filesize` 用于设置小表的大小阈值。`MAPJOIN(small_table)` 提示 Hive 使用 MAPJOIN。

1. **MAPJOIN 的工作原理：**(了解)
   - 将小表加载到内存中作为哈希表。
   - 对大表的每个 Map 任务，将小表的哈希表传递给 Map 任务。
   - 在 Map 任务中，使用哈希表进行连接操作，避免了数据传输和 Reduce 阶段的开销。
2. **MAPJOIN 的注意事项：**
   - 小表的大小应该足够小，以便可以加载到内存中。阈值通过 `hive.mapjoin.smalltable.filesize` 设置。
   - 尽量使用合适的索引或分区，以便更好地利用 MAPJOIN 进行连接操作。
   - 使用 MAPJOIN 时，需要适当配置 Hive 运行环境，确保内存和其他资源充足。

MAPJOIN 是一种**用于优化小表与大表连接操作的有效手段**，能够提高查询性能。在使用 MAPJOIN 时，需要注意合理设置阈值和配置环境，以达到最佳的性能优化效果。

## 11.Hive  UNION (纵向合并)

UNION ALL(保留重复数据)
UNION(不保留重复数据

## 12.Hive Data Movement – LOAD (数据迁移)

在Hive中移动数据，使用load关键字。这里的移动意味着原始数据被移动（剪切）到目标表/分区，并且不再存在于原始位置。

```sql
-从本地磁盘加载数据到hive对应的hdfs的目录，源地址的数据还存在。相当于复制
LOAD DATA LOCAL INPATH 'C://home/data/employee_hr.txt' OVERWRITE INTO TABLE
employee_hr;
--从HDFS加载数据，把目的地的数据移动到hive表对应的hdfs的目录，源地址的数据会被移动(mv)过去 ->
推荐
LOAD DATA INPATH '/home/data/employee_hr.txt' OVERWRITE INTO TABLE employee;
-- 加载数据到指定的分区，会自动生成元数据
LOAD DATA LOCAL INPATH '/home/data/employee_hr.txt' OVERWRITE INTO TABLE
employee_partitioned PARTITION (year=2014,month=12);
-- 从HDFS到hive表，加上hfds协议，这个协议可以省却，在生产环境需要加上去
LOAD DATA INPATH 'hdfs://hadoop5:8020/user/employee/employee.txt' OVERWRITE INTO
TABLE employee;
```

## 13. 5个By

在 Hive 中，`ORDER BY`、`SORT BY`、`DISTRIBUTE BY`、`CLUSTER BY` 和 `GROUP BY` 是用于对查询结果进行排序、分发和分组的关键字。

假设有一个表 `example_table` 包含以下数据：

```sql
id  |  name  |  age
--------------------
1   |  Alice |  25
2   |  Bob   |  30
3   |  Carol |  28
4   |  David |  22
```

### 1.**ORDER BY**

- 作用：对整个结果集按指定列进行排序。

```sql
SELECT * FROM example_table
ORDER BY age DESC;
```

结果：

```sql
id  |  name  |  age
--------------------
2   |  Bob   |  30
3   |  Carol |  28
1   |  Alice |  25
4   |  David |  22
```

### 2.**SORT BY**

- 作用：**对每个Reducer的输出进行本地排序，而不是全局排序**。
- 它在每个Reducer节点上对数据进行排序，但不合并各个Reducer的结果。这意味着最终的输出可能是未排序的。

```sql
SELECT * FROM example_table
SORT BY age DESC;
```

```sql
id  |  name  |  age
--------------------
2   |  Bob   |  30
3   |  Carol |  28
1   |  Alice |  25
4   |  David |  22
```

### 3.DISTRIBUTE BY

- 作用： `DISTRIBUTE BY` 用于将数据按照**指定列的哈希值分发到不同的Reducer上，但不会进行排序**。它确保具有相同 `DISTRIBUTE BY` 列值的记录被分发到同一个Reducer，从而避免了在Reduce端进行全局排序的开销。
- 通常和sort by一起用，distribute by必须要写在sort by之前。理解成：按照XX字段分区，再按照XX字段排序

```SQL
SELECT * FROM example_table
DISTRIBUTE BY age;
```

```sql
id  |  name  |  age
--------------------
3   |  Carol |  28
2   |  Bob   |  30
1   |  Alice |  25
4   |  David |  22
```

### 4.**CLUSTER BY** 

- 作用：对数据按指定列进行排序，并将相邻的相同值分发到同一个Reducer上。

```sql
SELECT * FROM example_table
CLUSTER BY age DESC;
```

按照 age 进行排序，并相邻的相同值分发到同一个Reducer

```sql
id  |  name  |  age
--------------------
2   |  Bob   |  30
3   |  Carol |  28
1   |  Alice |  25
4   |  David |  22
```

### 5.**GROUP BY**

- 作用：将数据按指定列进行分组，并对每个组应用聚合函数。

```sql
SELECT age, COUNT(*) as count
FROM example_table
GROUP BY age;
```

模拟结果：

```sql
age | count
-----------
30  | 1
28  | 1
25  | 1
22  | 1
```

## 14.Hive 排序函数

在 Hive 中，可以使用窗口函数（Window Functions）来执行类似于 SQL 标准的 `ROW_NUMBER`、`RANK`、`DENSE_RANK`、`NTILE` 和 `PERCENT_RANK` 等排名和百分比排名的操作。

```sql
id  |  name  |  age
--------------------
1   |  Alice |  22
2   |  Bob   |  30
3   |  Carol |  28
4   |  David |  22
5   |  Perter|  21
```

### 1.**ROW_NUMBER：**

- **语法：** `ROW_NUMBER() OVER (PARTITION BY partition_col1, partition_col2, ... ORDER BY order_col1 [ASC|DESC], order_col2 [ASC|DESC], ...) AS row_num`

- **作用：** 为每一行分配**一个唯一的整数值**，基于 `ORDER BY` 子句指定的排序顺序。经常用来去除重复数据。

  row_number()的值不会存在重复,当排序的值相同时,按照表中记录的顺序进行排列

```sql
SELECT id, name, age, ROW_NUMBER() OVER (ORDER BY age DESC) AS row_num
FROM example_table;
```

```sql
id  |  name  |  age  |  row_num
--------------------------------
2   |  Bob   |  30   |  1
3   |  Carol |  28   |  2
1   |  Alice |  22   |  3
4   |  David |  22   |  4
5   |  Perter|  21   |  5
```

### 2.**RANK：**

- **语法：** `RANK() OVER (PARTITION BY partition_col1, partition_col2, ... ORDER BY order_col1 [ASC|DESC], order_col2 [ASC|DESC], ...) AS rank_num`
- **作用：** 为每一行分配一个排名，**相同值的行将获得相同的排名，下一行的排名将跳过相同的排名**。

```sql
SELECT id, name, age, RANK() OVER (ORDER BY age DESC) AS rank_num
FROM example_table;
```

```sql
id  |  name  |  age  |  rank_num
--------------------------------
2   |  Bob   |  30   |  1
3   |  Carol |  28   |  2
1   |  Alice |  22   |  3
4   |  David |  22   |  3
5   |  Perter|  21   |  5
```

### 3.**DENSE_RANK：**

- **语法：** `DENSE_RANK() OVER (PARTITION BY partition_col1, partition_col2, ... ORDER BY order_col1 [ASC|DESC], order_col2 [ASC|DESC], ...) AS dense_rank_num`
- **作用：** 为每一行分配一个密集排名，相同值的行将获得相同的排名，下一行的排名不会跳过相同的排名。

```sql
SELECT id, name, age, DENSE_RANK() OVER (ORDER BY age DESC) AS dense_rank_num
FROM example_table;
```

```sql
id  |  name  |  age  |  dense_rank_num
--------------------------------------
2   |  Bob   |  30   |  1
3   |  Carol |  28   |  2
1   |  Alice |  22   |  3
4   |  David |  22   |  3
5   |  Perter|  21   |  4
```

### 4.NTILE

- **语法：** `NTILE(n) OVER (PARTITION BY partition_col1, partition_col2, ... ORDER BY order_col1 [ASC|DESC], order_col2 [ASC|DESC], ...) AS ntile_num`
- **作用：** 将有序分区划分为 n 个相等大小的桶，为每一行分配一个桶号。

```sql
SELECT id, name, age, NTILE(3) OVER (ORDER BY age DESC) AS ntile_num
FROM example_table;
```

```sql
id  |  name  |  age  |  ntile_num
----------------------------------
2   |  Bob   |  30   |  1
3   |  Carol |  28   |  1
1   |  Alice |  22   |  2
4   |  David |  22   |  2
5   |  Perter|  21   |  3
```

### 5.**PERCENT_RANK：**

- **语法：** `PERCENT_RANK() OVER (PARTITION BY partition_col1, partition_col2, ... ORDER BY order_col1 [ASC|DESC], order_col2 [ASC|DESC], ...) AS percent_rank_num`
- **作用：** 为每一行分配一个相对排名的百分比值，范围在 0 到 1 之间。

```sql
SELECT id, name, age, PERCENT_RANK() OVER (ORDER BY age DESC) AS percent_rank_num
FROM example_table;
```

```sql
id  |  name   |  age  |  percent_rank_num
-----------------------------------------
2   |  Bob    |  30   |  0.0
3   |  Carol  |  28   |  0.25
1   |  Alice  |  22   |  0.75
4   |  David  |  22   |  0.75
5   |  Peter  |  21   |  1.0
```

## 15.Hive 事务

事务是一系列操作一起执行或一起不执行的过程。

**原子性（Atomicity）：** 事务是不可分割的工作单元，事务中的所有操作要么全部发生，要么全部不发生。

**一致性（Consistency）：** 事务开始之前和事务结束后，数据库的完整性约束没有被破坏。这意味着数据库事务不能破坏关系数据的完整性以及业务逻辑上的一致性。

**隔离性（Isolation）：** 多个事务并发访问，事务之间是隔离的。每个事务在执行时应该与其他事务相互隔离，互不干扰。

**持久性（Durability）：** 在事务完成后，该事务对数据库所做的更改应该持久保存在数据库中，并且不会被回滚。

### 特性：

1. **支持行级别事务：** 从版本 0.14 开始，Hive 支持行级别事务，允许执行 INSERT、DELETE 和 UPDATE 操作。
2. **支持 INSERT、DELETE、UPDATE：** Hive 表在启用 ACID 特性后支持 INSERT、DELETE 和 UPDATE 操作。
3. **事务表支持 DELETE 和 UPDATE：** 只有**启用 ACID 特性的事务表**才支持 DELETE 和 UPDATE 操作。
4. **数据存储格式限制：** Hive 表的事务操作仅支持 ORC（Optimized Row Columnar）格式的数据存储。
5. **必须是桶表：** 事务表必须是桶表（bucketed clustered table），这是 ACID 特性的一个要求。
6. **需要压缩工作：** 使用事务表进行事务操作后，可能需要进行压缩工作，这会消耗时间、资源和空间。

### 限制：

1. **不支持 BEGIN、COMMIT 和 ROLLBACK：** Hive 不支持标准的 BEGIN、COMMIT 和 ROLLBACK 语句来显式控制事务。
2. **不支持对桶列或分区列的更新：** 事务操作不支持对桶列（bucket column）或分区列的更新。
3. **数据存储格式只支持 ORC：** 事务操作只支持 ORC 格式的数据存储，其他格式不受支持。
4. **不是完全生产就绪：** 尽管 Hive 的事务性操作得到支持，但在某些方面仍然被认为不是完全生产就绪，可能存在一些稳定性和性能方面的问题。

```sql
set hive.support.concurrency=true;
--启用Hive并发支持，允许多个用户同时访问Hive。
set hive.enforce.bucketing=true;
--启用Hive桶排序特性，确保表按照指定的列（这里是emp_id）进行分桶存储。
set hive.exec.dynamic.partition.mode=nonstrict;
--设置Hive动态分区模式为非严格模式，允许在分区字段中插入未知分区值。
set hive.txn.manager=org.apache.hadoop.hive.ql.lockmgr.DbTxnManager;
--设置Hive事务管理器为DbTxnManager。
set hive.compactor.initiator.on=true;
--启用Hive压缩器的初始化器，用于在表上启动压缩操作。
set hive.compactor.worker.threads=1;
--设置Hive压缩器的工作线程数为1

--创建一张带事务的表
CREATE TABLE IF NOT EXISTS employee_transactional (
emp_id int,
emp_name string,
dept_name string,
work_loc string
)
PARTITIONED BY (start_date string)
CLUSTERED BY (emp_id) INTO 5 BUCKETS -- 桶表
STORED AS ORC -- ORC存储
TBLPROPERTIES('transactional'='true') -- 在表属性中设置事务支持为true，表示这个表是一个事务表。

-- 该表可以进行update/delete/insert 操作 （一般hive表不支持update和delete）
```

## 16. Hive 数据倾斜

### 1.什么是数据倾斜？

 由于数据分布不均匀，造成数据大量的集中到一点，造成数据热点。

### 2.数据倾斜的现象

在执行任务时，任务进度长时间维持在99%左右，查看任务监控页面发现只有极少数（1个或几个）reduce子任务未完成。这是因为这些reduce子任务处理的数据量与其他reduce子任务存在显著差异，单一reduce的记录数与平均记录数相差很大，可能达到3倍甚至更多。因此，这些未完成的reduce子任务的最长执行时长远大于平均时长。

### 3.导致数据倾斜的操作

**分桶不均匀：**

- 如果使用分桶来存储数据，并且选择的分桶列不具有均匀分布的特性，可能导致某些桶中的数据量远远超过其他桶。

**静态分区：**

- 使用静态分区时，如果数据不均匀分布在分区键上，某些分区可能包含大量数据，导致倾斜。

**连接键选择不当：**

- 在进行连接操作时，选择的连接键如果不是均匀分布的，可能导致连接结果集中某些键对应的数据量过大。

**Group By 和 Order By 操作：**

- 在执行Group By或Order By操作时，如果选择的列中包含某些特定值的频率非常高，可能导致数据在执行这些操作时倾斜。

**聚合操作：**

- 某些聚合操作，如COUNT(DISTINCT column)可能导致数据倾斜，因为计算唯一值的操作可能会使某些计算任务更为繁重。

**数据导入策略：**

- 数据导入时，如果数据源本身就存在倾斜，或者导入数据的策略导致了数据分布不均匀，可能会影响后续的查询性能。

**MapReduce的默认分区器：**

- 在使用MapReduce引擎时，默认的分区器可能导致某些分区的负载过重，需要适当选择分区器。

**数据分布不均匀的表连接：**

- 在连接操作中，如果连接的两个表的连接键的数据分布不均匀，可能导致倾斜。

### 4.解决方案

#### 聚合

```sql
-- Map 端部分聚合，相当于Combiner
SET hive.map.aggr=true;

-- 有数据倾斜的时候进行负载均衡
SET hive.groupby.skewindata=true;
-- 当选项设定为 true，生成的查询计划会有两个 MR Job。
-- 第一个 MR Job 中，Map 的输出结果集合会随机分布到 Reduce 中，每个 Reduce 做部分聚合操作，并输出结果，
-- 这样处理的结果是相同的 Group By Key 有可能被分发到不同的 Reduce 中，从而达到负载均衡的目的；
-- 第二个 MR Job 再根据预处理的数据结果按照 Group By Key 分布到 Reduce 中（这个过程可以保证相同的 Group By Key 被分布到同一个 Reduce 中），
-- 最后完成最终的聚合操作。
```

#### SQL优化

**Join时选择均匀分布的表作为驱动表：**

- 在Join操作中，选择具有最均匀数据分布的表作为驱动表，进行列裁剪和filter操作，以减小Join的数据量。

**小表Join使用Map Join：**

- 对于小的维度表（记录条数在1000条以下），使用Map Join，将小表先加载到内存中，在Map端完成Join操作，提高效率。

**大表Join大表处理数据倾斜：**

- 在两个大表进行Join时，将空值的key变成带有随机数的字符串，以将倾斜的数据分到不同的Reduce上，避免数据倾斜影响最终结果。

**处理count distinct大量相同特殊值：**

- 对于计算count distinct时，将值为空的情况单独处理，可以不用参与计算，最后在结果中加1。如果还有其他计算，可以先将值为空的记录单独处理，再和其他计算结果进行union。

**特殊情况特殊处理：**

- 在业务逻辑优化效果不大的情况下，对于一些特殊情况，可以将倾斜的数据单独拿出来处理，最后再进行union操作。

```sql
--空值产生的数据倾斜
--场景：如日志中，常会有信息丢失的问题，比如日志中的 user_id，如果取其中的 user_id 和 用户表中的
--user_id 关联，会碰到数据倾斜的问题。
--解决办法1：user_id为空的不参与关联
select * from log a join users b on a.user_id is not null and a.user_id =
b.user_id union all select * from log a where a.user_id is null;
--解决办法2：赋与空值分新的key值
select * from log a left outer join users b on case when a.user_id is null then
concat(‘hive’,rand()) else a.user_id end = b.user_id;
--不同数据类型关联产生数据倾斜
--场景：用户表中user_id字段为int，log表中user_id字段既有string类型也有int类型。当按照user_id
--进行两个表的Join操作时，默认的Hash操作会按int型的id来进行分配，这样会导致所有string类型id的记
--录都分配到一个Reducer中
--解决办法
--把数字类型转换成字符串类型
select * from users a left outer join logs b on a.usr_id = cast(b.user_id as
string);
```

