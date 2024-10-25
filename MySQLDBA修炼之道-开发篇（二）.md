
# 四、开发进阶

## 1. 范式和反范式

范式是数据库规范化的一个手段，是数据库设计中的一系列原理和技术，用于减少数据库中的数据冗余，并增进数据的一致性。

**范式**

* 1.1 第一范式
  第一范式是指数据库表的每一列（属性）都是不可分割的基本数据项，这就要求数据库的每一列都只能存放单一值，即实体中的某个属性不能有多个值或不能有重复的属性。



* 1.2 第二范式
  一个数据表符合第二范式的前提是该数据表符合第一范式。它的规则是要求数据表里的所有数据都要和该数据表的主键有完全相依的关系；如果有哪些数据只和主键的一部分有关的话，就得把它们独立出来变成另一个数据表。如果一个数据表的主键只有单一一个字段的话，那么它就一定符合第二范式。



* 1.3 第三范式
  第三范式的所有非键属性都只和候选键有相关性，也就是说所有非键属性互相之间应该是无关的。候选键指的是能够唯一标识一笔记录的属性的最小集合，一般我们所说的候选键指的就是主键。



范式的好处是：使编程相对简单，数据量更小，更适合放入内存，更新更快，只需更新更少的数据。更少的冗余数据意味着更少地需要GROUP、DISTINCT之类的操作。

范式的坏处是：查询会变得更加复杂，查询时需要更多连接（JOIN）操作，一些可以复合索引的列由于范式化的需要被分布到了不同的表中，导致索引策略不佳。



**反范式**

反范式是试图通过增加冗余数据或通过分组数据来优化数据库读取性能的过程。

反范式的好处：减少了连接，因此可以更好地利用索引进行筛选和排序，对于一些查询操作可以提高性能。

反范式的坏处：冗余数据意味着更多的写入，如果冗余的数据量很大，还可能会碰到I/O瓶颈，这会导致性能变得更差，所以需要事先衡量对各个表的更新量和查询量，评估对其他查询的影响，避免引发性能问题。冗余数据也意味着可能要牺牲部分数据的一致性，我们有必要区分不同数据的一致性的优先级，对于重要的、用户比较敏感的数据一定要注意一致性的问题，以免影响用户的体验。

开发人员首先

创建一个完全规范化的设计，然后为了性能原因选择性地对一些表进行反范式化设计。我们要牢记一个准则，设计的数据库应该按照用户可能的访问路径、访问习惯进行设计，而不是严格地按照数据范式来设计。



## 2. 权限机制和安全

权限可以分为两类：

* 系统权限：系统权限允许执行一些特定的功能，如关闭数据库、终止进程、显示数据库列表、查看当前执行的查询等。
* 对象选项：对象权限是指对一些特殊的对象（表、列、视图、数据库）的访问权限，例如是否允许访问某张表，是否允许在某个库中创建表。



一般不允许直接更改MySQL的权限表，而是通过GRANT和REVOKE语句进行权限的赋予和收回，这也是更安全可靠的办法。授予的权限可以分为多个级别：服务器级别（全局）、数据库级别、表级别、列级别、子程序级别。撤销权限即回收已经存在的权限。

GRANT和REVOKE的基本语法

```sql
GRANT [privileges] ON [objects] TO [user]
GRANT [privileges] ON [objects] TO [user] IDENTIFIED BY [password]
REVOKE [privileges] ON [objects] FROM [user]
```

MySQL为有SUPER权限的用户专门保留了一个额外的连接，因此即使是所有的普通连接都被占用，MySQL root用户仍可以登录并检查服务器的活动。

如果想要限制单个账户允许的连接数量，可以通过设置max_user_connections变量来完成。

MySQL允许对不存在的数据库目标授予权限。这个特性是特意设计的，目的是允许数据库管理员为将在此后被创建的数据库目标预留用户账户和权限。



### 2.1 权限更改何时生效

当mysqld启动时，所有授权表的内容将被读进内存并且从此时开始生效。当服务器注意到授权表被改变了时，现存的客户端连接将会受到如下影响。

* 表和列的权限在客户端的下一次请求时生效。
* 数据库的权限改变在下一个USE db_name命令生效。
* 全局权限的改变和密码改变在下一次客户端连接时生效。

如果使用GRANT、REVOKE或SET PASSWORD命令对授权表进行修改，那么服务器会注意到更改并立即将授权表重新载入内存。

如果手动地修改授权表（使用INSERT、UPDATE或DELETE等），则应该执行mysqladmin flush-privileges或mysqladmin reload告诉服务器再重新装载授权表，否则手动的更改将不会生效，除非重启服务器。



### 2.2 常用的权限

