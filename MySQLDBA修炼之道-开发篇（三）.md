# 五、开发技巧



## 1. 存储树形数据

树形数据的存储方案：

* 路径枚举

  可以增加一个字段path，用于记录节点的所有祖先信息。记录的方式是把所有的祖先信息组织成一个字符串。因为路径（path）字段包含了该节点的所有祖先信息，所以可以轻易地获取某个节点的所有祖先节点，可以用程序先获取path字符串，然后再使用切割字符串的函数处理得到所有的祖先节点。

  ```sql
  # 查找comment_id等于3的所有后代
  SELECT * FROM comments WHERE path LIKE ‘1/3/_%’;
  # 查找下一层子节点
  SELECT * FROM comments WHERE path REGEXP “^1/3/[0-9]+/$”;
  ```

  插入操作也比较简单，只需要复制一份父节点的路径，并将新节点的ID值（comment_id）添加到路径末尾就可以了。



* 闭包表

  闭包表需要额外增加一张表，用于记录节点之间的关系。它不仅记录了节点之间的父子关系，也记录了树中所有节点之间的关系。
  使用如下命令语句新建表path

  ```sql
  CREATE TABLE path (
    ancestor int(11) NOT NULL,
    descendant int(11) NOT NULL,
    PRIMARY KEY (ancestor,descendant)
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8
  ```

  ancestor表示祖先，descendant表示后代，存储的是comment_id值。



## 2. 转换字符集

如果我们需要修改某个表的字符集，比如A表的字符集原来是gbk，现在要将其修改为utf8，一般有以下3种方法:

* 直接在mysql命令行下完成
  * 建立一个临时表B，字段类型和A一致，但字符集是utf8，即表定义中DEFAULTCHARSET=utf8。
  * INSERT INTO BSELECT*FROMA;
  * DROP TABLEA;
  * RENAMEBTO A;

* 使用mysqldump工具完成

  首先导出数据，默认mysqldump导出的dump转储文件为utf8编码的文件，有删除表、创建表的语句。然后修改dump转储文件，将创建表语句里的表或列的字符集定义修改为utf8。最后重新导入此文件即可。

* 使用ICONV命令转换文件编码

  * 以gbk字符集导出数据，不导出表定义。

    ```
    mysqldump -t -uroot -p database_name table_name1 table_name2 --default-character-set=gbk > a_gbk.sql
    ```

  * 使用iconv命令转换文件编码，将其转换为utf8编码。

    ```
    iconv -fgbk -tutf-8 a_gbk.sql > a_utf8.sql
    ```

  * 修改文件中的相关字符集设置。

    ```
    sed -i ‘s/SET NAMES gbk/SET NAMES utf8/’ a_utf8.sql
    ```

  * 删除旧表（table_name1，table_name2），新建表（table_name1，table_name2），注意新建的表应该是utf8字符集。

  * 使用修改过的文件导入数据。

    ```
    mysql -uroot -p database_name --default-character-set=utf8 < a_utf8.sql
    ```

    

## 3. 处理重复值

* 防止表中出现重复的记录：可以使用主键或唯一索引来防止出现重复的记录。也可以设置唯一索引，来强制记录是唯一的。对于可能出现重复的记录，可以使用INSERT IGNORE语句。如果插入的记录并没有和现存的记录发生冲突，则正常插入之；如果有重复冲突，那么INSERT IGNORE将会告诉MySQL丢弃这条记录，且不报错。还可以采用REPLACE语句，如果记录是新的，那么它等同于INSERT。如果插入的是一个重复的记录，那么新记录将会替换旧的记录。

* 统计和识别重复值

  如下语句将查询和计算表person_tbl中（last_name，first_name）组合有重复的记录的数量。

  ```sql
  SELECT COUNT (*) AS repetitions, last_name, first_name
  FROM person_tbl
  GROUP BY last_name, first_name
  HAVING repetitions > 1;
  ```

