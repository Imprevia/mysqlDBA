# 三、开发基础

## 1. 数据模型

### 1.1 关系数据模型介绍

**关于NULL**

如果某个字段的值是未知的或未定义的，数据库会提供一个特殊的值NULL来表示。NULL值很特殊，在关系数据库中应该小心处理。例如查询语句“select*from employee where 绩效得分<=85 or>绩效得分>85；” 就不能查询出绩效得分是未知的（NULL）的数据。

**关于key和索引**

key常指表中能唯一标识一笔记录的字段（属性）或多个字段的组合。现实中，key和索引可以简单地看作同义词，key不一定唯一标识。数据库管理系统为了高效地检索记录，往往会创建各种索引结构加速检索记录，或者按照索引/key的顺序存储记录，所以基于记录的索引/key会很容易查找到记录。

**实体–关系建模**

- ER建模

  ER建模是一种自上而下的数据库设计方法。我们通过标识模型中必须要表示的重要数据（称为实体）及数据之间的关系开始ER建模，然后增加细节信息，如实体和关系所要具有的信息（称为属性）。该方法的输出是实体类型、关系类型和约束条件的清单。

- UML

  UML是一种分析人员和开发人员广泛使用的标准建模语言，它可以以图形化的方式表示实体、关系。

- 实体

  实体代表现实世界的一组对象集合，可以粗略地认为它是名词，如学生、雇员、订单、演员、电影。实体一般用矩形来表示。

- 关系

  关系指特定实体之间的关系。可以粗略地认为是动词，如公司拥有员工、演员演电影。关系用线来表示。一般为二元关系。二元关系的基数就是我们所说的一对一、一对多、多对多。在数据库设计中，需要选择合适的基数表示法，如IDEF1X表示法、关系表示法或Crow’s foot表示法。

  对于Crow’s foot表示法，实体表示为矩形框，关系表示为矩形框之间的线，线两端的形状表示关系的基数。空心圆表示零或多，单阴影线标记表示一或多，单阴影线标记和空心圆表示零或一，双阴影线标记表示恰好为一。



## 2. SQL基础

### 2.1 变量

MySQL里的变量可分为用户变量和系统变量。

**用户变量**

用户变量与连接有关。也就是说，一个客户端定义的变量不能被其他客户端看到或使用。当客户端退出时，该客户端连接的所有变量将自动释放。这点不同于在函数或存储过程中通过DECLARE语句声明的局部变量，局部变量的生存周期在它被声明的“BEGIN…END”块内。对于用户变量的值，可以先保存在用户变量中，然后以后再引用它；这样就可以将值从一个语句传递到另外一个语句。

设置用户变量执行SET语句

```sql
SET @var_name= expr[, @var_name= expr] ...
```



**系统变量**

MySQL服务器维护着两种系统变量：

- 全局变量影响MySQL服务的整体运行方式；

  所有全局变量初始化为默认值。这些默认值可以在选项文件中或在命令行中对指定的选项进行更改。服务器启动后，通过连接服务器并执行SETGLOBAL var_name语句，可以动态更改这些全局变量。要想更改全局变，必须具有SUPER权限。修改了全局变量，只影响更改后连接的客户的相应会话变量。

- 会话变量影响具体客户端连接的操作。

  服务器还为每个连接的客户端维护一系列的会话变量。在连接时使用相应全局变量的当前值对客户端的会话变量进行初始化。对于动态会话变量，客户端可以通过SET SESSION var_name语句更改它们。设置会话变量不需要特殊权限，但客户端只能更改自己的会话变量，而不能更改其他客户端的会话变量。



### 2.2 保留字

- ‌**A**‌：ALL, ALTER, AND, ANY, ARRAY, AS, ASC
- ‌**B**‌：BEGIN, BETWEEN, BIGINT, BINARY, BLOB, BOOLEAN, BY, BYTES
- ‌**C**‌：CASE, CAST, CHAR, CHECK, COLUMN, COMPUTE, CONDITION, CONSTRAINT, CONTINUE, CREATE
- ‌**D**‌：DATABASE, DATE, DATETIME, DECIMAL, DECLARE, DEFAULT, DELETE, DESC, DESCRIBE, DISTINCT, DIV, DOUBLE
- ‌**E**‌：ELSE, ESCAPE, EXCEPT, EXISTS, EXPLAIN, EXTRACT
- ‌**F**‌：FALSE, FETCH, FLOAT
- ‌**G**‌：GEO2D, GRANT, GROUP, GROUPING, GROUP_CONCAT
- ‌**H**‌：HAVING
- ‌**I**‌：IDENTIFIED, IF, IGNORE, IN, INDEX, INNER, INSERT, INT
- ‌**J**‌：JOIN
- ‌**K**‌：KEY
- ‌**L**‌：LEAVE, LEFT, LIKE, LIMIT, LOCK
- ‌**M**‌：MATCH, MEDIUMINT, MERGE, MINUS
- ‌**N**‌：NATURAL
- ‌**O**‌：ON, OPEN, OR
- ‌**P**‌：PRIMARY
- ‌**R**‌：RELEASE



### 2.3 数据类型

MySQL支持常用的数据类型：数值类型、日期/时间类型和字符串（字符）类型。

**数值类型**