* SELECT、INSERT、UPDATE和DELETE权限允许用户在一个数据库现有的表上实施读取、插入、更新和删除记录的操作。这也是一般程序账号所需要的权限。
* SHOW VIEW权限允许用户查看已经创建了的视图。
* ALTER权限允许用户使用ALTER TABLE命令来修改现有数据表的结构。
* CREATE和DROP权限允许用户创建新的数据库和表，或者删除现存的数据库和表。生产环境中一般不赋予程序账号DROP的权限。
* GRANT权限允许用户把自己拥有的权限授予其他的用户。
* FILE权限允许被授予该权限的用户都能读或写MySQL服务器能读写的任何文件。
* SHUTDOWN权限允许用户使用SHUTDOWN命令关掉服务器。可以创建一个用户专门用来关闭服务器。
* PROCESS权限允许用户使用PROCESSLIST命令显示在服务器内执行的进程的信息；使用KILL命令终止服务器进程。用户总是能显示或终止自己的进程，但是，显示或终止其他用户启动的进程则需要PROCESS权限。一些监控工具需要PROCESS权限查看正在执行的命令。



### 2.3 强化安全

强化安全的目的有如下三点：

* 保护好MySQL主机的安全，同时也需要关注其他能访问数据库的主机的安全。

* 确保MySQL自身的安全，包括生产库和备份，应使用强密码，尽可能分配最小的权限给用户。

* 确保网络、物理的安全，同时也需要关注信息内容的保密。



安全的指导原则和注意事项：

* 加强安全意识。比如加密办公电脑、个人笔记本上的重要数据，不要将未加密的数据上传到各种公共云存储中。在不安全的网络环境下，比如一些公共Wi-Fi中，涉及账号的操作可能会泄露你的信息。
* 一般将所有数据库都部署于内网（仅监听内网IP），需要慎重对待跨IDC的数据库同步，MySQL自身并没有很好的方式加密数据传输。
* 开放外网访问的MySQL服务器，需要有相应的访问控制策略，例如通过部署防火墙来限制来源IP。
* 如果条件允许，应该增加网络安全团队进行安全检查和审计。
* 在不安全的网络环境中访问公司或远程维护机器，建议使用VPN。
* 不要让任何人（除了MySQL root账户）访问MySQL数据库中的mysql系统库！
* 用GRANT和REVOKE语句来控制对MySQL的访问。不要授予超过需求的权限。绝对不能为所有主机授权。
* 不要给程序账号授予SUPER权限。
* 生产库上不要留研发人员的账号。
* 隔离生产环境、开发环境和测试环境，不允许研发、测试人员有权限更改生产环境或知道生产环境的账号密码。
* 初始安装后应该移除匿名和空密码账号，可以尝试用“mysql-u root”，如果你能够成功连接服务器而没有要求/输入任何密码，则说明有问题。
* 不要将纯文本密码保存到数据库中，不要从字典中选择密码，如果你的程序是一个客户端，必须用可读的方式存储密码，那么建议使用可解码的加密办法来存储。一些工具，如telnet、ftp，使用的是明文传输密码，建议不要使用，使用ssh、sftp是更安全的方式。
* 使用更安全的算法加密密码，一些流行算法，如MD5已经被证明是弱加密，不适合用于加密密码。曾经比较流行的散列算法SHA-1也被证明不够安全。推荐的方式是在将密码传入散列函数进行加密之前，将其和一个无意义的字符串拼接在一起，这样即使用户选择了一个在字典中存在的单词作为密码，攻击者也很难使用字典攻击的手段破解密码。
* 试试从Internet上使用工具扫描端口，或者使用shell命令shell>telnetserver_host 3306，如果得到连接并得到一些垃圾字符，则端口是打开着的，这种情况应从防火墙或路由器上关闭端口，除非你有足够合理的理由让它开着。
* 避免SQL注入，不要信任应用程序的用户输入的任何数据。
* 有时候人们会认为如果数据库只包含供公共使用的数据，则不需要保护。这是不正确的。即使允许显示数据库中的任何记录，也仍然应该保护和防范、拒绝服务攻击。
* 不要向非管理用户授予FILE权限。拥有FILE权限的任何用户都能在拥有mysqld守护进程权限的文件系统里写入一个文件！
* FILE权限也可以被用来读取任何作为运行服务器的Unix用户可读取或访问的文件。使用该权限，可以将任何文件读入数据库表。这可能会被滥用，例如，通过使用LOADDATA装载“/etc/passwd”进入一个数据库表，然后就能用SELECT显示它。



### 2.4 SQL注入

SQL注入的原理有如下4点：

* 拼接恶意查询。SQL命令可查询、插入、更新、删除数据，以分号字符分隔不同的命令。

  ```sql
  select * from users where user_id = $user_id
  # user_id是传入的参数，如果传入了“1234;deletefromusers”
  select * from users where user_id = 1234; delete from users
  ```

* 利用注释执行非法命令。SQL命令中，可以插入注释。

  ```sql
  select count(*) as 'num' from game_score where game_id=24411 and platform_id=11 and version=$version and session_id = sessid='d7a157-0f-48b6-98-c35592'
  # 如果version包含了恶意的字符串“'-1'OR3 ANDSLEEP(500)--”
  select count(*) as 'num' from game_score where game_id=24411 and platform_id=11 and version='-1' OR 3 AND SLEEP(500)-- 'and session_id = sessid='d7a157-0f-48b6-98-c35592'
  ```

