---
title: 关于MySQL的优化思路
date: 2021-06-11 15:39:04
tags: mysql
categories: SQL
---

<!--more-->

## SQL 优化步骤

当面对一个需要优化的 SQL 时，我们有哪几种排查思路呢？

### 通过 show status 命令了解 SQL 执行次数

首先，我们可以使用 **show status** 命令查看服务器状态信息。show status 命令会显示每个服务器变量 variable\_name 和 value，状态变量是只读的。如果使用 SQL 命令，可以使用 like 或者 where 条件来限制结果。like 可以对变量名做标准模式匹配。

![图片](https://img-blog.csdnimg.cn/img_convert/df0228046f8cda9907e2e85259e81bd1.png)

图我没有截全，下面还有很多变量，读者可以自己尝试一下。也可以在操作系统上使用 **mysqladmin extended-status** 命令来获取这些消息。

但是我执行 mysqladmin extended-status 后，出现这个错误。

![图片](https://img-blog.csdnimg.cn/img_convert/6e67bbb69e51e9eec7381637972c6ca7.png)

应该是我没有输入密码的原因，使用 **mysqladmin \-P3306 \-uroot \-p \-h127.0.0.1 \-r \-i 1 extended-status** 后，问题解决。

这里需要注意一下 show status 命令中可以添加统计结果的级别，这个级别有两个

- session 级：默认当前链接的统计结果

- global 级：自数据库上次启动到现在的统计结果

如果不指定统计结果级别的话，默认使用 session 级别。

对于 show status 查询出来的统计结果，有两类参数需要注意下，一类是以 `Com_` 为开头的参数，一类是以 `Innodb_` 为开头的参数。

下面是 Com\_ 为开头的参数，参数很多，我同样没有截全。

![图片](https://img-blog.csdnimg.cn/img_convert/fa90d54962f22320be3d3b99bb3b271e.png)

Com\_xxx 表示的是每个 xxx 语句执行的次数，我们通常关心的是 select 、insert 、update、delete 语句的执行次数，即

- Com\_select：执行 select 操作的次数，一次查询会使结果 + 1。

- Com\_insert：执行 INSERT 操作的次数，对于批量插入的 INSERT 操作，只累加一次。

- Com\_update：执行 UPDATE 操作的次数。

- Com\_delete：执行 DELETE 操作的次数。

以 Innodb\_ 为开头的参数主要有

- Innodb\_rows\_read：执行 select 查询返回的行数。

- Innodb\_rows\_inserted：执行 INSERT 操作插入的行数。

- Innodb\_rows\_updated：执行 UPDATE 操作更新的行数。

- Innodb\_rows\_deleted：执行 DELETE 操作删除的行数。

通过上面这些参数执行结果的统计，我们能够大致了解到当前数据库是以更新（包括插入、删除）为主还是查询为主。

除此之外，还有一些其他参数用于了解数据库的基本情况。

- Connections：查询 MySQL 数据库的连接次数，这个次数是不管连接是否成功都算上。

- Uptime：服务器的工作时间。

- Slow\_queries：满查询次数。

- Threads\_connected：查看当前打开的连接的数量。

下面这个博客汇总了几乎所有 show status 的参数，可以当作参考手册。

https://blog.csdn.net/ayay\_870621/article/details/88633092

### 定位执行效率较低的 SQL

定位执行效率比较慢的 SQL 语句，一般有两种方式

- 可以通过**慢查询日志**来定位哪些执行效率较低的 SQL 语句。

MySQL 中提供了一个慢查询的日志记录功能，可以把查询 SQL 语句时间大于多少秒的语句写入慢查询日志，日常维护中可以通过慢查询日志的记录信息快速准确地判断问题所在。用 \--log-slow-queries 选项启动时，mysqld 会写一个包含所有执行时间超过 long\_query\_time 秒的 SQL 语句的日志文件，通过查看这个日志文件定位效率较低的 SQL 。

比如我们可以在 my.cnf 中添加如下代码，然后退出重启 MySQL。

```
log-slow-queries = /tmp/mysql-slow.log
long_query_time = 2
```

通常我们设置最长的查询时间是 2 秒，表示查询时间超过 2 秒就记录了，通常情况下 2 秒就够了，然而对于很多 WEB 应用来说，2 秒时间还是比较长的。

也可以通过命令来开启：

我们先查询 MySQL 慢查询日志是否开启

```
show variables like "%slow%";
```

![图片](https://img-blog.csdnimg.cn/img_convert/f12c417c58bed5de2c46147200cc2906.png)

启用慢查询日志

```
set global slow_query_log='ON';
```

![图片](https://img-blog.csdnimg.cn/img_convert/57df048943a64aeb47a71c8fb0fc6877.png)

然后再次查询慢查询是否开启

![图片](https://img-blog.csdnimg.cn/img_convert/02b2cbe1c1d5839fc921e1be79e48af0.png)

如图所示，我们已经开启了慢查询日志。

慢查询日志会在查询结束以后才记录，所以在应用反应执行效率出现问题的时候慢查询日志并不能定位问题，此时应该使用 **show processlist** 命令查看当前 MySQL 正在进行的线程。包括线程的状态、是否锁表等，可以实时的查看 SQL 执行情况。同样，使用**mysqladmin processlist**语句也能得到此信息。

![图片](https://img-blog.csdnimg.cn/img_convert/8282291cafbc0201bb433c0cae1eb9df.png)

下面就来解释一下各个字段对应的概念

- Id ：Id 就是一个标示，在我们使用 kill 命令杀死进程的时候很有用，比如 kill 进程号。

- User：显示当前的用户，如果不是 root，这个命令就只显示你权限范围内的 SQL 语句。

- Host：显示 IP ，用于追踪问题

- Db：显示这个进程目前连接的是哪个数据库，为 null 是还没有 select 数据库。

- Command：显示当前连接锁执行的命令，一般有三种：查询 query，休眠 sleep，连接 connect。

- Time：这个状态持续的时间，单位是秒

- State：显示当前 SQL 语句的状态，非常重要，下面会具体解释。

- Info：显示这个 SQL 语句。

State 列非常重要，关于这个列的内容比较多，读者可以参考一下这篇文章

https://blog.csdn.net/weixin\_34357436/article/details/91768402

这里面涉及线程的状态、是否锁表等选项，可以实时的查看 SQL 的执行情况，同时对一些锁表进行优化。

### 通过 EXPLAIN 命令分析 SQL 的执行计划

通过以上步骤查询到效率低的 SQL 语句后，可以通过 EXPLAIN 或者 DESC 命令获取 MySQL 如何执行 SELECT 语句的信息，包括在 SELECT 语句执行过程中表如何连接和连接的顺序。

比如我们使用下面这条 SQL 语句来分析一下执行计划

```
explain select * from test1;
```

![图片](https://img-blog.csdnimg.cn/img_convert/230949e49f2eb9f0d75b11f6e3722df8.png)

上表中涉及内容如下

- select\_type：表示常见的 SELECT 类型，常见的有 SIMPLE，SIMPLE 表示的是简单的 SQL 语句，不包括 UNION 或者子查询操作，比如下面这段就是 SIMPLE 类型。

![图片](https://img-blog.csdnimg.cn/img_convert/9c620f1fa2648014b79e1eb39d81da85.png)

PRIMARY ，查询中最外层的 SELECT（如两表做 UNION 或者存在子查询的外层的表操作为 PRIMARY，内层的操作为 UNION），比如下面这段子查询。

![图片](https://img-blog.csdnimg.cn/img_convert/dcf7088fa50d696761693cf128fecc2c.png)

UNION，在 UNION 操作中，查询中处于内层的 SELECT（内层的 SELECT 语句与外层的 SELECT 语句没有依赖关系时）。

SUBQUERY：子查询中首个SELECT（如果有多个子查询存在），如我们上面的查询语句，子查询第一个是 sr（sys\_role）表，所以它的 select\_type 是 SUBQUERY。

- table ，这个选项表示输出结果集的表。

- type，这个选项表示表的连接类型，这个选项很有深入研究的价值，因为很多 SQL 的调优都是围绕 type 来讲的，但是这篇文章我们主要围绕优化方式来展开的，type 这个字段我们暂时作为了解，这篇文章不过多深入。

  type 这个字段会牵扯到连接的性能，它的不同类型的性能由好到差分别是

  system ：表中仅有一条数据时，该表的查询就像查询常量表一样。

  const ：当表中只有一条记录匹配时，比如使用了表主键（primary key）或者表唯一索引（unique index）进行查询。

  eq-ref ：表示多表连接时使用表主键或者表唯一索引，比如

  ```
  select A.text, B.text where A.ID = B.ID
  ```

  这个查询语句，对于 A 表中的每一个 ID 行，B 表中都只能有唯一的 B.Id 来进行匹配时。

  ref ：这个类型不如上面的 eq-ref 快，因为它表示的是因为对于表 A 中扫描的每一行，表 C 中有几个可能的行，C.ID 不是唯一的。

  ref\_or\_null ：与 ref 类似，只不过这个选项包含对 NULL 的查询。

  index\_merge ：查询语句使用了两个以上的索引，比如经常在有 and 和 or 关键字出现的场景，但是在由于**读取索引过多**导致其性能有可能还不如 range（后面说）。

  unique\_subquery ：这个选项经常用在 in 关键字后面，子查询带有 where 关键字的子查询中，用 sql 来表示就是这样

  ```
  value IN (SELECT primary_key FROM single_table WHERE some_expr)
  ```

  range ：**索引范围查询**，常见于使用 =，\<>，>，>=，\<，\<=，IS NULL，\<=>，BETWEEN，IN\(\) 或者 like 等运算符的查询中。

  index ：索引全表扫描，把索引从头到尾扫一遍。

  all ：这个我们接触的最多了，就是全表查询，select \* from xxx ，性能最差。

上面就是 type 内容的大致解释，关于 type 我们经常会在 SQL 调优的环节使用 explain 分析其类型，然后改进查询方式，越靠近 system 其查询效率越高，越靠近 all 其查询效率越低。

![图片](https://img-blog.csdnimg.cn/img_convert/af5bfcb5d65e0f0c7dec14a89a23d81e.png)

- possible\_keys ：表示查询时，可能使用的索引。

- key ：表示实际使用的索引。

- key\_len ：索引字段的长度。

- rows ：扫描行的数量。

- filtered ：通过查询条件查询出来的 SQL 数量占用总行数的比例。

- extra ：执行情况的描述。

通过上面的分析，我们可以大致确定 SQL 效率低的原因，一种非常有效的提升 SQL 查询效率的方式就是使用索引，接下来我会讲解一下如何使用索引提高查询效率。

## 索引

索引是数据库优化中最常用也是最重要的手段，通过使用不同的索引可以解决大多数 SQL 性能问题，也是面试经常会问到的优化方式，围绕着索引，面试官能让你造出火箭来，所以总结一点就是索引非常非常重！要！不只是使用，你还要懂其原！理！

### 索引介绍

索引的目的就是用于快速查找某一列的数据，对相关数据列使用索引能够大大提高查询操作的性能。不使用索引，MySQL 必须从第一条记录开始读完整个表，直到找出相关的行，表越大查询数据所花费的时间就越多。如果表中查询的列有索引，MySQL 能够快速到达一个位置去搜索数据文件，而不必查看所有数据，那么将会节省很大一部分时间。

### 索引分类

先来了解一下索引都有哪些分类。

- `全局索引(FULLTEXT)`：全局索引，目前只有 MyISAM 引擎支持全局索引，它的出现是为了解决针对文本的模糊查询效率较低的问题，并且只限于 CHAR、VARCHAR 和 TEXT 列。

- `哈希索引(HASH)`：哈希索引是 MySQL 中用到的唯一 key-value 键值对的数据结构，很适合作为索引。HASH 索引具有一次定位的好处，不需要像树那样逐个节点查找，但是这种查找适合应用于查找单个键的情况，对于范围查找，HASH 索引的性能就会很低。默认情况下，MEMORY 存储引擎使用 HASH 索引，但也支持 BTREE 索引。

- `B-Tree 索引`：B 就是 Balance 的意思，BTree 是一种平衡树，它有很多变种，最常见的就是 B+ Tree，它被 MySQL 广泛使用。

- `R-Tree 索引`：R-Tree 在 MySQL 很少使用，仅支持 geometry 数据类型，支持该类型的存储引擎只有MyISAM、BDb、InnoDb、NDb、Archive几种，相对于 B-Tree 来说，R-Tree 的优势在于范围查找。

从逻辑上来对 MySQL 进行分类，主要分为下面这几种

- 普通索引：普通索引是最基础的索引类型，它没有任何限制 。创建方式如下

  ```
  create index normal_index on cxuan003(id);
  ```

 

  ![图片](https://img-blog.csdnimg.cn/img_convert/ff83638300efddbf54bba53bc5591f72.png)

 

  删除方式

  ```
  drop index normal_index on cxuan003;
  ```

 

  ![图片](https://img-blog.csdnimg.cn/img_convert/3d1e28a09fda0bd1320bcf7a7d8e535f.png)

- 唯一索引：唯一索引列的值必须唯一，允许有空值，如果是组合索引，则列值的组合必须唯一，创建方式如下

  ```
  create unique index normal_index on cxuan003(id);
  ```

 

  ![图片](https://img-blog.csdnimg.cn/img_convert/3197d902b85441d79053985591791795.png)

- 主键索引：是一种特殊的索引，一个表只能有一个主键，不允许有空值。一般是在建表的时候同时创建主键索引。

  ```
  CREATE TABLE table (
       id int(11) NOT NULL AUTO_INCREMENT ,
       title char(255) NOT NULL ,
       PRIMARY KEY (id)
  )
  ```

 

  ![图片](https://img-blog.csdnimg.cn/img_convert/3065fc966d06bd2c9fd82265328109fd.png)

- 组合索引：指多个字段上创建的索引，只有在查询条件中使用了创建索引时的第一个字段，索引才会被使用。使用组合索引时遵循最左前缀原则，下面我们就会创建组合索引。

- 全文索引：主要用来查找文本中的关键字，而不是直接与索引中的值相比较，目前只有 char、varchar，text 列上可以创建全文索引，创建表的适合添加全文索引

  ```
  CREATE TABLE table (
  id int(11) NOT NULL AUTO_INCREMENT ,
  title char(255) CHARACTER NOT NULL ,
  content text CHARACTER NULL ,
  time int(10) NULL DEFAULT NULL ,
  PRIMARY KEY (id),
  FULLTEXT (content)
  );
  ```

  当然也可以直接创建全局索引

  ```
  CREATE FULLTEXT INDEX index_content ON article(content)
  ```

### 索引使用

索引可以在创建表的时候进行创建，也可以单独创建，下面我们采用单独创建的方式，我们在 cxuan004 上创建前缀索引

![图片](https://img-blog.csdnimg.cn/img_convert/86c710e974bff5efb1e3372a8c506064.png)

我们使用 `explain` 进行分析，可以看到 cxuan004 使用索引的情况

![图片](https://img-blog.csdnimg.cn/img_convert/92302060148d81ac76058bb9ed59c8fe.png)

如果不想使用索引，可以删除索引，索引的删除语法是

![图片](https://img-blog.csdnimg.cn/img_convert/b1a310a78ad59670f45c6a5ceac8bc50.png)

索引使用细则

我们在 cxuan005 上根据 id 和 hash 创建一个复合索引，如下所示

```
create index id_hash_index on cxuan005(id,hash);
```

![图片](https://img-blog.csdnimg.cn/img_convert/ed7f8f7a8ce5c011f8ba3b7caad5bba9.png)

然后根据 id 进行执行计划的分析

```
explain select * from cxuan005 where id = '333';
```

![图片](https://img-blog.csdnimg.cn/img_convert/c7bb3dc5a8850093384c494bf6bb0b2f.png)

可以发现，即使 where 条件中使用的不是复合索引（Id 、hash），索引仍然能够使用，这就是索引的前缀特性。但是如果只按照 hash 进行查询的话，索引就不会用到。

```
explain select * from cxuan005 where hash='8fd1f12575f6b39ee7c6d704eb54b353';
```

![图片](https://img-blog.csdnimg.cn/img_convert/2504cc78a4038c352b893c8605091846.png)

如果 where 条件使用了 like 查询，并且 `%` 不在第一个字符，索引才可能被使用。

对于复合索引来说，只能使用 id 进行 like 查询，因为 hash 列不管怎么查询都不会走索引。

```
explain select * from cxuan005 where id like '%1';
```

![图片](https://img-blog.csdnimg.cn/img_convert/d07b9c45713b4066fd774cf73e6e7102.png)

可以看到，如果第一个字符是 \% ，则没有使用索引。

```
explain select * from cxuan005 where id like '1%';
```

![图片](https://img-blog.csdnimg.cn/img_convert/a8a2f477fd83cba63b1035686183b0d1.png)

如果使用了 \% 号，就会触发索引。

如果列名是索引的话，那么对列名进行 NULL 查询，将会触发索引。

```
explain select * from cxuan005 where id is null;
```

![](https://img-blog.csdnimg.cn/20210611153727568.gif)

还有一些情况是存在索引但是 MySQL 并不会使用的情况。

- 最简单的，如果使用索引后比不使用索引的效率还差，那么 MySQL 就不会使用索引。

- 如果 SQL 中使用了 OR 条件，OR 前的条件列有索引，而后面的列没有索引的话，那么涉及到的索引都不会使用，比如 cxuan005 表中，只有 id 和 hash 字段有索引，而 info 字段没有索引，那么我们使用 or 进行查询。

  ```
  explain select * from cxuan005 where id = 111 and info = 'cxuan';
  ```

![](https://img-blog.csdnimg.cn/20210611153727569.gif)

- 我们从 explain 的执行结果可以看到，虽然 possible\_keys 选项上仍然有 id\_hash\_index 索引，但是从 key、key\_len 可以得知，这条 SQL 语句并未使用索引。

- 在带有复合索引的列上查询不是第一列的数据，也不会使用索引。

  ```
  explain select * from cxuan005 where hash = '8fd1f12575f6b39ee7c6d704eb54b353';
  ```

  ![图片](https://img-blog.csdnimg.cn/img_convert/03a58d6c271ce79e80e7ad48db23fa34.png)

- 如果 where 条件的列参与了计算，那么也不会使用索引

  ```
  explain select * from cxuan005 where id + '111' = '666';
  ```

  ![图片](https://img-blog.csdnimg.cn/img_convert/2721d3543748ece8a0a046effd71063b.png)

- 索引列使用函数，一样也不会使用索引

  ```
  explain select * from cxuan005 where concat(id,'111') = '666';
  ```

  ![图片](https://img-blog.csdnimg.cn/img_convert/ec5086d89874bafb56117264df1dbe62.png)

- 索引列使用了 like ，并且 `%` 位于第一个字符，则不会使用索引。

- 在 order by 操作中，排序的列同时也在 where 语句中，将不会使用索引。

- 当数据类型出现隐式转换时，比如 varchar 不加单引号可能转换为 int 类型时，会使索引无效，触发全表扫描。比如下面这两个例子能够显而易见的说明这一点

  ![图片](https://img-blog.csdnimg.cn/img_convert/9d02aed0b543beca0f2e84ac7bc3c5be.png)

- 在索引列上使用 IS NOT NULL 操作

  ![图片](https://img-blog.csdnimg.cn/img_convert/17062c9db14db1bd86a4f67ff492b256.png)

- 在索引字段上使用 \<>，\!=。不等于操作符是永远不会用到索引的，因此对它的处理只会产生全表扫描。

  ![图片](https://img-blog.csdnimg.cn/img_convert/2685df910c22153926df2daaa0a61bf9.png)

关于设置索引但是索引没有生效的场景还有很多，这个需要小伙伴们工作中不断总结和完善，不过我上面总结的这些索引失效的情景，能够覆盖大多数索引失效的场景了。

### 查看索引的使用情况

在 MySQL 索引的使用过程中，有一个 `Handler_read_key` 值，这个值表示了**某一行被索引值读的次数**。Handler\_read\_key 的值比较低的话，则表明增加索引得到的性能改善不是很理想，可能索引使用的频率不高。

还有一个值是 `Handler_read_rnd_next`，这个值高则意味着查询运行效率不高，应该建立索引来进行抢救。这个值的含义是在数据文件中读下一行的请求数。如果正在进行大量的表扫描，Handler\_read\_rnd\_next 的值比较高，就说明表索引不正确或写入的查询没有利用索引。

![图片](https://img-blog.csdnimg.cn/img_convert/e304165a9e6cc79681c6fa5b090f58eb.png)

## MySQL 分析表、检查表和优化表

对于大多数开发者来说，他们更倾向于解决简单 SQL的优化，而复杂 SQL 的优化交给了公司的 DBA 来做。

下面就从普通程序员的角度和你聊几个简单的优化方式。

### MySQL 分析表

分析表用于分析和存储表的关键字分布，分析的结果可以使得系统得到准确的统计信息，使得 SQL 生成正确的执行计划。如果用于感觉实际执行计划与预期不符，可以执行分析表来解决问题，分析表语法如下

```
analyze table cxuan005;
```

![图片](https://img-blog.csdnimg.cn/img_convert/c49e8e83471a76c90e8d61c481d2bcf3.png)

分析结果涉及到的字段属性如下

Table：表示表的名称；

Op：表示执行的操作，analyze 表示进行分析操作，check 表示进行检查查找，optimize 表示进行优化操作；

Msg\_type：表示信息类型，其显示的值通常是状态、警告、错误和信息这四者之一；

Msg\_text：显示信息。

对表的定期分析可以改善性能，应该成为日常工作的一部分。因为通过更新表的索引信息对表进行分析，可改善数据库性能。

### MySQL 检查表

数据库经常可能遇到错误，比如数据写入磁盘时发生错误，或是索引没有同步更新，或是数据库未关闭 MySQL 就停止了。遇到这些情况，数据就可能发生错误： **Incorrect key file for table: ' '. Try to repair it**. 此时，我们可以使用 Check Table 语句来检查表及其对应的索引。

```
check table cxuan005;
```

![图片](https://img-blog.csdnimg.cn/img_convert/0b2e1f7bc2c7acfbc25401d8c681cb93.png)

检查表的主要目的就是检查一个或者多个表是否有错误。Check Table 对 MyISAM 和 InnoDB 表有作用。Check Table 也可以检查视图的错误。

### MySQL 优化表

MySQL 优化表适用于删除了大量的表数据，或者对包含 VARCHAR、BLOB 或则 TEXT 命令进行大量修改的情况。MySQL 优化表可以将大量的空间碎片进行合并，消除由于删除或者更新造成的空间浪费情况。它的命令如下

```
optimize table cxuan005;
```

![图片](https://img-blog.csdnimg.cn/img_convert/ceb7a348cb85e7d2315f62759a6052c7.png)

我的存储引擎是 InnoDB 引擎，但是从图可以知道，InnoDB 不支持使用 optimize 优化，建议使用 recreate + analyze 进行优化。optimize 命令只对 MyISAM 、BDB 表起作用。