数值类型可分为两类：整型和实数。对于实数，MySQL支持确切精度的值（定点数）和近似精度的值（浮点数）。确切精度的数值类型有DECIMAL类型，近似精度的数值类型有单精度（FLOAT）或双精度（DOUBLE）两种类型。

1. 整型

   整型包括TINYINT、SMALLINT、MEDIUMINT、INT、BIGINT，下面是各种整型的空间占用及表示的数值范围。无符号（unsigned）属性可扩展一倍的最大值上限。

   | 类型      | 字节 | 最小值                      | 最大值                                        |
   | --------- | ---- | --------------------------- | --------------------------------------------- |
   | TINYINT   | 1    | -128<br />0                 | 127<br />255                                  |
   | SMALLINT  | 2    | -32768<br />0               | 32767<br />65535                              |
   | MEDIUMINT | 3    | -8388608<br />0             | 8388607<br />16777215                         |
   | INT       | 4    | -2147483648<br />0          | 2147483647<br />4294967295                    |
   | BIGINT    | 8    | -9223372036854775808<br />0 | 9223372036854775807<br />18446744073709551615 |

2. DECIMAL和NUMERIC类型（定点数）

   定点数也就是DECIMAL型，指的是数据的小数点的位置是固定不变的。也就是说，小数点后面的位数是固定的。

   DECIMAL和NUMERIC在MySQL中被视为相同的类型。它们用于保存必须为确切精度的值，例如货币数据。

3. FLOAT和DOUBLE类型（浮点数）

   单精度值（FLOAT）使用4个字节，双精度值（DOUBLE）使用8个字节。浮点数可以比整型、定点数表示更大的数值范围。

   为了保证最大可能的可移植性，对于使用近似数值存储的代码，应使用FLOAT或DOUBLE来表示，不规定精度或位数。由于浮点数存在误差问题，如果用到浮点数，要特别注意误差的问题，并尽量避免做浮点数比较。



**日期/时间类型**

表示时间值的日期和时间类型有DATETIME、DATE、TIMESTAMP、TIME和YEAR等。每个时间类型都有一个有效值范围，TIMESTAMP类型有其特有的自动更新特性。

* 可以使用ALLOW_INVALID_DATES SQL模式让MySQL接受不合法的日期，例如'1999-11-31'

* 不使用NO_ZERO_DATE SQL模式，MySQL还允许将'0000-00-00'保存为“伪日期”。这在某些情况下比使用NULL值更方便，并且数据和索引占用的空间更小。

  

1. DATETIME、DATE和TIMESTAMP类型

   1. 当需要同时包含日期和时间信息的值时，建议使用DATETIME（日期时间组合）类型。支持的范围为'1000-01-0100:00:00'到'9999-12-3123:59:59'。DATETIME类型占8个字节。

   2. 当只需要日期值而不需要时间部分时，建议使用DATE（日期）类型。支持的范围是'1000-01-01'到'9999-12-31'。DATE类型占3个字节。

   3. TIMESTAMP（时间戳）列用于在进行INSERT或UPDATE操作时记录日期和时间。显示宽度固定在19个字符，并且格式为'YYYY-MM-DDHH:MM:SS'。TIMESTAMP的范围是从'1970-01-

      0100:00:01'UTC到'2038-01-0903:14:07'UTC。TIMESTAMP类型占4个字节。

2. TIME（时间）类型

   该时间类型的范围是'-838:59:59'到'838:59:59'。TIME类型占3个字节。

3. YEAR（两位或四位格式的年）类型

   YEAR类型表示两位或四位格式的年。默认是四位格式，在四位格式中，允许的值是1901~2155和0000。

   在两位格式中，如果是两位字符串，那么范围为'00'\~'99'。'00'\~'69'和'70'\~'99'范围的值被分别转换为2000\~2069和1970\~1999范围的YEAR值。

   如果是两位整数，范围为1\~99。1\~69和70\~99范围的值被分别转换为2001\~2069和1970\~1999范围的YEAR值。

   YEAR类型占1个字节。



**字符串类型**

字符串类型指CHAR、VARCHAR、BINARY、VARBINARY、BLOB、TEXT、ENUM和SET。

1. CHAR和VARCHAR类型

   CHAR与VARCHAR类型类似，但它们保存和检索数据的方式不同。在保存VARCHAR的值时，只保存需要的字符数，然后用1~2个字节来存储值的长度，所以如果是很短的值（如仅一个字符），那么耗费的存储空间比CHAR还会多些，所以，如果想存储很短的类型，使用CHAR会更合适。VARCHAR可选的一种场景是最长记录的长度值比平均长度的值大得多。

   CHAR是固定长度的字符串，它的长度固定为创建表时声明的长度。长度范围为0到255个字符。当保存CHAR值时，在它们的右边填充空格以达到指定的长度。当检索到CHAR值时，尾部的空格会被删除掉，这是MySQL服务器级别控制的，和存储引擎无关。CHAR类型适合存储大部分值的长度都差不多的数据，例如MD5值。

   VARCHAR列中的值为可变长度的字符串。长度可以指定为0到65535之间的值

2. BINARY和VARBINARY类型

   BINARY和VARBINARY类似于CHAR和VARCHAR，不同的是，它们包含的是二进制字符串而不是非二进制字符串。也就是说，它们包含的是字节字符串而不是字符字符串，它们的长度是字节长度而不是字符长度。这说明它们没有字符集，并且排序和比较也是基于字节的二进制值进行的。相对来说，二进制字符串的比较比字符字符串的比较更为简单有效。