* 从结果集中消除重复记录

  使用DISTINCT关键字即可从结果集中消除重复记录。

  ```sql
  SELECT DISTINCT last_name, first_name
  FROM person_tbl
  ORDER BY last_name;
  ```

  也可以使用GROUP BY子句

  ```sql
  SELECT last_name, first_name
  FROM person_tbl
  GROUP BY (last_name, first_name);
  ```

* 删除表中的重复记录

  ```sql
  CREATE TABLE tmp SELECT last_name, first_name, sex
  FROM person_tbl;
  GROUP BY (last_name, first_name);
  
  DROP TABLE person_tbl;
  
  ALTER TABLE tmp RENAME TO person_tbl;
  ```

  

## 4. 分页算法

```
mysql> SELECT col_1,col_2 FROM profiles WHERE sex='M' ORDER BY rating limit 100000, 10;
```

这种查询，无论如何索引，效率都会奇差，因为大偏距（high offset）值的查询，会花费大部分时间来扫描大量数据，而这些数据最终都会被丢弃；可以使用覆盖索引（covering index）解决。以下示例中的表已经在（sex，rating）上创建了索引，id是主键。

```
mysql> SELECT col_1,col_2 FROM profiles INNER JOIN (SELECT id FROM profiles WHERE x.sex='M' ORDER BY rating) AS x USING id;
```

以上语句中的SELECT子查询（SELECT id……）可以利用到覆盖索引，由于覆盖索引一般已被加载到内存，因此这种方式的排序效率会高许多。在一定的数据量下，性能尚可。



## 5. 处理NULL值

在SQL中，NULL值与任何其他值的比较（即使是NULL）永远都不会为“真”。

使用LOADDATAINFILE读取数据时，对于空的或丢失的列，将用空字符串“''”来更新它们。如果希望在列中具有NULL值，应在数据文件中使用\N。

使用DISTINCT、GROUP BY或ORDERBY时，所有NULL值将被视为是等同的。

使用ORDERBY时，首先将显示NULL值，如果指定了DESC按降序排列，那么NULL值将在最后面显示。

对于聚合（累计）函数，如COUNT（）、MIN（）和SUM（），将忽略NULL值。对此的例外是COUNT（*），它将计数行而不是单独的列值。



友好地显示NULL值:

* 使用CASE语句

  ```sql
  SELECT
      CASE
      WHEN SUM(size) IS NULL THEN 0
      ELSE SUM(size)
      END
  INTO @l_sum_vol FROM table_a ;
  ```

* 使用COALESCE函数

  ```sql
  SELECT COALESCE( sum(size) , 0 ) FROM table_a
  ```

  COALESCE(value,...)函数：返回值为列表当中的第一个非NULL值，在没有非NULL值的情况下返回值将为NULL。

* 使用IFNULL函数

  ```sql
  SELECT SUM (ifnull(size,0)) FROM table_a;
  ```

  IFNULL(expr1，expr2)函数：假如expr1不为NULL，则IFNULL（）的返回值为expr1；否则其的返回值为expr2。

* 使用IF函数

  ```sql
  SELECT SUM (IF (size is null, 0, size)) AS totalsize FROM table_a;
  ```

  IF(expr1，expr2，expr3)：如果expr1是TRUE，则IF()的返回值为expr2；否则返回值为expr3。IF()的返回值是数字还是字符串视其所在的语境而定。



## 6. 存储URL地址

* 对于存储域名，可按照字符颠倒的方式进行存储，这样做可方便索引。

  ```
  com.fabulab.marcomacaco
  com.fabulapps.kiko
  com.fandora9.angryvirus
  ```

* 存储URL值，一般推荐的做法是对URL值做一个散列，散列值最好是整型，然后存储这个散列值，并在其上创建索引。

  ```sql
  SELECT CONV (RIGHT(MD5(‘http://www.mysql.com/’), 16), 16, 10) AS HASH64;
  ```

  可以这样查询

  ```sql
  SELECT id FROM url WHERE url_hash=CONV(RIGHT(MD5('http://www.mysql.com/'), 16), 16, 10) AND url=http://www.mysql.com;
  ```

  