* SQL命令对于传入的字符串参数是用单引号引起来的。如果字符串本身包含单引号而没有被处理，则可能会篡改原本的SQL语法的作用。

  ```sql
  select * from user_name where user_name = $user_name
  # 如果user_name传入的是G'chen
  select * from user_name where user_name ='G'chen'
  ```

* 添加一些额外的条件为真值表达式，改变执行行为。

  ```sql
  update users set userpass=SHA2('$userpass') where user_id=$user_id;
  # 如果user_id被传入恶意的字符串“1234 ORTRUE”
  update users set userpass=SHA2('123456') where user_id=1234 OR TRUE;
  ```

  

下面是避免SQL注入的一些方法。

* 过滤输入内容，校验字符串

  应该在将数据提交到数据库之前，就把用户输入中的不合法字符剔除掉。

* 参数化查询

  参数化查询目前已被视作是最有效的预防SQL注入攻击的方法。不同于在SQL语句中插入动态内容，查询参数的做法是在准备查询语句的时候，就在对应参数的地方使用参数占位符。然后，在执行这个预先准备好的查询时提供一个参数。

  但是绑定参数也有如下一些限制。

  * 不能让占位符“?”代替一组值

    ```sql
    SELECT * FROM departments WHERE userid IN ( ? )
    ```

  * 不能让占位符“?”代替数据表名或列名

  * 不能让占位符“?”代替SQL关键字

* 安全测试、安全审计



## 3. 慢查询日志

慢查询日志可以用来定位执行时间很长的查询。



### 3.1 查看慢查询日志

**优化策略**

性能优化的一个很重要的步骤是识别导致问题的BADSQL。对于一般的数据库调优，调优人员往往会采用调优TOP 10的策略，如果我们把最“昂贵”的10个查询优化完（更高效地运行它们，例如添加一个索引），那么就会立即看到对整体MySQL的性能的提升。然后就可以重复这一过程，并优化新的前10名的查询。



**慢查询日志的格式**

不同数据库TOP 10基于的标准可能不太一样，商业数据库提供了更完善的成本分析方法，MySQL的慢查询日志比较粗略，主要是基于以下3项基本的信息：

* Query_time：查询耗时。
* Rows_examined：检查了多少条记录。
* Rows_sent：返回了多少行记录（结果集）。

其他信息：

* Time：执行SQL的开始时间。
* Lock_time：等待tablelock的时间，注意InnoDB的行锁等待是不会反应在这里的。
* User@Host：执行查询的用户和客户端IP。

可以使用mysqldumpslow命令获得慢查询日志摘要来处理慢查询日志，或者使用更好的第三方工具pt-query-digest。



**如何识别需要关注的SQL**

第一步，确认已经开启了慢查询日志，并记录了合理的阈值。

命令将查看慢查询是否启用

```bash
mysql> show variables like'%query_log%';
--------------
show variables like'%query_log%'
--------------
+---------------------+-----------------------------------------+
| Variable_name | Value |
+---------------------+-----------------------------------------+
| slow_query_log | ON |
| slow_query_log_file |/path/to/log3304/slowquery.log |
+---------------------+-----------------------------------------+
2rows in set (0.00sec)
```

如果配置文件或启动参数没有给出file_name值，慢查询日志将默认命名为“主机名-slow.log”，如果给出了文件名，但不是绝对路径名，文件则写入数据目录。

使用命令“SHOW VARIABLES LIKE'%query_time%'”查看全局变量long_query_time。所有执行时间超过long_query_time秒的SQL语句都会被记录到慢查询日志里。





### 3.2 使用工具分析慢查询日志

如果慢查询日志增长得非常快，很难筛选和查找里面的信息，那么在这种情况下，有如下两种选择：

* 调整阈值，先设置为较大的阈值，这样慢查询记录就很少了，等优化得差不多了，再减少阈值，不断进行优化。
* 使用命令/脚本、工具进行分析，如mysqldumpslow、pt-query-digest等。



**使用操作系统命令分析**

可以使用操作系统自带的命令进行一些简单的统计，如grep、awk、wc，但不容易实现更高级的筛选排序。

下面来看个示例，通过如下命令可以看到每秒的慢查询的统计，当检查到有突变时，往往会有异常发生，这时便可以更进一步到具体的慢查询日志里去查找可能的原因。

```bash
awk '/^# Time:/{print $3, $4, c;c=0}/^# User/{c++}' slowquery.log > /tmp/aaa.log
```



**mysqldumpslow**

mysqldumpslow命令是官方自带的，此命令可获得日志中的查询摘要。

```sql
# 访问时间最长的10个sql语句的命令
mysqldumpslow -t10 /path/to/log3304/slowquery.log
# 访问次数最多的10个sql语句的命令
mysqldumpslow -s c -t10 /path/to/log3304/slowquery.log
# 访问记录集最多的10个sql语句的命令
mysqldumpslow -s r -t10 /path/to/log3304/slowquery.log
```



**pt-query-digest**

pt-query-digest可以生成一份比官方mysqldumpslow可读性好得多的报告。安装也很简单：