3. BLOB和TEXT类型

   BLOB是一个二进制大对象，可以容纳可变数量的数据。BLOB类型共有4种：TINYBLOB、BLOB、MEDIUMBLOB和LONGBLOB。BLOB是SMALLBLOB的同义词。

   TEXT类型也有4种：TINYTEXT、TEXT、MEDIUMTEXT和LONGTEXT。它们分别对应上面的4种BLOB类型，有相同的最大长度和存储需求。TEXT是SMALLTEXT的同义词。

   BLOB用于存储二进制字符串（字节字符串），而TEXT列则被视为非二进制字符串（字符字符串）的存储方式，它是有字符集和排序规则的。

   在大多数情况下，可以将BLOB列视为能够存储足够大数据的VARBINARY列。同样，也可以将TEXT列视为VARCHAR列。但是，BLOB和TEXT在以下几个方面不同于VARBINARY和VARCHAR。

   * 保存或检索BLOB和TEXT列的值时不用删除尾部的空格。
   * 对于BLOB和TEXT列的索引，必须指定索引前缀的长度。
   * BLOB和TEXT列不能有默认值。
   * 排序时只使用该列的前max_sort_length个字节。max_sort_length的默认值是1024。

   BLOB或TEXT对象的最大长度由其类型来确定，但在客户端和服务器之间实际可以传递的最大数据量是由可用内存数量和通信缓存区的大小来确定的。可以通过更改max_allowed_packet变量的值更改消息缓存区的大小，但必须同时修改服务器和客户端的程序。

   使用BLOB、TEXT等大字段可能会导致严重的性能问题，比如导致产生磁盘临时表。

4. ENUM类型

   ENUM（枚举）类型是一个字符串对象，其值通常选自一个允许值列表，该列表是在创建表时被定义的。慎重使用。

   

### 2.4 选择合适的数据类型

MySQL支持许多数据类型，选择合适的数据类型可以获得更好的性能，从而更节省空间。

指导原则：

1. 各表使用一致的数据类型

   字段在每个表中都应该使用一样的数据类型、长度，因为以后可能需要进行JOIN（连接）操作，这样做是为了避免无谓的转换或可能出现不期望的结果。我们不仅要考虑数据类型是如何存储的，也要清楚数据类型是如何计算和比较的。

2. 小往往更好

   选择更短的数据类型。更短的类型意味着更少的磁盘空间、更少的内存空间、更少的CPU处理时间。

3. 简单类型更好

   简单的数据类型能够进行更快的处理。

4. 尽可能避免NULL值

   应尽量显式定义“not NULL”，如果查询涉及的是NULL值的字段，MySQL会很难去优化查询。可使用0、空字符串或特殊的值来代替NULL存储。当然，也不要去刻意追求“not NULL”，因为更改NULL字段为“not NULL”，对性能的提升可能没什么太大的作用，让设计更自然、更具可理解性应该更值得看重。



### 2.5 函数

**数值函数**

* 算数操作符

  可使用常见的算数操作符。例如‘+’、‘-’、‘*’、‘/’、DIV（整除）。

* 数学函数

  * ABS(X)：X的绝对值。
  * CEIL(X)：返回不小于X的最小整数值。
  * FLOOR(X)：返回不大于X的最大整数值。
  * CRC32(X)：计算循环冗余码校验值并返回一个32比特无符号值。
  * RAND()、RAND(N)：返回一个随机浮点值v，范围在0到1之间（即其范围为0≤v≤1.0）。若已指定一个整数参数N，则它被用作种子值，用来产生重复序列。
  * SIGN(X)：返回X的符号。
  * TRUNCATE(X,D)：返回被舍去至小数点后D位的数字X。若D的值为0，则结果不带有小数点或不带有小数部分。
  * ROUND(X)、ROUND(X,D)：返回参数X，其值接近于最近似的整数。在有两个参数的情况下，返回X，其值保留到小数点后D位，而第D位的保留方式为四舍五入。若要保留X值到小数点左边的D位，可将D设为负值，例如，ROUND（123.45，-1）的结果是120，ROUND（167.8，-2）的结果是200。

​	

**字符类型处理函数**

* CHAR_LENGTH(str)：返回值为字符串str的长度，长度的单位为字符。一个多字节字符算作一个单字符。对于一个包含5个二字节的字符集，LENGTH()的返回值为10，而CHAR_LENGTH()的返回值为5。
* LENGTH(str)：返回值为字符串str的长度，单位为字节。
* CONCAT(str1,str2,...)：返回结果为连接参数产生的字符串。如下查询将拼接'My'、'S'、'QL'3个字符串。
* LEFT(str,len)：从字符串str开始，返回最左len个字符。
* RIGHT(str,len)：从字符串str开始，返回最右len个字符。
* SUBSTRING(str,pos)、SUBSTRING(str,pos,len)：不带有len参数的格式是从字符串str返回一个子字符串，起始于位置pos。带有len参数的格式是从字符串str返回一个长度同len字符相同的子字符串，起始于位置pos。
* LOWER(str)：返回字符串str转化为小写字母的字符。
* UPPER(str)：返回字符串str转化为大写字母的字符。



