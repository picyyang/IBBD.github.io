# Hive基础入门

```
hive> show databases;
OK
default
Time taken: 2.023 seconds, Fetched: 1 row(s)
hive> use default;
OK
Time taken: 0.038 seconds
hive> show tables;
OK
t_hive
Time taken: 0.043 seconds, Fetched: 1 row(s)
```

## Hive数据管理
Hive是建立在Hadoop上的数据仓库基础架构。它提供了一系列的工具，用来进行数据提取、转换、加载，这是一种可以存储、查询和分析存储在Hadoop中的大规模数据机制。可以把Hadoop下结构化数据文件映射为一张成Hive中的表，并提供类sql查询功能，除了不支持更新、索引和事务，sql其它功能都支持。可以将sql语句转换为MapReduce任务进行运行，作为sql到MapReduce的映射器。

### （1）元数据存储

Hive将元数据存储在RDBMS中，有三种方式可以连接到数据库：

- 内嵌模式：元数据保持在内嵌数据库的Derby，一般用于单元测试，只允许一个会话连接
- 多用户模式：在本地安装Mysql，把元数据放到Mysql内
- 远程模式：元数据放置在远程的Mysql数据库

### （2）数据存储

首先，Hive没有专门的数据存储格式，也没有为数据建立索引，用于可以非常自由的组织Hive中的表，只需要在创建表的时候告诉Hive数据中的列分隔符和行分隔符，这就可以解析数据了。

其次，Hive中所有的数据都存储在HDFS中，Hive中包含4中数据模型：Tabel、ExternalTable、Partition、Bucket。

- Table：类似与传统数据库中的Table，每一个Table在Hive中都有一个相应的目录来存储数据。例如：一个表zz，它在HDFS中的路径为：/wh/zz，其中wh是在hive-site.xml中由${hive.metastore.warehouse.dir}指定的数据仓库的目录，所有的Table数据（不含External Table）都保存在这个目录中。
- Partition：类似于传统数据库中划分列的索引。在Hive中，表中的一个Partition对应于表下的一个目录，所有的Partition数据都存储在对应的目录中。例如：zz表中包含ds和city两个Partition，则对应于ds=20140214，city=beijing的HDFS子目录为：/wh/zz/ds=20140214/city=Beijing;
- Buckets：对指定列计算的hash，根据hash值切分数据，目的是为了便于并行，每一个Buckets对应一个文件。将user列分数至32个Bucket上，首先对user列的值计算hash，比如，对应hash=0的HDFS目录为：/wh/zz/ds=20140214/city=Beijing/part-00000;对应hash=20的，目录为：/wh/zz/ds=20140214/city=Beijing/part-00020。
- ExternalTable指向已存在HDFS中的数据，可创建Partition。和Table在元数据组织结构相同，在实际存储上有较大差异。Table创建和数据加载过程，可以用统一语句实现，实际数据被转移到数据仓库目录中，之后对数据的访问将会直接在数据仓库的目录中完成。删除表时，表中的数据和元数据都会删除。ExternalTable只有一个过程，因为加载数据和创建表是同时完成。世界数据是存储在Location后面指定的HDFS路径中的，并不会移动到数据仓库中。

### （3）数据交换

- 用户接口：包括客户端、Web界面和数据库接口
- 元数据存储：通常是存储在关系数据库中的，如Mysql，Derby等
- Hadoop：用HDFS进行存储，利用MapReduce进行计算。
- 关键点：Hive将元数据存储在数据库中，如Mysql、Derby中。Hive中的元数据包括表的名字、表的列和分区及其属性、表的属性（是否为外部表）、表数据所在的目录等。

Hive的数据存储在HDFS中，大部分的查询由MapReduce完成。

## 第一个demo

准备文件：

```
hive> dfs -cat file:/root/hive-test.log;
1,hello
2,world
```

创建表，导入数据，删表等：

```
# 创建测试数据表
hive> CREATE TABLE t_hive (a int, b string) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';
OK
Time taken: 2.366 seconds

# 导入数据
hive> LOAD DATA LOCAL INPATH '/root/hive-test.log' OVERWRITE INTO TABLE t_hive ;
Loading data to table default.t_hive
OK
Time taken: 0.97 seconds

# 查询数据
hive> select * from t_hive;
OK
1	hello
2	world
Time taken: 0.118 seconds, Fetched: 2 row(s)

# 查看表结构
hive> desc t_hive;
OK
a                   	int                 	                    
b                   	string              	                    
Time taken: 2.091 seconds, Fetched: 2 row(s)

# 删除数据表
drop table t_hive;
```

## 数据结构

```sql
-- 扩展数据类型
data_type
  : primitive_type
  | array_type
  | map_type
  | struct_type
  | union_type  -- (Note: Available in Hive 0.7.0 and later)
array_type : ARRAY < data_type >
map_type : MAP < primitive_type, data_type >
struct_type : STRUCT < col_name : data_type [COMMENT col_comment], ...>
union_type : UNIONTYPE < data_type, data_type, ... >  -- (Note: Available in Hive 0.7.0 and later)

-- 基本数据类型 
primitive_type
  : TINYINT
  | SMALLINT
  | INT
  | BIGINT
  | BOOLEAN
  | FLOAT
  | DOUBLE
  | STRING
  | BINARY      -- (Note: Available in Hive 0.8.0 and later)
  | TIMESTAMP   -- (Note: Available in Hive 0.8.0 and later)
  | DECIMAL     -- (Note: Available in Hive 0.11.0 and later)
  | DECIMAL(precision, scale)  -- (Note: Available in Hive 0.13.0 and later)
  | DATE        -- (Note: Available in Hive 0.12.0 and later)
  | VARCHAR     -- (Note: Available in Hive 0.12.0 and later)
  | CHAR        -- (Note: Available in Hive 0.13.0 and later)
```