```bash
wget www.percona.com/get/pt-query-digest
chmod u+x pt-query-digest
# 基本语法格式如下所示
pt-query-digest [OPTIONS] [FILES] [DSN]

# 直接分析慢查询的命令
pt-query-digest /path/of/slow.log > slow.rtf
# 分析半个小时内的慢查询的命令
pt-query-digest --since 1800s /path/of/slow.log > slow.rtf
# 分析一段时间范围内的慢查询的命令
pt-query-digest --since '2014-04-14 22:00:00' --until '2014-04-14 23:00:00' /path/of/slow.log > slow.rtf
# 显示所有分析的查询命令
pt-query-digest --limit 100% /path/of/slow.log > slow.rtf # 其中，“--limit”参数默认是“95%:20”，表示显示95%的最差的查询，或者20个最差的查询。
```



也可以用这个工具来分析二进志日志，以查看我们日常的修改语句是如何分布的，首先需要把二进志日志转换为文本格式。

```bash
mysqlbinlog mysql-bin.012639 > /tmp/012639.log
pt-query-digest --type binlog /tmp/012639.log
```



**如何查看pt-query-digest报告**

以下是一个输出报告，为了节省篇幅，删除了部分信息。

```
# 140.9s user time, 1.4s system time, 57.93M rss, 154.03M vsz
# Current date: Sun Feb 16 09:16:39 2011
```

上面是执行pt-query-digest工具的时间



```
# Hostname: db1000
# Files: /usr/lcoal/mysql/data/slowquery.log
# Overall: 304.88k total, 159 unique, 0.22 QPS, 0.15x concurrency
```

上面是慢查询次数一共是304.88k，唯一的查询159个。



```
# Time range: 2010-12-01 00:00:01 to 2010-12-17 09:05:17
```

这里记录的是发现第一条慢查询的时间到最后一条慢查询的时间。



```
# Attribute total min max avg 95% stddev median
# ============ ======= ======== ======= ======= ====== ======= =======
# Exec time 216112s 500ms 21s 709ms 1s 968ms 552ms
# Lock time 414s 21us 101ms 1ms 626us 7ms 84us
# Rows sent 169.69M 0 213.73k 583.60 97.36 10.75k 9.83
# Rows examine 60.26G 0 866.23k 207.25k 328.61k 70.68k 201.74k
# Query size 120.31M 35 21.07k 413.76 719.66 148.97 363.48
```

* Exectime：执行时间。
* Lock time：表锁的时间。
* Rows sent：返回的结果集记录数。
* Rowsexamine：实际扫描的记录数。
* Query size：应用和数据库交互的查询文本大小。



```
# Profile
# Rank Query ID Response time Calls R/Call Apdx V/M Item
# ==== ================== ================ ====== ======= ==== ===== =====
# 1 0x5931CCE8168ECE59 92062.4390 42.6% 168672 0.5458 1.00 0.01 SELECT game_info game_stat
# 2 0x0E8691F18411F3DC 23404.4270 10.8% 18602 1.2582 0.60 0.04 SELECT game_info game_stat game_info_2
```

* Rank：所有查询日志分析完毕后，此查询的排序。
* Query ID：查询的标识字符串。
* Responsetime：总的响应时间，以及总占比。一般小于5%可以不用关注。
* Calls：查询被调用执行的次数。
* R/Call：每次执行的平均响应时间。
* Apdx：应用程序的性能指数得分。（Apdex响应的时间越长，得分越低。）
* V/M：响应时间的方差均值比（变异数对平均数比，变异系数）。可说明样本的分散程度，这个值越大，往往是越值得考虑优化的对象。
* Item：查询的简单显示，包括查询的类型和所涉及的表。



以下将按默认的响应时间进行排序，并列出TOP n条查询。并且pt-query-digest输出了EXPLAIN的语句，以方便我们验证查询计划。

```
# Query 1: 0.12 QPS, 0.07x concurrency, ID 0x5931CCE8168ECE59 at byte 243208985
# This item is included in the report because it matches --limit.
# Scores: Apdex = 1.00 [1.0], V/M = 0.01
# Query_time sparkline: | ^__|
# Time range: 2010-12-01 00:00:01 to 2010-12-17 09:04:53
# Attribute pct total min max avg 95% stddev median
# ============ === ======= ======= ======= ======= ======= ======= =======
# Count 55 168672
# Exec time 42 92062s 500ms 11s 546ms 640ms 77ms 501ms
# Lock time 68 283s 58us 101ms 2ms 690us 8ms 80us
# Rows sent 1 2.04M 10 100 12.67 9.83 14.86 9.83
# Rows examine 54 33.12G 204.96k 208.16k 205.90k 201.74k 0.00 201.74k
# Query size 50 60.64M 376 378 376.97 363.48 0 363.48
# String:
# Hosts
# Users sd_game
# Query_time distribution
# 1us
# 10us
# 100us
# 1ms
# 10ms
# 100ms ################################################################
# 1s #
# 10s+ #
# Tables
# SHOW TABLE STATUS LIKE 'game_info'\G
# SHOW CREATE TABLE 'game_info'\G
# SHOW TABLE STATUS LIKE 'game_stat'\G
# SHOW CREATE TABLE 'game_stat'\G
# EXPLAIN /*!50100 PARTITIONS*/
select ...
```