**日期和时间函数**

* NOW()：返回当前日期和时间的值，其格式为'YYYY-MM-DDHH:MM:SS'或YYYYMMDDHHMMSS。

* CURTIME()：将当前时间以'HH:MM:SS'或HHMMSS的格式返回。
* CURDATE()：将当前日期按照'YYYY-MM-DD'或YYYYMMDD格式的值返回。
* DATEDIFF(expr1,expr2)：是返回开始日期expr1与结束日期expr2之间相差的天数，计算中只用到这些值的日期部分。返回值为正数或负数。
* DATE_ADD(date,INTERVAL expr type)、DATE_SUB(date,INTERVAL expr type)：这些函数执行日期运算。date是一个DATETIME或DATE值，用来指定起始时间。expr是一个表达式，用来指定从起始日期添加或减去的时间间隔值。type为关键词，它指示了表达式被解释的方式。type常用的值有SECOND、MINUTE、HOUR、DAY、WEEK、MONTH、YEAR。
* DATE_FORMAT(date,format)：下面的代码会根据format字符串安排date值的格式。常用的日期格式'YYYY-MM-DDHH:MM:SS'
* STR_TO_DATE(str,format)：是DATE_FORMAT()函数的倒转。它将获取一个字符串str和一个格式字符串format。
* 若格式字符串包含日期和时间部分，则STR_TO_DATE()返回一个DATETIME值，若该字符串只包含日期部分或只包含时间部分，则返回一个DATE或TIME值。



### 2.6 操作符及优先级

以下是按照从低到高的优先级列出的各种运算操作符:

* :=
* ||、OR、XOR
* &&、AND
* NOT
* BETWEEN、CASE、WHEN、THEN、ELSE
* =、<=>、>=、>、<=、<、<>、!=、IS、LIKE、REGEXP、IN
* |
* &
* <<、>>
* -、+
* *、/（DIV）、%（MOD）
* ^（按位异或）
* -（负号）、~（按位取反）
* !



### 2.7 SQL语法

#### 1. 数据定义语句（DDL）

* 创建表

  ```sql
  CREATE TABLE employees_2 (
    emp_no int(11) NOT NULL,
    birth_date date NOT NULL,
    first_name varchar(14) NOT NULL,
    last_name varchar(16) NOT NULL,
    gender enum('M','F') NOT NULL,
    hire_date date NOT NULL,
    primary key (emp_no)
  ) engine=innodb default charset=latin1
  ```

* 删除表

* ```sql
  DROP TABLE employees_2;
  ```

* 修改表

  ```sql
  # 修改表名
  ALTER TABLE t1 RENAME t2;
  # 添加列
  ALTER TABLE t2 ADD d TIMESTAMP;
  # 添加索引
  ALTER TABLE t2 ADD INDEX (d), ADD INDEX (a);
  # 删除列
  ALTER TABLE t2 DROP COLUMN c;
  # 修改列类型
  ALTER TABLE t2 MODIFY a TINYINT NOT NULL;
  # 修改列名
  ALTER TABLE t2 CHANGE b c CHAR(20);
  ```

  `CHANGE`和`MODIFY`都用于修改表的列属性，但它们之间存在一些差异：

  - 重命名列。`CHANGE`允许修改列名，而`MODIFY`不支持重命名列。12345678

  - 数据类型和约束修改。`CHANGE`和`MODIFY`都可以修改列的数据类型，但`CHANGE`还可以修改列的其他属性，如长度、默认值、是否为空等。
  - 使用频率。`MODIFY`通常用于进行较小的修改，例如修改列的数据类型，而`CHANGE`用于进行幅度较大的修改，如修改列名。

* 验证表结构

  ```sql
  DESC employees_2
  ```

* 创建索引

  ```sql
  # 在表的列id上创建索引。
  CREATE INDEX id_index ON employees_2 (id);
  # 在customer表的name列上创建一个索引，索引使用name列的前10个字符。
  CREATE INDEX part_of_name ON customer (name(10));
  # 在tbl_name表的a、b、c列上创建一个复合索引
  CREATE INDEX idx_a_b _c ON tbl_name(a,b,c);
  ```

* 句删除索引

  ```sql
  # 删除表tbl_name上的index_name索引时使用如下命令。
  DROP INDEX index_name ON tbl_name;
  ```

* 修改字符集和排序规则

  ```sql
  # 更改排序字符集
  ALTER TABLE test.tt1 CHANGE v2 v2 VARCHAR(10) CHARACTER SET utf8 COLLATE utf8_general_ci;
  # 更改排序规则
  ALTER TABLE table_name CHANGE col_a col_a VARCHAR(50) CHARACTER SET latin1 COLLATE latin1_bin
  ```

  

#### 2. 数据操作语句（DML）

* INSERT语句

  ```sql
  INSERT INTO table_name (column1, column2....)
  VALUES (value1, value2...);
  ```

* 修改数据（UPDATE）

  ```sql
  UPDATE table_name SET column_name1 = value1,column_name2 = value2,column_name3 = value3 ...
  [WHERE conditions];
  ```

* 删除（DELETE）

  ```sql
  DELETE FROM table_name [WHERE conditions];
  ```