## 7. 归档历史数据

将历史数据从日常使用的数据中删除，或将其移动到归档历史表中是比较常见的需求。对过期数据的查询很少时，这样做可以提高性能，而且也不用对程序做大的变动。还可以把过期的历史数据放到其他性能较差的实例、机器上，以便更好地利用资源。

另一种比较常见的方式，是按照时间分表，比如按月份、按日来分表存储数据。这种方式比较容易区分数据，方便维护。但要留意如果有跨越多个表的查询，效率可能会比较差，需要综合考虑平衡分表的粒度。



## 8. 使用数据库存储图片

MySQLBLOB类型（MEDIUMBLOB，最大支持16MB的数据）对于绝大部分图片来说都足够了，我们可以使用LOAD_FILE()方法读取一个文件，然后将内容保存到BLOB列中。

由于数据库毕竟不适合于存储大量的图片，如果存储大量图片的话，仍然建议使用文件系统或分布式文件系统。分布式文件系统配合缓存、CDN技术，往往是海量图片存储系统的优选方案。



## 9. 多表UPDATE

对于多表的UPDATE操作需要慎重，建议在更新之前，先使用SELECT语句查询验证下要更新的数据与自己期望的是否一致。

多表UPDATE的方法：

* “UPDATE table1 t1,table2,...,table n”的方式

  ```sql
  UPDATE product p, product_price pp SET pp.price = p.price * 0.8 WHERE p.productId = pp.productId
  ```

* 使用INNERJOIN然后更新

  ```sql
  UPDATE product p INNER JOIN product_price pp ON p.productId = pp.productId SET pp.price = p.price * 0.8
  ```

* 使用LEFT JOIN来做多表UPDATE

  ```sql
  UPDATE product p LEFT JOIN product_price pp ON p.productId = pp.productId SET p.deleted = 1 WHERE pp.productId IS NULL
  ```

  同时更新两张表

  ```sql
  UPDATE product p INNER JOIN product_price pp ON p.productId = pp.productId SET pp.price = p.price * 0.8,p.dateUpdate = CURDATE ()
  ```

  



## 10. 生成全局唯一ID

由于分布式数据库的部署，多个节点之间为了避免数据冲突，需要有一个全局唯一的ID进行标识，一些NoSQL数据库从设计之初，就考虑了ID的不重复，而MySQL在这方面仍然需要借助一些特殊的手段来生成全局唯一的ID。可以考虑如下这些方式。

* 利用数据库自身的特性，在数据库启动参数里配置auto_increment_increment和auto_increment_offset，不过我们不推荐这种方式，因为这会导致数据库的维护成本上升。

* 配置一个单独的服务生成全局ID，可以是MySQL，也可以是NoSQL产品，甚至可以构建自己的专门用来生成唯一ID的服务，为了提高效率，还可以批量获取唯一的ID序列。

* 另外一种方式是，通过函数、程序算法或字段组合生成唯一ID，这种方式，可能会产生冲突，但是可以将这个冲突的概率做到非常小，我们更推荐使用这种方式。



## 11. 使用SQL生成升级SQL

可以使用SQL去生成升级的SQL文件，如使用CONCAT函数拼接生成SQL语句。例如，批量删除前缀为“prefix”的表，命令如下

```sql
SELECT CONCAT ('drop table ',table_name,';') INTO OUTFILE '/tmp/drop_table.sql' FROM information_schema.tables WHERE
table_name LIKE 'prefix%' AND table_schema='db_name';
```



# 六. 查询优化

## 1 基础知识

### 1.1. 查询优化的常用策略

* 优化数据访问

  应该尽量减少对数据的访问。一般有如下两个需要考虑的地方：应用程序应减少对数据库的数据访问，数据库应减少实际扫描的记录数。