## 4. 数据库设计

数据库设计中DBA关注的两个阶段：逻辑数据库设计阶段和物理数据库设计阶段。



### 4.1 逻辑设计

逻辑数据库的设计大体可以分为以下这些步骤

* 创建并检查ER模型

  此步骤主要是标识实体及实体之间的关系。标识实体的一种方法就是研究用户需求说明里的名词或名词短语。

  标识关系也可以通过研究需求说明书来实现，需求说明书里的动词或动词短语往往表征了某种关系。大多数情况下，关系都是二元的。

* 将ER模型映射为表

  这个步骤的主要目的是为步骤1建立的ER模型产生表的描述。这组表应该代表逻辑数据模型中的实体、关系、属性和约束。产生表的描述后，需要检查表是否满足用户的需求和业务规则。



### 4.2 物理设计

物理数据库设计用于确定逻辑设计如何在目标关系数据库中物理地实现。它描述了基本表、文件组织、用户高效访问数据的索引、相关的完整性约束及安全性限制。

物理设计又可以分为如下几步：

把逻辑设计转换为物理表、分析事务、选择文件组织方式、选择索引（基于最重要的事务），以及适当地进行反范式设计（这么做是为了拥有更好的性能）、列出最终表的详细说明。



* 将逻辑设计转换为物理表
  将逻辑设计转换为物理表即用特定的数据库语言来实现逻辑设计过程中产生的表的描述，可以输出的信息有表汇总。
* 分析事务
  分析事务指的是分析数据库需要满足的用户需求，只有了解了必须要支持的事务的细节，才能做出有意义的物理设计抉择。分析预期的所有事务是极为耗时的，只需研究最重要的那部分事务即可。
  最重要的事务一般是指如下两种事务：
  * 经常运行的事务和对性能产生重大影响的事务。
  * 业务操作的关键事务。

* 选择文件组织方式
  选择文件组织方式是指选择表数据的存放方式。
  物理设计数据库的目标之一就是以有效的方式存储数据。如果目标DBMS允许，则可以为每个表选择一个最佳的文件组织方式。一般有如下两种方法。
  
  * 保持记录的无序性并且创建所需数目的二级索引。
  * 通过指定主键或聚簇索引使表中记录为有序的。这种情况下，应该选择如下的列来排序或聚簇索引记录。
    * 经常用于连接操作的列，因为这样会使连接更有效率。
    * 在表中经常按某列的顺序访问记录的列。
* 选择索引
  设计索引需要平衡性能的提升和维护的成本。以下是创建索引的一些基本指导原则：
  * 不必为小表创建索引。在内存中查询该表会比存储额外的索引结构更加有效。
  * 为检索数据时大量使用的列增加二级索引。
  * 为经常有如下情况的列添加二级索引。
    * 查询或连接条件
    * ORDERBY
    * GROUP BY
    * 其他操作（如UNION或DISTINCT）
  * 考虑是否可以用覆盖索引（covering index）。
  * 如果查询将检索表中的大部分记录（例如25%），即使表很大，也不创建索引。这时候，查询整表可能比用索引查询更有效。
  * 避免为由长字符串组成的列创建索引。
* 反范式设计
  反范式的一些方法包括但不限于如下几点
  * 合并表。
  * 冗余列减少连接。
  * 引入重复组。
  * 创建统计表。
  * 水平/垂直分区。

* 列出最终表的详细说明

  只需要列出重要的表即可。

  以下索引是否建立、数据量及数据增长的情况要根据具体的业务需求来确定。

  * 记录数：记录数，可补充说明未来半年、1年或2年的记录数。
  * 增长量：单位时间的数据增长量。如果量大可以按每天；如果量不大则可以按每月。
  * 表字段的区别度：主要是考虑到将来在此字段上建立索引类型选择时作参考，当字段值唯一时可以不考虑；当字段值不唯一时，估算一个区别度，近似即可。
  * 表的并发：根据具体的业务需求预测表的并发访问，或者说明高峰期的并发程度。



## 5. 导入导出数据

MySQL提供了好几种导出导入数据的方法：LOADDATA、mysqlimport、SELECT…INTO OUTFILE、mysqldump、mysql。其中，mysqldump和mysqlimport是相反的操作，SELECT…INTO OUTFILE和LOADDATAINFILE是相反的操作。



### 5.1 规则简介

* 文本文件里的特殊字符处理

  LOADDATA和SELECT…INTO OUTFILE、mysqlimport和mysqldump有一组专门的用来处理文本文件中特殊字符的选项，具体如下所示。

  * FIELDS TERMINATED BY'fieldtermstring'：各列（字段）之间用什么字符分隔，默认是tab，一般设置为逗号“,”。
  * [OPTIONALLY]ENCLOSED BY'char'：值被什么字符引起来，一般设置为引号'"'，如果指定了OPTIONALLY，则ENCLOSEDBY'char'只对字符串数据类型（比如CHAR、BINARY、TEXT或ENUM）生效。
  * ESCAPED BY'escchar'：定义转义字符，默认是“\”。
  * LINES TERMINATED BY'linetermstring'：定义行结束符，用于分隔行。