* SELECT语句

  ```sql
  SELECT column_names FROM table_name [WHERE ...conditions];
  ```

  

**SQL模式匹配**

SQL有两个通配符，“-”匹配任意单个字符，“%”匹配任意多个字符（包括0个字符）。

模式匹配默认是区分大小写的，它一般使用LIKE或NOT LIKE这些比较操作符。

```sql
SELECT * FROM employees WHERE first_name LIKE 'D%';
SELECT * FROM employees WHERE first_name LIKE 'Ang__'
```



**逻辑操作符与或非（AND、OR、NOT）**

可以用逻辑操作符组合成多个筛选条件



**范围操作符IN和BETWEEN**

```sql
SELECT * FROM employees WHERE birth_date IN ('1964-06-01','1964-06-02','1964-06-04');
SELECT * FROM employees WHERE birth_date BETWEEN '1964-06-01' AND '1964-06-04';
```



**限制获取记录数（使用LIMIT子句）**

```sql
SELECT * FROM employees LIMIT 5;
```



**排序（ORDERBY）**

```sql
SELECT * FROM employees ORDER BY birth_date ASC LIMIT 10;
```



**数据计算**

```sql
SELECT
  emp_no,
  first_name,
  last_name,
  birth_date,
  curdate(),
  timestampdiff(year,birth_date,curdate()) as age
FROM
  employees
ORDER BY first_name , last_name
LIMIT 10;
```



**使用DISTINCT获取不重复的唯一值**

```sql
SELECT DISTINCT first_name FROM employees ;
```



**聚集函数COUNT、MIN、MAX、AVG、SUM**

```sql
SELECT COUNT(*) FROM employees;
SELECT MIN(emp_no) FROM employees;
SELECT MAX(emp_no) FROM employees;
SELECT AVG(salary) FROM salaries WHERE to_date='9999-01-01';
SELECT SUM(salary) FROM salaries WHERE to_date='9999-01-01';
```



**分组统计GROUP BY子句**

一般将GROUP BY语句和聚集函数一起使用，从而实现分组统计。

```sql
SELECT MAX(a),MAX(b),MAX(c) FROM table_name WHERE ... GROUP BY d ;
```

可以在GROUP BY语句后添加HAVING子句，并对聚集结果进行筛选。

```sql
SELECT first_name,COUNT(*) cnt FROM employees GROUP BY first_name HAVING cnt > 270 ORDER BY cnt DESC ;
```



**并集操作（UNION和UNION ALL）**

UNION和UNION ALL都是将两个结果集合并为一个，但UNION ALL比UNION更快。UNION实际上是UNION DISTINCT，在进行表连接后会筛选掉重复的记录，而UNION ALL则是不管有没有重复记录，都直接返回合并后的记录。

```sql
SELECT * FROM a
UNION ALL
SELECT * FROM b;
```



**NULL值**

NULL值的判断一般使用IS NULL或IS NOTNULL



#### 3. JOIN（连接）

MySQL使用JOIN来连接多个表查询数据，主要使用的JOIN算法只有一种，那就是nested-loop join。

nested-loop join算法实现的机制很简单，就是从驱动表中选取数据作为循环基础数据，然后以这些数据作为查询条件到下一个表中进行查询，如此往复。这个实现机制类似于foreach函数的遍历。因此带来的问题就是连接的表越多，函数嵌套的层数就越多，算法复杂度呈指数级增长。

JOIN语句的含义是把两张（或者多张）表的属性通过它们的值组合在一起，一般会遇到如下3种连接：

* 等值连接（[INNER]JOIN）
* 左外连接（LEFT JOIN）
* 右外连接（RIGHT JOIN）



**内连接（INNER JOIN）**

内连接可以被进一步分为等值连接、自然连接和交叉连接。较常用的是等值连接。



**外连接（OUT JOIN）**

外连接可依据连接表保留左表、右表或全部表的行而进一步分为左外连接、右外连接和全外连接。

* 左外连接（LEFT JOIN、LEFT OUTER JOIN）

  左外连接也简称为左连接（LEFT JOIN），若A和B两表进行左外连接，那么结果表中将包含“左表”（即表A）的所有记录，即使那些记录在“右表”B中没有符合连接条件的匹配。这就意味着即使ON语句在表B中的匹配项是0条，连接操作也还是会返回一条记录，只不过这条记录中的来自于表B的每一列的值都为NULL。

* 右外连接（RIGHT JOIN、RIGHTOUT JOIN）

  右外连接也简称右连接（RIGHT JOIN），它与左外连接完全类似，只不过是连接表的顺序相反而已。如果A表右连接B表，那么“右表”B中的每一行在连接表中至少会出现一次。如果B表的记录在“左表”A中未找到匹配行，则连接表中来源于A表中的列的值将设为NULL



#### 4. 子查询

子查询是指查询语句里的SELECT语句。

```sql
SELECT * FROM t1 WHERE column1 = (SELECT column1 FROM t2);
```

这个子查询是嵌套在外部查询中的，子查询嵌套的层次不宜过多，否则性能可能会很差。

MySQL对于子查询的优化不佳，由于子查询一般可以改写成JOIN语句，因此一般建议使用JOIN的方式查询数据。



## 3. 索引

数据库索引，是数据库管理系统中一个排序的数据结构，用于协助快速查询、更新数据库表中的数据。