* 重写SQL

  由于复杂查询严重降低了并发性，因此为了让程序更适于扩展，我们可以把复杂的查询分解为多个简单的查询。一般来说多个简单查询的总成本是小于一个复杂查询的。

  对于需要进行大量数据的操作，可以分批执行，以减少对生产系统产生的影响，从而缓解复制超时。

  由于MySQL连接（JOIN）严重降低了并发性，对于高并发，高性能的服务，应该尽量避免连接太多表，如果可能，对于一些严重影响性能的SQL，建议程序在应用层就实现部分连接的功能。这样的好处是：可以更方便、更高效地缓存数据，方便迁移表到另外的机器，扩展性也更好。

* 重新设计库表

  有些情况下，我们即使是重写SQL或添加索引也是解决不了问题的，这个时候可能要考虑更改表结构的设计。比如，可以增加一个缓存表，暂存统计数据，或者可以增加冗余列，以减少连接。优化的主要方向是进行反范式设计。

* 添加索引

  生产环境中的性能问题，可能80%的都是索引的问题，所以优化好索引。



### 1.2 优化器介绍



#### 1. 优化器的不足

MySQL优化器也有很多不足之处，它不一定能保证选择的执行计划就是最优的。

* 数据的统计信息有可能是错误的，对于复杂的查询，数据库可能会执行错误的执行计划，从而导致严重的性能问题。
* MySQL优化器的优化是基于简单的成本评估进行的，总是会选择成本更小的执行计划，其对成本衡量的标准是读取的随机块的数量，但是，本质上成本往往包括了诸多因素，CPU、内存、数据是否在缓存中，都是需要考虑到的因素，这样往往会导致MySQL计算得出的成本最小的执行计划不一定是响应最快的。
* 优化器不会考虑并发的情况，而实际的数据库执行，并发处理则是复杂的，资源的争用可能会导致性能问题。
* 一些商业数据库在执行的过程中会对各种优化结果的执行情况进行统计评估，以便自动改进后续的执行优化状况，而MySQL目前还没有这些功能。



#### 2. 优化器加提示

有时我们需要告诉优化器，让它按我们的意图生成执行计划，但是，加提示（hint）的方式不到万不得已，建议不要使用。

比较常用的加提示的方式：

* 使用索引（USE INDEX）

  USE INDEX(index_list)将告诉MySQL使用我们指定的索引去检索记录。

  ```sql
  SELECT * FROM table1 USE INDEX (col1_index,col2_index) WHERE col1=1 AND col2=2 AND col3=3;
  ```

* 不使用索引（IGNORE INDEX）

  IGNORE INDEX(index_list)将建议MySQL不使用指定的索引。

  ```sql
  SELECT * FROM table1 IGNORE INDEX (col3_index) WHERE col1=1 AND col2=2 AND col3=3;
  ```

* 强制使用索引（FORCE INDEX）

  使用USE INDEX指定了索引，但MySQL优化器仍然选择不使用我们指定的索引，这时可以考虑使用FORCE INDEX提示。

* 不使用查询缓冲（SQL_NO_CACHE）

  SQL_NO_CACHE提示MySQL对指定的查询关闭查询缓冲机制。

* 使用查询缓冲（SQL_CACHE）

  有时我们将查询缓冲设置为显式模式（explicitmode，query_cache_type=2），也就是说，除非指明了SQL需要缓存，否则MySQL是不考虑缓存它的，我们使用SQL_CACHE来指定哪些查询需要被缓存。

* STRAIGHT_JOIN

  这个提示将告诉MySQL按照FROM子句描述的表的顺序进行连接。

注意，USE INDEX、IGNORE INDEX和FORCE INDEX这些提示方式只会影响MySQL在表中检索记录或连接要使用的索引，它们并不会影响ORDERBY或GROUP BY子句对于索引的选择。



#### 3. MySQL的连接机制