* 文本文件的数据格式
  所有命令都要求有关的文本文件必须严格遵守一种数据格式，具体如下所示

  * 数值：可以用科学计数法。
  * 字符串：字符串里的特殊字符必须加上反斜线字符作为识别标志，以区别于各种分隔符。
  * NULL值：假设“\”作为转义前导字符，“'”作为字符串的前后缀标记，那么在导出操作中，NULL值将被表示为\N；在没有指定转义前导字符的导出操作中，NULL值将被表示为由4个字符构成的字符串。在指定了转移前导字符的操作中，MySQL将把NULL、\N、'\N'都解释为NULL值，但'NULL'将被解释为一个字符串'NULL'。



### 5.2 使用mysqldump导出，使用mysql导入

虽然mysqldump速度较慢，但这种方式有最好的兼容性，这也是目前使用最为广泛的备份数据的方式。

**导出数据：**

1. 导出指定的表

   ```
   mysqldump test --tables test1 test4 > test1_test4.sql
   ```

2. 分别导出sql文件和数据文件（数据值以tab分隔）。

   ```
   mysqldump --tab=/home/garychen/tmp test
   ```

3. 分离导出sql文件和数据文件（定制数据格式，数据值以逗号分隔）

   ```
   mysqldump --tab=/home/garychen/tmp --fields-terminated-by=',' --fields-enclosed-by=''' test
   ```

4. 导出某个库

   ```
   mysqldump --complete-insert --force --add-drop-database --insert-ignore --hex-blob --databases test > test_db.sql
   ```

   参数说明：

   --complete-insert：导出的dump文件里，每条INSERT语句都包括了列名。

   --force：即使出现错误（如VIEW引用的表已经不存在了），也要继续执行导出操作（mysqldump会打印出错误，注释完VIEW定义后继续后续的数据导出）。

   --insert-ignore：生成的INSERT语句是INSERT IGNORE的形式，如果导入此文件，即使出错了也仍然可以继续导入数据（当作警告）。

   --databases：类似--tables，后面可以跟多个值。

   --compatible=name：导出的文件和其他数据库更兼容（但不确保），name的值可以是ANSI、MYSQL323、MYSQL40、POSTGRESQL、ORACLE、MSSQL、DB2、MAXDB、NO_KEY_OPTIONS、NO_TABLE_OPTIONS或NO_FIELD_OPTIONS。

5. 导出所有的数据库。

   ```
   mysqldump --all-databases --add-drop-database > db.sql
   ```

6. 导出xml格式的数据

   ```
   mysqldump -u root -p --xml mylibrary > /tmp/mylibrary.xml
   ```

   如果有二进制数据，则要使用选项--hex-blob。

   InnoDB若想获得一致性的数据库副本，则要启用选项--single-transaction。



mysqldump不能利用通配符导出多个表，表比较多的时候，可以先SELECT出要导出的表，如下语句即可查询到所有的表。

```
select group_concat(table_name SEPARATOR ' ') from information_schema.tables where table_schema ='db_name' and table_name
like 'prefix%';
```

或者，可以采用如下方式将表名导出到一个文件

```
mysql -N information_schema -e "select table_name from tables where table_name like 'prefix_%' " > tbs.txt
```

然后运行如下命令导出数据

```
mysqldump db 'cat tbs.txt' > dump.sql
```

也可以忽略部分表，加上参数--ignore-table=db_name.tbl_name1、--ignore-table=db_name.tbl_name2。

mysqldump可以把警告和错误追加记录在文件中，加上参数--log-error=file_name即可。

如果使用mysqldump导出数据，可以考虑的优化的方式有如下5种。

* 选择I/O活动低的时候。
* I/O分离（数据盘和备份盘I/O分离）。
* 输出到管道压缩（gzip）。
* --quick跳过内存缓冲（--opt默认启用）。
* 从数据保留策略上想办法，把不需要修改的大量数据放到历史表中，而不是每次都备份。



**导入数据：**

mysqldump导出的SQL转储文件，可以用如下的形式将数据导入到数据库中。

```
mysql db_name < db_name.sql
```

转储文件（dump文件）里面一般指定了set names utf8，所以我们在导入的时候不再需要指定特殊的字符集。有一些特殊的场合，SQL文件是以其他的字符集导出的，这个时候导入要注意保持文件的字符集、客户端字符集和连接的字符集的一致性。

```
mysql --default-character-set=charset_name database_name < import_table.sql
```

--default-character-set的意思是，客户端和连接都默认使用charset_name字符集。



### 5.3 使用SELECT INTO OUTFILE命令导出数据

如果想要进行SQL级别的表备份，可以使用SELECT INTO OUTFILE命令语句。对于SELECT INTO OUTFILE，输出的文件不能先于输出存在。