目前MySQL主要支持的几种索引有：B树索引（B-tree）、散列索引（hash）、空间索引（R-tree）和全文索引（full-text）。由于索引是在存储引擎层实现的，所以不同的存储引擎的索引实现会有一些差异。

逻辑上又可以分为：单列索引、复合索引（多列索引）、唯一（Unique）索引和非唯一（NonUnique）索引。

如果索引键值的逻辑顺序与索引所服务的表中相应行的物理顺序相同，那么该索引被称为簇索引（cluster index），也称为

聚集索引、聚簇索引，也就是说数据和索引（B+树）在一起，记录被真实地保存在索引的叶子中，簇索引也称为索引组织表，

反之为非聚集索引。

簇索引的优点如：

* 将相关的的数据保持在一起，叶子节点内可保存相邻近的记录。
* 因为索引和数据存储在一起，所以查找数据通常比非簇索引更快。由于主键是有序的，很显然，对于InnoDB表，最高效的存取方式是按主键存取唯一记录或进行小范围的主键扫描。

簇索引的不足：

* 簇索引对I/O密集型的负荷性能提升最佳，但如果数据是在内存中（访问次序不怎么重要），那么簇索引并没有明显益处。

* 插入操作很依赖于插入的顺序，按primary key的顺序插入是最快的。
* 更新簇索引列的成本比较高，因为InnoDB不得不将更新的行移动到新的位置。
* 全表扫描的性能不佳，尤其是数据存储得不那么紧密时，或者因为页分裂（pagesplit）而导致物理存储不连续。
* 二级索引的叶节点中存储了主键索引的值，如果主键采用的是较长的字符，那么索引可能会很大，且通过二级索引查找数据也需要进行两次索引查找。



### 3.1 使用索引的场景及注意事项

* MySQL目前仅支持前导列

  筛选记录的条件应能组成复合索引最左边的部分，即按最左前缀的原则进行筛选。

  ```sql
  # 复合索引idx_a_b_c
  CREATE INDEX idx_a_b_c ON tb1(a,b,c);
  # 只有使用如下条件才可能应用到这个复合索引
  WHERE a=?
  WHERE a=? AND b=?
  WHERE a=? AND b=? AND c=?
  WHERE a=? AND c=? #注意这个查询仅仅利用了MySQL索引的a列信息
  ```

* 索引列上的范围查找

  在对某个条件进行范围查找时，如果这个列上有索引，且使用的是WHERE…BETWEEN…AND…、>、<等范围操作符时，那么可能就会用到索引范围查找。一般应该避免大范围的索引范围查找，如果索引范围查找的成本太高，那么数据库可能会选择全表扫描的方式。

  *IN(...)并不属于范围查找的范畴。*

* JOIN列

  在联合查询两个表时，比如查询语句为“SELECT a.col1,b.col2 FROMaJOIN b ON a.id=b.id”，其中id为主键，若a表是驱动表，那么数据库可能全表扫描a表，并用a表的每个id去探测b表的索引查找匹配的记录。

* WHERE子句

  WHERE子句的条件列是复合索引前面的索引列再加上紧跟的另一个列的范围查找。

  ```sql
  CREATE INDEX idx_a_b_c_d ON tb1(a,b,c,d);
  # 只有使用如下条件才可能应用到这个复合索引。
  WHERE a=? AND b=? AND c > 10000;
  WHERE a=? AND b=? AND c=? AND d<10000;
  # 下面d<100000这个筛选操作并不会走索引
  WHERE a=? AND b=? AND c > 10000 AND d<100000;
  ```

* MySQL优化器

  MySQL优化器会做一些特殊优化，比如对于索引查找MAX（索引列），那么可以进行直接定位。

​	

**注意事项和建议**

* WHERE条件中的索引列不能是表达式的一部分，MySQL也不支持函数索引。
* InnoDB的非主键索引存储的不是实际记录的指针，而是主键的值，所以主键最好是整型值，如自增ID，基于主键存取数据是最高效的，使用二级索引存取数据则需要进行两次索引查找。
* 最好是按主键的顺序导入数据，如果导入大量随机id的数据，那么可能需要运行OPTIMIZE TABLE命令来优化表。
* 索引应尽量是高选择性的，而且需要留意“基数（cardinality）”值，基数指的是一个列中不同值的个数，显然，最大基数意味着该列中的每个值都是唯一的，最小基数意味着该列中的所有值都是相同的。索引列的基数相对于表的行数较高时（也就是说重复值更少），索引的工作效果更好。
* 使用更短的索引。可以考虑前缀索引，前缀索引仅索引前面一部分字符（值），但应确保所选择的前缀的长度可以保证大部分值是唯一的。
* 索引太多时可能会浪费空间，且降低修改数据的速度。所以，不要创建过多的索引，也不要创建重复的索引。MySQL允许在同样的列上创建多个索引而不会提示报错，一些其他分支的版本有统计信息可以甄别出没有被使用的索引。而对于官方版本，你可能需要借助工具清理掉过多的重复索引。
* 如果是唯一值的列，创建唯一索引会更佳，也可以确保不会出现重复数据。
* 使用覆盖索引（covering index）也可以大大提高性能。所谓“覆盖索引”是指所有数据都可以从索引中得到，而不需要去读取物理记录。
* 利用索引来排序。MySQL有两种方式可以产生有序的结果。一种是使用文件排序（filesort）来对记录集进行排序，另一种是扫描有序的索引。我们应尽量利用索引来排序。