MySQL优化器一般会选择更小的表或更小子集（满足查询条件的记录行数少）的表作为驱动表。随着驱动表（外部表）行数的增加，成本会增加得很快，选择更小的外部表或更小子集的外部表，是为了尽量减少嵌套连接的循环次数，而且，内部表一般在连接列有索引，索引一般常驻于内存中，这样可以保证很快完成连接。



## 2. 各种语句优化



### 2.1 连接的优化

由于连接的成本比较高，因此对于高并发的应用，应该尽量减少有连接的查询，连接的表的个数不能太多，连接的表建议控制在4个以内。

优化连接的一些要点：

* ON、USING子句中的列确认有索引。如果优化器选择了连接的顺序为B、A，那么我们只需要在A表的列上创建索引即可。
* 最好是能转化为INNER JOIN，LEFT JOIN的成本比INNER JOIN高很多。
* 使用EXPLAIN检查连接，留意EXPLAIN输出的rows列，如果rows列太高，比如几千，上万，那么就需要考虑是否索引不佳或连接表的顺序不当。
* 反范式设计，这样可以减少连接表的个数，加快存取数据的速度。
* 考虑在应用层实现连接。
* 一些应用可能需要访问不同的数据库实例，这种情况下，在应用层实现连接将是更好的选择。



### 2.2 GROUP BY、DISTINCT、ORDER BY语句优化

* 尽量对较少的行进行排序。

* 如果连接了多张表，ORDER BY的列应该属于连接顺序的第一张表。

* 利用索引排序，如果不能利用索引排序，那么EXPLAIN查询语句将会看到有filesort。

* GROUP BY、ORDER BY语句参考的列应该尽量在一个表中，如果不在同一个表中，那么可以考虑冗余一些列，或者合并表。

* 需要保证索引列和ORDER BY的列相同，且各列均按相同的方向进行排序。

* 增加sort_buffer_size。

  sort_buffer_size是为每个排序线程分配的缓冲区的大小。增加该值可以加快ORDER BY或GROUP BY操作。但是，这是为每个客户端分配的缓冲区，因此不要将全局变量设置为较大的值，因为每个需要排序的连接都会分配sort_buffer_size大小的内存。

* 增加read_rnd_buffer_size。

  当按照排序后的顺序读取行时，通过该缓冲区读取行，从而避免搜索硬盘。将该变量设置为较大的值可以大大改进ORDER BY的性能。但是，这是为每个客户端分配的缓冲区，因此你不应将全局变量设置为较大的值。相反，只用为需要运行大查询的客户端更改会话变量即可。

* 改变tmpdir变量指向基于内存的文件系统或其他更快的磁盘。

  如果MySQL服务器正作为复制从服务器被使用，那么不应将“--tmpdir”设置为指向基于内存的文件系统的目录，或者当服务器主机重启时将要被清空的目录。因为，对于复制从服务器，需要在机器重启时仍然保留一些临时文件，以便能够复制临时表或执行LOAD DATA INFILE操作。如果在服务器重启时丢失了临时文件目录下的文件，那么复制将会失败。

* 指定ORDER BY NULL。

  默认情况下，MySQL将排序所有GROUP BY的查询，如果想要避免排序结果所产生的消耗，可以指定ORDER BY NULL。

* 优化GROUP BY WITHROLLUP。

  GROUP BY WITHROLLUP可以方便地获得整体分组的聚合信息（superaggregation），但如果存在性能问题，可以考虑在应用层实现这个功能，这样往往会更高效，伸缩性也更佳。

* 使用非GROUP BY的列来代替GROUP BY的列

* 可以考虑使用Sphinx等产品来优化GROUP BY语句，一般来说，它可以有更好的可扩展性和更佳的性能。

  

### 2.3 优化子查询

对于数据库来说，在绝大部分情况下，连接会比子查询更快。使用连接的方式，MySQL优化器一般可以生成更佳的执行计划，可以预先装载数据，更高效地处理查询。而子查询往往需要运行重复的查询，子查询生成的临时表上也没有索引，因此效率会更低。



### 2.4 优化limit子句