```
SELECT * INTO OUTFILE '/tmp/testfile.txt' FROM exporttable;
SELECT * INTO OUTFILE '/tmp/testfile.txt' FIELDS TERMINATED BY ':' OPTIONALLY ENCLOSED BY '+' ESCAPED BY '!' FROM
exporttable;
SELECT a,b,a+b INTO OUTFILE '/tmp/result.text' FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"' LINES TERMINATED BY '\n'
FROM test_table;
```



### 5.4 使用LOAD DATA导入数据

LOAD DATA INFILE是读取文件导入表中。

如果MySQL服务器和LOADDATA命令不在同一台计算机上执行，当想导入本地文件系统的文件时，则需要使用语法变体LOAD DATA...LOCAL INFILE...，也就是说，如果指定LOCAL关键词，则表明从客户主机读文件。如果没指定LOCAL，那么文件必须位于服务器上。

如果LOADDATAINFILE不支持指定字符集，那么在导入前需要确认当前数据库的字符集，如果与当前数据库的字符集不符，则使用SET character_set_database命令进行更改。

**LOAD DATA的优化**

对于InnoDB的优化，建议的方式如下。

* 将innodb_buffer_pool_size设置得更大些。
* 将innodb_log_file_size设置得更大些，如256MB。
* 设置忽略二级索引的唯一性约束，SETUNIQUE_CHECKS=0。
* 设置忽略外键约束，SET FOREIGN_KEY_CHECKS=0。
* 设置不记录二进制日志，SET sql_log_bin=0。
* 按主键顺序导入数据。由于InnoDB使用了聚集索引，如果是顺序自增ID的导入，那么导入将会更快，我们可以把要导入的文件按照主键顺序先排好序再导入。
* 对于InnoDB引擎的表，可以在导入前，先设置autocommit=0
* 可以将大的数据文件切割为更小的多个文件，例如使用操作系统命令split切割文件，然后再并行导入数据。



对于MyISAM的优化，建议的方式如下

* 将bulk_insert_tree_size、myisam_sort_buffer_size、key_buffer_size设置得更大些。
* 先禁用key（ALTERTABLE…DISABLEKEYS），然后再导入数据，然后启用key（ALTERTABLE…ENABLEKEYS）。重新启用key后，可以批量重新创建索引，批量创建索引的效率比在逐笔插入记录时创建索引要高效得多。注意ALTERTABLE…DISABLEKEYS禁用的只是非唯一索引，唯一索引或主键是不能禁用的，除非你先手动移除它。
* 使用LOAD DATA INFILE，tab分隔的文件更容易解析，比其他方式更快。



### 5.5 用mysqlimport工具导入

mysqlimport命令的语法格式如下

```
mysqlimport databasename tablename.txt
```



### 5.6 用split切割文件，加速导入数据

split命令的作用是切割文件，语法格式如下所示。

```
split [OPTION] [INPUT [PREFIX]]
```

如果不加入任何参数，默认情况下是以1000行的大小来分割的。



## 6. 事务和锁

MySQL Server级别的锁大致有两种：

* Table locks（表锁）
* Global locks（全局锁）



### 6.1 MyISAM的表锁

表锁定的原理：

* 对于WRITE，MySQL使用的表锁定方法原理如下。

  * 如果在表上没有锁，则在它上面放一个写锁。

  * 否则，把锁定请求放在写锁定队列中。

* 对于READ，MySQL使用的锁定方法原理如下。

  * 如果在表上没有写锁定，则把一个读锁定放在它上面。

  * 否则，把锁定请求放在读锁定队列中

当一个锁定被释放时，锁定可先被写锁定队列中的线程得到，然后是读锁定队列中的线程。

以上的机制，在很多基于MyISAM引擎表的程序中可能会导致严重的性能问题。建议的解决方案是设置变量-low-priority-updates=1，即可以在系统级别进行设置，以避免SELECT查询线程大量累计。



### 6.2 事务定义和隔离级别

事务是数据库管理系统执行过程中的一个逻辑单元，由有限的操作序列构成。

**事务的ACID特性**

* 原子性（Atomic）

  事务作为一个整体被执行，包含在事务中的对数据库的操作要么全部被执行，要么全部都不执行。MyISAM引擎的表，它不支持事务，那么在出

  错之前的值是可以被正常插入到表中的。

* 一致性（Consistency）

  事务应确保数据库的状态从一个一致状态转变为另一个一致状态。一致状态的含义是数据库中的数据应满足约束。

* 隔离性（Isolation）

  多个事务并发执行时，一个事务的执行不应影响其他事务的执行。

* 持久性（Durability）

  已被提交的事务对数据库的修改应该被永久保存在数据库中。



**事务的隔离级别**

事务隔离级别越高，越能保证数据的完整性和一致性，但是对并发性能的影响也会越大。MySQL事务包含如下4个隔离级别，按隔离级别从低到高排列如下。

* read uncommitted（dirty read）

  read uncommitted也称为读未提交，事务可以看到其他事务更改了但还没有提交的数据，即存在脏读的情况。