* 添加冗余索引，需要权衡。

  如果已有一个索引（columnA），那么一个新的索引（columnA，columnB）就是冗余索引，因为后面的索引包含了前面索引的所有信息。

  比如原来的索引是创建在一个整型列上的，要是再添加一个很长的字符列，那么索引会变得很大，从而影响性能。这种情况下，可能不得不选择添加新的复合索引，保留原来的索引，这样做的不利之处是增加了索引维护的开销，而且一个新的索引也需要占据内存空间。



### 3.2 索引的错误用法

以下是生产环境中常犯的一些错误，而且由于表结构不易调整，因此往往会导致严重的性能影响。

* 创建了太多的索引或无效的索引。比如在WHERE条件的每个列上都建立单独的索引，当单个索引效率不高的时候，MySQL往往就会选择全表扫描，太多的索引可能会导致索引所占用的磁盘空间比实际数据还大得多。
* 对于复合索引，如果不考虑ORDERBY、GROUP BY这样的一些操作，那么把最具选择性的列放在前面是合适的，复合索引主要用于优化WHERE查找。但如果是排序之类的操作，把最具选择性的列放在前面则不一定最有效，因为避免随机I/O和排序可能才是我们更值得考虑的。
* 忽略了值的分布。某些值只有少量记录，查询对这些值的筛选执行就会很快，而某些值即使经过了索引筛选，满足条件的仍然还有大量的记录，这样索引效果就会很差。一般来说，数据表内值的分布应该尽量均匀，由于MySQL的统计信息不完善，数据分布不均匀很可能会产生很差的执行计划，导致严重的性能问题。
* InnoDB主键过长，导致二级索引过大。主键的选择，一般建议是整型。



### 3.3 如何使用EXPLAIN工具

使用EXPLAIN工具可以确认执行计划是否良好，查询是否走了合理的索引。

使用EXPLAIN命令查看执行计划

```sql
EXPLAIN SELECT……
```

EXPLAIN命令还有如下两种变体:

* EXPLAIN EXTENDEDSELECT……

  以上命令将执行计划“反编译”成SELECT语句，运行SHOW WARNINGS可得到被MySQL优化器优化后的查询语句。

* EXPLAIN PARTITIONS SELECT……

  以上命令用于分区表的EXPLAIN命令



**执行计划包含的信息及解读**

详细阐述EXPLAIN输出的各项内容：

* id：id包含一组数字，表示查询中执行SELECT子句或操作表的顺序。

  * 如果id相同，则执行顺序由上至下。
  * 如果是子查询，id的序号会递增，id值越大则优先级越高，越先被执行。
  * 如果id相同，则可以认为它们是一组，从上往下顺序执行。在所有组中，id值越大，优先级就越高，越先执行。

* select_type

  select_type表示查询中每个SELECT子句的类型（是简单还是复杂）。

  * SIMPLE：查询中不包含子查询或UNION。
  * 查询中若包含任何复杂的子部分，最外层查询则被标记为PRIMARY。
  * 在SELECT或WHERE列表中若包含了子查询，则该子查询被标记为SUBQUERY。
  * 在FROM列表中包含的子查询将被标记为DERIVED（衍生）。
  * 若第二个SELECT出现在UNION之后，则被标记为UNION；若UNION包含在FROM子句的子查询中，则外层SELECT将被标记为DERIVED。
  * 从UNION表中获取结果的SELECT将被标记为UNION RESULT。

* type

  type表示MySQL在表中找到所需行的方式，又称“访问类型”。下面类型，由上至下，由最差到最好。

  * ALL：FullTable Scan，MySQL将遍历全表以找到匹配的行。
  * index：FullIndex Scan，index与ALL区别为index类型只遍历索引树。
  * range：索引范围扫描，对索引的扫描开始于某一点，返回匹配值域的行，常见于between、<、>等的查询。
  * ref：非唯一性索引扫描，将返回匹配某个单独值的所有行。常见于使用非唯一索引或唯一索引的非唯一前缀进行的查找。
  * eq_ref：唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配。常见于主键或唯一索引扫描。
  * const、system：当MySQL对查询的某部分进行优化，并转换为一个常量时，可使用这些类型进行访问。
  * NULL：MySQL在优化过程中分解语句，执行时甚至不用访问表或索引。

* possible_keys

  possible_keys将指出MySQL能使用哪个索引在表中找到行，查询涉及的字段上若存在索引，则该索引将被列出，但不一定会被查询使用。

* key

  key将显示MySQL在查询中实际使用到的索引，若没有使用索引，则显示为NULL。

* key_len

  key_len表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度。

  注意，key_len显示的值为索引字段的最大可能长度，并非实际使用长度，即key_len是根据表定义计算而得的，而不是通过表内检索得出的。

* ref

  ref表示上述表的连接匹配条件，即哪些列或常量被用于查找索引列上的值。

* rows

  rows表示MySQL根据表统计信息及索引选用的情况，估算地找到所需的记录所需要读取的行数。

