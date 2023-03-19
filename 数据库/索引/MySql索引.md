# 索引

![MySQL索引](D:\study\person_blog\数据库\索引\Mysql索引.png)

MySQL 索引是一种数据结构，用于加快查询速度。它类似于书籍的目录，可以快速定位到某个关键字对应的数据行。MySQL 索引可以在查询时提供快速的访问路径，从而减少全表扫描的开销，提高查询效率。

## 原理

MySQL 索引的原理是通过创建一个索引数据结构，将索引列的值映射到对应的数据行位置，然后通过索引数据结构来定位符合查询条件的数据行，从而避免了全表扫描的过程。MySQL 支持多种索引类型，包括：

## 分类

建立方式上分为：

1. 单列索引：只针对一列数据创建的索引
2. 多列索引：由多个列组成的索引
3. 唯一索引：可以单列，也可以多列。索引值必须唯一，允许有空值（Null），但只允许有一个空值（Null）。
4. 主键索引：属于单列索引的一种。定为主键后，数据库自动建立索引，InnoDB为聚簇索引，主键索引列值不能为空（Null）。
5. 全文索引：Full Text（MySQL5.7之前，只有MYISAM存储引擎引擎支持全文索引）。全文索引类型为FULLTEXT，在定义索引的列上支持值的全文查找允许在这些索引列中插入重复值和空值。全文索引可以在Char、VarChar
   上创建。
6. 前缀索引：在文本类型为char、varchar、text类列上创建索引时，可以指定索引列的长度，但是数值类型不能指定。

实现方式上分为：

1. B-Tree 索引：是最常用的索引类型，可以用于普通列、唯一列和主键列。它使用平衡树结构来存储索引数据，可以快速定位到符合条件的数据行。
2. Hash 索引：适用于等值比较查询，使用哈希表结构来存储索引数据，可以快速定位到符合条件的数据行。但是，Hash 索引不支持范围查询和排序操作。
3. Full-Text 索引：适用于全文搜索，可以快速查找包含特定单词或短语的文本数据。
4. R-Tree 索引：适用于空间数据查询，例如地理位置数据、图片数据等。

## 索引操作

```sql
-- 查看索引
show
    index from 表名;

-- 删除索引
drop index 索引名 on 表名;

-- 创建单列索引，只能包含一个字段
create index name_index on table_name (target_field);

alter table table_name
    add index name_index (target_field);

-- 创建唯一索引，只能有一个列
create unique index index_name on table_name (target_field);

alter table table_name
    add unique index_name (target_field);

-- 复合索引
create index index_name on table_name (target_field1, target_field2...);

alter table table_name
    add index index_name (target_field1, target_field2...);

```

## 索引的数据结构

MySQL索引使用的数据结构主要有BTree索引和hash索引。 对于hash索引来说，底层的数据结构就是哈希表，因此在绝大多数需求为单条记录查询的时候，可以选择哈希索引，查询性能最快；其余大部分场景建议选择BTree索引。
根据存储引擎的不同，实现方式也不同

### B+Tree索引

MySQL的索引数据结构最常使用的是B树中的B+Tree，但对于主要的两种存储引擎的实现方式是不同的。InnoDB中data阈存储的是行数据，而MyISAM中存储的是磁盘地址

- MyISAM:
  B+Tree叶节点的data域存放的是数据记录的地址。在索引检索的时候，首先按照B+Tree搜索算法搜索索引，如果指定的Key存在，则根据data域中磁盘地址到磁盘中寻址定位到对应的磁盘块，然后读取相应的数据记录，这被称为“非聚簇索引”。B+Tree叶节点的data域存放的是数据记录的地址。在索引检索的时候，首先按照B+Tree搜索算法搜索索引，如果指定的Key存在，则根据data域中磁盘地址到磁盘中寻址定位到对应的磁盘块，然后读取相应的数据记录，这被称为“非聚簇索引”。