## Hive  DDL
Hive DDL的语方法为类SQL语法，所以标准的SQL语法大多数在Hive中都可用。

```sql
-- Hive建表 语法
CREATE [EXTERNAL] TABLE [IF NOT EXISTS] table_name   
  [(col_name data_type [COMMENT col_comment], ...)]   
  [COMMENT table_comment]   
  [PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)]   
  [CLUSTERED BY (col_name, col_name, ...)   
  [SORTED BY (col_name [ASC|DESC], ...)] INTO num_buckets BUCKETS]   
  [ROW FORMAT row_format]   
  [STORED AS file_format]   
  [LOCATION hdfs_path]  
-- CREATE TABLE 创建一个指定名字的表。如果相同名字的表已经存在，则抛出异常；用户可以用 IF NOT EXIST 选项来忽略这个异常  
-- EXTERNAL 关键字可以让用户创建一个外部表，在建表的同时指定一个指向实际数据的路径（LOCATION）  
-- LIKE 允许用户复制现有的表结构，但是不复制数据  
-- COMMENT可以为表与字段增加描述  

-- ROW FORMAT  
    DELIMITED [FIELDS TERMINATED BY char] [COLLECTION ITEMS TERMINATED BY char]  
        [MAP KEYS TERMINATED BY char] [LINES TERMINATED BY char]  
        | SERDE serde_name [WITH SERDEPROPERTIES (property_name=property_value, property_name=property_value, ...)]  
-- 用户在建表的时候可以自定义 SerDe 或者使用自带的 SerDe。如果没有指定 ROW FORMAT 或者 ROW FORMAT DELIMITED，将会使用自带的 SerDe。在建表的时候，用户还需要为表指定列，用户在指定表的列的同时也会指定自定义的 SerDe，Hive 通过 SerDe 确定表的具体的列的数据。  


create table person( 
id int, 
name string, 
age int, 
likes array<string>, 
address map<string,string> 
) 
row format delimited  
-- 指定导入数据的列与列之间的分隔符
fields terminated by ','  
-- 指定Array类型的分隔符
collection ITEMS TERMINATED BY  '-' 
-- 指定map类型的分隔符
map keys terminated by ':'  
-- 指定行与行之间的分隔符
lines terminated by '\n'; 
```

数据格式例如：

```
# 三条数据，列与列之间用,号隔开；array之间用-号隔开；map之间用：号隔开；行与行用换行符隔开
1,tom,28,game-music-book,stu:henan-home:henan-work:beijing
2,jack,21,money-meinv,stu:wuhan-home:wuhan
3,lusi,18,shopping-music,stu:shanghai-home:beijing
```

## 数据查询

```sql
-- 查询所有
select * from person;

-- 还可以这样查
 select * from person where name='tom';

-- 或者这样
select * from person where likes[1]='music';

-- 还有这样
select * from person where address['stu']='shanghai'; 

-- 还有这样
select avg(age) from person;

-- ... 等标准的SQL语法大多都可以在Hive中使用包括一些函数，因为Hive是类SQL的；
```

但在Hive中`不推荐进行这些操作：Insert、Update、Delete等操作`，因为Hive的特性是对数据仓库的数据进行提取，针对的数据是批量的，`不适合行级的运算`。

## 清空表

```sql
-- 使truncate清空表
TRUNCATE TABLE person;
-- 通过覆盖的方式清空表
insert overwrite table person select * from person where 1=2;
```

## 加载数据

```
语法：LOAD DATA [LOCAL] INPATH 'filepath' [OVERWRITE] INTO TABLE tablename [PARTITION (partcol1=val1, partcol2=val2 ...)]
```

## Hive的内表
Hive 的内表，就是正常创建的表。在删除时，既删除内表的元数据，也删除内表的数据。

## Hive的外表
删除时，仅仅删除外表的元数据。创建Hive 的外表，需要使用关键字 External：

## 分区表
分区表是通过关键字PARTITIONED BY来实现分区，一个表有一个或多个分区，分区是以字段的形式在表结构中存在，通过describe table命令可以查看到字段存在，但是该字段不存放实际的数据内容，仅仅是分区的表示（伪列）。

在HDFS中，分区每个分区的值都会产生相应的文件夹，然后在对应的文件夹下存放相应的表数据。

分区表的应用场景分析：假如针对全球的一家电子商务公司，现在有这样一个业务需求就是需要了解一下海外市场的情况，目的是进一步想拓展海外市场，但是公司对海外市场的拓展现在还没有任何的了解，那么怎么帮助公司管理层提供数据上的支撑呢？这个时候可以对之前公司针对全球销售的商品通过按“国家”来分区，在查询的时候，以国家为为纬度来进行分析海外的市场情况。

```
partitioned by (date string)

-- 修改分区的存储路径
-- 注意格式：dt='2008-08-08' ：dt表示分区名，'2008-08-08'表示对应分区值。注意格式。若是字符串的话就是“string”
-- "new location"就是对应存储路径，应该是绝对路径
ALTER TABLE table_name PARTITION (dt='2008-08-08') SET LOCATION "new location";

-- 修改分区名称
ALTER TABLE table_name PARTITION (dt='2008-08-08') RENAME TO PARTITION (dt='20080808');

-- 删除分区
ALTER TABLE login DROP IF EXISTS PARTITION (dt='2008-08-08'); 
ALTER TABLE page_view DROP IF EXISTS PARTITION (dt='2008-08-08', country='us');

-- 添加分区
ALTER TABLE table_name ADD PARTITION (partCol = 'value1') location 'loc1';
```