* Extra

  Extra包含不适合在其他列中显示但十分重要的额外信息。它可能包含如下4种信息。

  * Using index。该值表示相应的SELECT操作中使用了覆盖索引。包含满足查询需要的所有数据的索引称为覆盖索引。
  * Usingwhere。该值表示MySQL服务器在存储引擎收到记录后进行“后过滤”（Post-filter）。
  * Using temporary。该值表示MySQL需要使用临时表来存储结果集，常见于排序和分组查询。
  * Using filesort。Using filesort即文件排序。MySQL中将无法利用索引完成的排序操作称为“文件排序”。



**MySQL执行计划的局限**

* EXPLAIN不会告诉你关于触发器、存储过程的信息或用户自定义函数对查询的影响情况。
* EXPLAIN不考虑各种Cache。
* EXPLAIN不能显示MySQL在执行查询时所做的优化工作。
* 部分统计信息是估算的，并非精确值。
* MySQL 5.6之前EXPALIN只能解释SELECT操作，其他操作需要重写为SELECT后才能查看执行计划。
* 如果FROM子句里有子查询，那么MySQL可能会执行这个子查询，如果有昂贵的子查询或使用了临时表的视图，那么EXPLAIN其实会有很大的开销。



## 4. 优化索引的方法学

生产环境中数据库出现性能问题，有80%的原因是索引策略导致的，表结构不易变动，而调整索引或SQL往往可以很快就能解决问题。

* 有性能测量

  在应用程序中记录访问数据库的性能日志，这样就可以对整体的访问吞吐有一个很直观很全面的统计，我们应该优化对数据库操作最频繁、最耗资源的那些SQL，但由于性能统计框架的缺位，大部分中小公司更多地依赖于数据库自身的慢查询日志来定位耗时较长的SQL，由慢查询日志入手也是一个很好的出发点，但可能存在一些滞后，不能及时发现性能问题，MySQL的慢查询日志默认记录查询时间超过1s的查询。

* 查看执行计划

  找到消耗资源最多的查询请求后，可以使用EXPLAIN工具查看其执行计划，检查是否走的是合适的索引。

* 优化索引

  我们应该熟悉数据量、数据类型等信息及表之间的关系，按照自己的索引经验，调整或增加索引。

* 测试验证

  如果是线上生产环境，那么请不要在线上环境进行测试验证，除非是非常紧急的情况。应该选择在开发环境中尽量使用和线上环境一样的数据规模，来进行验证测试。

* 上线

  当确认优化达到了预期的效果后，就可以安排上线了。

有一个错误的观念是定期重建索引，如果需要重建索引，首先要证明，重建索引真的能够大大改善性能，否则建议不要做这种费力又不讨好的事情，数据库索引本来就应该是“不好不坏”的状态，不要期望它始终以一种理想的状态在运行。



## 5. ID主键

* 建议主键是整型。
* 如果表中包含一列能够确保唯一、非空（NOTNULL），以及能够用来定位一条记录的字段，就不要因为传统而觉得一定要加上一个自增ID做主键。
* 主键也遵从索引的一些约定，注意联合主键的字段顺序。
* 为主键选择更有意义的名称，如ID这个名称太过笼统，表达的信息可能不准确。



**自增ID主键**

自增列的默认起始值是1，默认可以按步长为1进行递增，自增列的增长将受两个MySQL全局参数的影响。

* auto_increment_offset：确定AUTO_INCREMENT列值的起点。
* auto_increment_increment：控制列值增加的间隔，即步长。

InnoDB选择主键创建簇索引。如果没有主键，就会选取一个唯一非空的索引来替代；如果仍然找不到合适的列，那么将创建一个隐含的主键来创建簇索引。选取一个唯一非空的索引做主键可能不是我们所期待的，一般的解决办法是删除我们不期望的主键（唯一索引），创建一个非空的自增列，再增加这个唯一索引。



**自增ID可以插入指定的值**

自增ID还有一个特性，那就是如果插入0值或NULL值，InnoDB会认为没有设定值，然后帮你自增一个值。



## 6.字符集和国际化支持

字符集（character set）是一套符号和编码。校对规则（collation）是在字符集内用于比较字符的一套规则，即字符集的排序规则。

对于字符集，MySQL能够做如下这些事情。

* 使用多种字符集来存储字符串。
* 使用多种校对规则来比较字符串。
* 在同一台服务器、同一个数据库甚至在同一个表中，使用不同的字符集或校对规则来混合字符串。
* 允许定义任何级别的字符集和校对规则。



### 6.1 字符集设置

字符集设置可以分为两类：

* 一类是创建对象的默认值；

  字符集和校对规则有4个级别的默认设置：服务器级、数据库级、表级和连接级。更低级别的配置会继承更高级别的配置。

* 另一类是控制server端和client端交互通信的配置。

  绝大部分MySQL客户端都不具备同时支持多种字符集的能力，每次都只能使用一种字符集。客户和服务器之间的字符集转换工作是由如下几个MySQL系统变量来控制的。

  * character_set_server：MySQL Server默认字符集。
  * character_set_database：数据库默认字符集。
  * character_set_client：MySQL Server假定客户端发送的查询使用的字符集。
  * character_set_connection：MySQL Server接收客户端发布的查询后，将其转换为character_set_connection变量指定的字符集。
  * character_set_result：MySQL Server把结果集和错误信息转换为character_set_result指定的字符集，并发送给客户端。