- InnoDB：
  其数据文件本身就是索引文件。相比MyISAM，索引文件和数据文件是分离的，其表数据文件本身就是按照B+Tree组织的一个索引结构，树的叶节点data域保存了完整的数据记录。这个索引的Key是数据表的主键，因此InnoDB表数据文件本身就是主索引。这被称为“聚簇索引（聚集索引）”。而其余的索引都作为辅助索引，辅助索引的data域存储相应记录主键的值而不是地址，这也是和MyISAM不同的地方

    - 在根据主索引搜索时，直接找到Key所在的节点即可取出数据；
    - 在根据辅助索引查找时，则需要先取出主键的值，再走一遍主索引。
    - 因此在设计表的时候，不建议使用过长的字段作为主键，也不建议使用非单调的字段作为主键，这样会造成主索引频繁分裂。

### Hash索引

Hash表，在Java中的HashMap，TreeMap就是Hash表结构，以键值对的形式存储数据。我们使用hash表存储表数据结构，Key可以存储索引列，Value可以存储行记录或者行磁盘地址。Hash表在等值查询时效率很高，时间复杂度为O(
1)；但是不支持范围快速查找，范围查找时只能通过扫描全表的方式，筛选出符合条件的数据。

### mysql B+Tree索引的进化过程

[https://developer.aliyun.com/article/831250](https://developer.aliyun.com/article/831250)

## 注意事项

1. 索引并不是越多越好，过多的索引可能会影响数据的插入、更新和删除操作的性能，甚至可能导致索引失效。
2. 对于较小的表或经常进行全表扫描的查询，不应该创建索引，因为这会增加额外的开销，反而会降低查询效率。
3. 对于经常用于查询的列，应该创建索引，以提高查询效率。
4. 在设计索引时应该考虑查询的频率、查询条件的复杂度、表的大小等因素，以达到最优的查询性能。
5. 索引本身很大
6. 数据量比较少的表，不建议创建索引

# 避免索引失效

- 复合索引，遵循最佳做前缀原则，不要跨列或乱序使用；反之索引失效
- 不要再索引上进行函数调用、逻辑运算、类型转换等操作，否则索引失效
- 复合索引，不要使用 `!= 、 <> 、is null、is not null` 运算符，否则自身及右侧索引失效
- 尽量使用覆盖索引
- 范围查询 in、between、>、<、>=、<=、可能会导致索引失效
- like查询，如果以%开头，索引失效
- 不要使用类型转换，包括隐式转换和显式转换，否则索引失效
- 不要使用or，否则索引失效
- exist和in的使用：如果主查询数据集大，则用in。如果子查询结果集大就用exist

# order by优化

- using filesort：表示查询结果需要一次额外的排序；排序有2种算法。
    - 双路排序：扫描2次磁盘，一次单独扫描排序字段，一次扫描所需查询的其他字段
    - 单路排序：将排序字段和其他字段一起扫描，然后排序。排序会在buffer中进行，如果buffer不够大，会导致排序失败，这时会使用双路排序。使用 max_length_for_sort_data
      参数来设置buffer的大小，。如果排序字段的长度超过了这个值，就会使用双路排序。
- 避免使用select *。使用select *会有一次额外的计算。并且会导致查询结果集变大（如果有些字段你不需要的话）
- 保证排序字段的一致性，要么都升序，要么都降序

# 慢sql排查

- 可以使用 SHOW VARIABLES LIKE 'slow_query_log' 命令来查看。如果未启用，可以使用 SET GLOBAL slow_query_log = ON 命令来启用它。

- 查看慢查询阈值。可以使用 SHOW VARIABLES LIKE 'long_query_time' 命令来查看。这个值表示超过多少秒的查询将被记录到慢查询日志中。

- 查看慢查询日志文件的位置。可以使用 SHOW VARIABLES LIKE 'slow_query_log_file' 命令来查看。

- 使用mysqldumpslow工具筛选具体的慢查询。例如：可以使用 mysqldumpslow -s t -t 10 /var/log/mysql/slow.log 命令来查看最耗时的10条查询。

# 海量数据分析

- profiles:
- profile:
- 全局日志：

profiles是一种性能分析工具，用于分析MySQL服务器的运行情况，包括查询执行时间、锁定等待时间、访问表的次数等，从而帮助用户定位性能瓶颈，优化数据库的性能。

```sql
-- 查看profiles是否开启
show variables like '%profiling%';
-- 开启profiles
SET profiling = 1;
-- 查看sql执行情况
SHOW PROFILES;
-- 针对某条sql查看执行情况
SHOW PROFILE ALL FOR QUERY <Query_ID>;

```