Web应用经常需要对查询的结果进行分页，分页算法经常需要用到“LIMIT offset,row_count ORDERBYcol_id”之类的语句。一旦offset的值很大，效率就会很差，因为MySQL必须检索大量的记录（offset+row_count），然后丢弃大部分记录。

优化办法：

* 限制页数，只显示前几页，超过了一定的页数后，直接显示“更多（more）”，一般来说，对于N页之后的结果，用户一般不会关心。

* 要避免设置offset值，也就是避免丢弃记录。

* 使用Sphinx。

* 使用INNERJOIN。

  以下的例子中，先按照索引排序获取到id值，然后再使用JOIN补充其他列的数据。customers表的主键列是id列，name列上有索引，由于“SELECT id FROM customers…”可以用到覆盖索引，所以效率尚可。

  ```sql
  SELECT id, name, address, phone
  FROM customers
  INNER JOIN (
  SELECT id
  FROM customers
  ORDER BY name
  LIMIT 999,10)
  AS my_results USING(id);
  ```

  

### 2.5 优化IN列表

IN列表不宜过长，最好不要超过200。对于高并发的业务，小于几十为佳。如果能够将其转化为多个等于的查询，那么这种方式会更优。



### 2.6 优化UNION

UNION语句默认是移除重复记录的，需要用到排序操作，如果结果集很大，成本将会很高，所以，建议尽量使用UNION ALL语句。对于UNION多个分表的场景，应尽可能地在数据库分表的时候，就确定各个分表的数据是唯一的，这样就无须使用UNION来去除重复的记录了。

另外，查询语句外层的WHERE条件，并不会应用到每个单独的UNION子句内，所以，应在每一个UNION子句中添加上WHERE条件，从而尽可能地限制检索的记录数。



### 2.7 优化带有BLOB、TEXT类型字段的查询

由于MySQL的内存临时表不支持BLOB、TEXT类型，如果包含BLOB或TEXT类型列的查询需要用到临时表，就会使用基于磁盘的临时表，性能将会急剧降低。

规避BLOB、TEXT列的办法: 

* 使用SUBSTRING()函数
* 设置MySQL变量tmpdir，把临时表存放在基于内存的文件系统中。

优化的办法:

* 如果必须使用，可以考虑拆分表，把BLOB、TEXT字段分离到单独的表。
* 如果有许多大字段，可以考虑合并这些字段到一个字段，存储一个大的200KB比存储20个10KB更高效。
* 考虑使用COMPRESS()，或者在应用层进行压缩，再存储到BLOB字段中。



### 2.8 filesort的优化

filesort的字面意思可能会导致混淆，它和文件排序没有任何关系，可以理解为不能利用索引实现排序。

**两种filesort算法**

* two-pass

  列长度之和超过max_length_for_sort_data字节时就使用这个算法，其原理是：先按照WHERE筛选条件读取数据行，并存储每行的排序字段和行指针到排序缓冲（sort buffer）。如果排序缓冲大小不够，就在内存中运行一个快速排序（quick sort）操作，把排序结果存储到一个临时文件里，用一个指针指向这个已经排序好了的块。然后继续读取数据，直到所有行都读取完毕为止。。这是第一次读取记录。

  然后合并如上的临时文件，进行排序。

  然后依据排序结果再去读取所需要的数据，读入行缓冲（row buffer，由read_rnd_buffer_size参数设定其大小）。这是第二次读取记录。

  然排序字段是有序的，行缓冲里存储的行指针是有序的，但所指向的物理记录需要随机读，所以这个算法可能会带来很多随机读，从而导致效率不佳。

  优点： 排序的数据量较小，一般在内存中即可完成。

  缺点： 需要读取记录两次，第二次读取时，可能会产生许多随机I/O，成本可能会比较高。