* read committed

  read committed也称为读提交，事务可以看到在它执行的时候，其他事务已经提交的数据，已被大部分数据库系统采用。允许不可重复读，但不允许脏读。

* repeatable read

  repeatableread也称为可重复读。同一个事务内，同一个查询请求，若多次执行，则获得的记录集是相同的，但不能杜绝幻读。

* serializable

  serializable也称为序列化，最高级别的锁，它解决了幻读，它将锁施加在所有访问的数据上。

  

### 6.3 InnoDB的行锁

行级锁定的优点：

* 当在很多线程中访问不同的行时只存在少量锁定冲突。
* 回滚时只有少量的更改。
* 可以长时间锁定单一的行



**几种行锁技术**

* 记录锁（index-rowlocking）：这是一个索引记录锁。
* 间隙锁：这是施加于索引记录间隙上的锁。
* next-key锁：记录锁加间隙锁的组合。



**等待行锁超时**

有时我们在慢查询日志中会看到一些很耗时的查询，但单独执行却很快，此时有可能就是因为该查询因等待InnoDB行锁而超时。



**MVCC简要介绍**

单纯靠行级别的锁，是不可能实现好的并发性的，MySQL InnoDB还需要配合MVCC（MultiversionConcurrencyControl）技术来提供高并发访问。在很多情况下MVCC可以不需要使用锁，即可实现更新数据时的无阻塞读。

在常用的事务隔离级别read committed和repeatableread级，都应用了MVCC技术。MVCC会保存某个时间点上的数据快照。这就意味着事务可以看到一个一致的数据视图，不管它们还需要运行多久。



## 7. 死锁

死锁是指两个或两个以上的事务在执行过程中，因争夺资源而造成的一种互相等待的现象，若无外力作用，它们都将无法进行下去。

产生死锁有4个必要条件。

* 禁止抢占（no preemption）
* 持有和等待（hold and wait）
* 互斥（mutualexclusion）
* 循环等待（circular waiting）



MySQL死锁的解决方法

经常提交你的事务。小事务更少地倾向于冲突。

* 以固定的顺序访问你的表和行。这样事务就会形成定义良好的查询并且没有死锁。
* 将精心选定的索引添加到你的表中。这样你的查询就只需要扫描更少的索引记录，并且因此也可以设置更少的锁定。
* 不要把无关的操作放到事务里面。
* 在并发比较高的系统中，不要显式加锁，特别是在事务里显式加锁。
* 尽量按照主键/索引去查找记录，范围查找增加了锁冲突的可能性，也不要利用数据库做一些额外的计算工作。
* 优化SQL和表设计，减少同时占用太多资源的情况。



## 8. 其他特性

### 8.1 临时表

临时表指的是CREATE TEMPORARYTABLE命令创建的临时的表，临时表只对当前连接可见，对其他连接不可见，结束连接或中断，数据表（数据）将丢失。常使用临时表来存储一些中间结果集，如果需要执行一个很耗资源的查询或需要多次操作大表，那么把中间结果或小的子集放到一个临时表里，可能会有助于加速查询。



### 8.2 分区表

分区表技术允许按照设置的规则，跨文件系统分配单个表的多个部分。实际上，表的不同部分在不同的位置被存储为单独的表。用户所选择的、实现数据分割的规则被称为分区函数，在MySQL中它可以是模数，或者是简单地匹配一个连续的数值区间或数值列表，或者是一个内部HASH函数，或者是一个线性HASH函数。

分区类型，RANGE分区、LIST分区、HASH分区、KEY分区和子分区。常用的存储引擎，如InnoDB、MyISAM、MEMORY都支持分区表。

* RANGE分区的表是通过如下这种方式进行分区的，基于一个连续区间的列值，把多行分配给分区，例如某个时间段的值属于某个分区，某个数值范围的值应该属于某个分区。
* LIST分区在很多方面都类似于RANGE分区。和按照RANGE进行分区的方式一样，每个分区都必须明确定义。它们的主要区别在于，LIST分区中每个分区的定义和选择是基于值列表的，而RANGE分区是从属于一个连续区间值的集合的。
* HASH分区是基于用户定义的表达式的返回值选择分区。它主要用来确保数据在预先确定了数目的分区中是平均分布的。
* 按照KEY进行分区类似于按照HASH进行分区，除了HASH分区使用的是用户自定义的表达式，而KEY分区的散列函数是由MySQL服务器提供的。
* 子分区是分区表中每个分区的再次分割。



由于分区表的不成熟，可能会给整个系统带来隐患。这里有一些通用的建议。

* 只有大表才可能需要分区，几百万笔记录的表并不算大，对于一些高配置的数据库主机，几千万甚至上亿条数据的表也不算大。
* 分区数不能过多，很难想象大于500的分区数。
* 查询的时候，不要跨越多个分区，建议最多跨越1~2个分区。
* 索引的列应该是分区的列，或者有其他条件限制的分区，否则访问所有分区上面的索引进行查找，开销会比较大。