* single-pass

  按筛选条件，把SQL中涉及的字段全部读入排序缓冲中，然后依据排序字段进行排序，如果排序缓冲不够，则会将临时排序结果写入到一个临时文件中，最后合并临时排序文件，直接返回已经排序好的结果集。

  优点： 不需要读取记录两次，相对于two-pass，可以减少I/O开销。

  缺点： 由于要读入所有字段，排序缓冲可能不够，需要额外的临时文件协助进行排序，导致增加额外的I/O成本。



**相关参数的设置和优化**

* max_length_for_sort_data：如果各列长度之和（包括选择列、排序列）超过了max_length_for_sort_data字节，那么就使用two-pass算法。如果排序BLOB、TEXT字段，使用的也是two-pass算法，那么这个值设置得太高会导致系统I/O上升，CPU下降，建议不要将max_length_for_sort_data设置得太高。
* max_sort_length：如果排序BLOB、TEXT字段，则仅排序前max_sort_length个字节。



优化方向

* 加大sort_buffer_size。

  一般情况下使用默认的single-pass算法即可。可以考虑加大sort_buffer_size以减少I/O。

* ·对于two-pass算法，可以考虑增大read_rnd_buffer_size，但由于这个全局变量是对所有连接都生效的，因此建议只在会话级别进行设置，以加速一些特殊的大操作。

* 在操作系统层面，优化临时文件的读写。



### 2.9 优化SQL_CALC_FOUND_ROWS

建议不要使用SQL_CALC_FOUND_ROWS这个提示，虽然它可以让开发过程变得简单一些，但并没有减少数据库所做的事情。



### 2.10 优化临时表

MySQL的临时表分为“内存临时表”和“磁盘临时表”，其中，内存临时表使用MySQL的MEMORY存储引擎。磁盘临时表使用MySQL的MyISAM存储引擎；一般

情况下，MySQL会先创建内存临时表，但当内存临时表超过配置参数指定的值后，MySQL会将内存临时表导出到磁盘临时表。

触发以下条件，会创建临时表。

* ORDERBY子句和GROUP BY子句引用的列不一样。
* 在连接查询中，ORDER BY或GROUP BY使用的列不是连接顺序中的第一个表。
* ORDERBY中使用了DISTINCT关键字。



如果查询创建了临时表（in-memory table）来排序或检索结果集，分配的内存大于tmp_table_size与max_heap_table_size参数之间的最小值，那么内存临时表就会转换为磁盘临时表（on-disk table），MySQL会在磁盘上创建磁盘临时表，这样会可能导致I/O瓶颈，进而影响性能。

* tmp_table_size：指定系统创建的内存临时表的最大大小。
* max_heap_table_size：指定用户创建的内存表的最大大小。



可能会导致使用到磁盘临时表：

* 表中有BLOB或TEXT字段。
* 使用UNION或UNION ALL时，SELECT子句中包含了大于512字节的列。



避免临时表的方法：

* 创建索引：在ORDERBY或GROUP BY的列上创建索引。

* 分拆长的列：一般情况下，TEXT、BLOB，大于512字节的字符串，基本上都是为了显示信息，而不会用于查询条件，因此设计表的时候，可以考虑将这些列分离到另外一张表中。

* 不需要用DISTINCT时就没必要用DISTINCT，能用UNION ALL就不要用UNION。

  

## 3. OLAP业务优化

OLAP类型的业务需要考虑的一些要点。

* 使用冗余数据

  有时最好的办法是在表中保存冗余的数据，虽然这些冗余数据有时也可以由其他的列推断得出。冗余数据可以让查询执行得更快。

* 计算复用，使用缓存表

  我们可以使用缓存表存储一些结果，这里所说的“缓存表”，意思是这些值在逻辑上是冗余的，可以从原始表中获取到，但显然从原始表中获取数据更慢。

* 预计算

  预先对一些常用的大查询生成汇总表。

* 统计框架的改善

  需要将一个复杂的查询任务放在一个SQL查询中来完成，往往会导致性能问题，使用这种方式最常见的原因是你正在使用一个编程框架或一个可视化组件库直接和数据源相连，然后在程序里直接展示数据，简单的商务智能和报表工具都属于这一分类。
