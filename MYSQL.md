

# 数据库基础

## 1.什么是事务

事务是应用程序中一系列严密的操作，所有操作必须成功完成，否则在每个操作中所作的所有更改都会被撤消。也就是事务具有原子性，一个事务中的一系列的操作要么全部成功，要么一个都不做。

事务的结束有两种，当事务中的所以步骤全部成功执行时，事务提交。如果其中一个步骤失败，将发生回滚操作，撤消撤消之前到事务开始时的所以操作。

## 2.事务的 ACID

事务具有四个特征：原子性（ Atomicity ）、一致性（ Consistency ）、隔离性（ Isolation ）和持续性（ Durability ）。这四个特性简称为 ACID 特性。

- 原子性。事务是数据库的逻辑工作单位，事务中包含的各操作要么都做，要么都不做
- 一致性。事 务执行的结果必须是使数据库从一个一致性状态变到另一个一致性状态。因此当数据库只包含成功事务提交的结果时，就说数据库处于一致性状态。如果数据库系统 运行中发生故障，有些事务尚未完成就被迫中断，这些未完成事务对数据库所做的修改有一部分已写入物理数据库，这时数据库就处于一种不正确的状态，或者说是 不一致的状态。
- 隔离性。一个事务的执行不能其它事务干扰。即一个事务内部的操作及使用的数据对其它并发事务是隔离的，并发执行的各个事务之间不能互相干扰。
- 持续性。也称永久性，指一个事务一旦提交，它对数据库中的数据的改变就应该是永久性的。接下来的其它操作或故障不应该对其执行结果有任何影响。

## 3.Mysql的四种隔离级别

SQL标准定义了4类隔离级别，包括了一些具体规则，用来限定事务内外的哪些改变是可见的，哪些是不可见的。低级别的隔离级一般支持更高的并发处理，并拥有更低的系统开销。

### Read Uncommitted（读取未提交内容）

在该隔离级别，所有事务都可以看到其他未提交事务的执行结果。本隔离级别很少用于实际应用，因为它的性能也不比其他级别好多少。读取未提交的数据，也被称之为脏读（Dirty Read）。

### Read Committed（读取提交内容）

这是大多数数据库系统的默认隔离级别（但不是MySQL默认的）。它满足了隔离的简单定义：一个事务只能看见已经提交事务所做的改变。这种隔离级别 也支持所谓的不可重复读（Nonrepeatable Read），因为同一事务的其他实例在该实例处理其间可能会有新的commit，所以同一select可能返回不同结果。

### Repeatable Read（可重读）

这是MySQL的默认事务隔离级别，它确保同一事务的多个实例在并发读取数据时，会看到同样的数据行。不过理论上，这会导致另一个棘手的问题：幻读 （Phantom Read）。简单的说，幻读指当用户读取某一范围的数据行时，另一个事务又在该范围内插入了新行，当用户再读取该范围的数据行时，会发现有新的“幻影” 行。InnoDB和Falcon存储引擎通过多版本并发控制（MVCC，Multiversion Concurrency Control）机制解决了该问题。

### Serializable（可串行化）

这是最高的隔离级别，它通过强制事务排序，使之不可能相互冲突，从而解决幻读问题。简言之，它是在每个读的数据行上加上共享锁。在这个级别，可能导致大量的超时现象和锁竞争。

这四种隔离级别采取不同的锁类型来实现，若读取的是同一个数据的话，就容易发生问题。例如：

- 脏读(Drity Read)：某个事务已更新一份数据，另一个事务在此时读取了同一份数据，由于某些原因，前一个RollBack了操作，则后一个事务所读取的数据就会是不正确的。
- 不可重复读(Non-repeatable read):在一个事务的两次查询之中数据不一致，这可能是两次查询过程中间插入了一个事务更新的原有的数据。
- 幻读(Phantom Read):在一个事务的两次查询中数据笔数不一致，例如有一个事务查询了几列(Row)数据，而另一个事务却在此时插入了新的几列数据，先前的事务在接下来的查询中，就有几列数据是未查询出来的，如果此时插入和另外一个事务插入的数据，就会报错。

在MySQL中，实现了这四种隔离级别，分别有可能产生问题如下所示：

![image-20200527121255296](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200527121255296.png)

## 4.测试Mysql的隔离级别

下面，将利用MySQL的客户端程序，我们分别来测试一下这几种隔离级别。

测试数据库为demo，表为test；表结构：

![image-20200527121345594](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200527121345594.png)

两个命令行客户端分别为A，B；不断改变A的隔离级别，在B端修改数据。

### 将A的隔离级别设置为read uncommitted(未提交读)


![img](https://mmbiz.qpic.cn/mmbiz/rtJ5Lhxxzwm1Mk8Ox5XvDSjhcZGicztZI64flb9aGZHRnsVml9cUjDkfbHTO2ljT0rVETjJoicl5I0CvMOOlNHpw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

A：启动事务，此时数据为初始状态

![img](https://mmbiz.qpic.cn/mmbiz_jpg/TNUwKhV0JpTmtXibg7dLhBHCricaRjbz8TfqBCdRFicyjJWZqGQ3DFaia1orkZ6ChOlVtqRFeHNxsDftmkWBwRjeibg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

B：启动事务，更新数据，但不提交

![img](https://mmbiz.qpic.cn/mmbiz_jpg/TNUwKhV0JpTmtXibg7dLhBHCricaRjbz8TAWEDomDhy1tHUygFQdoBttl2Y8x4icOWQiawGxMsn8EfWSqYDdnic3vfQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

A：再次读取数据，发现数据已经被修改了，这就是所谓的“脏读”

![img](https://mmbiz.qpic.cn/mmbiz/rtJ5Lhxxzwm1Mk8Ox5XvDSjhcZGicztZIGow8fznJngaPic6Ieo3zNoYnDK1o3ictSiaIEsPxQbc8uqLcHpXRwH0ow/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

B：回滚事务

![img](https://mmbiz.qpic.cn/mmbiz/rtJ5Lhxxzwm1Mk8Ox5XvDSjhcZGicztZIw4jpAicbNpXj8CkAXrcVaOXI1tkqxIISTMo5HWtn02DicH4kesNMKmWw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

A：再次读数据，发现数据变回初始状态

![img](https://mmbiz.qpic.cn/mmbiz/rtJ5Lhxxzwm1Mk8Ox5XvDSjhcZGicztZIKnLTNCtL812icYhmicZE9OPgTsteuHvQf0Z7jmz3JwHpzcHAib0x8GFOw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

经过上面的实验可以得出结论，事务B更新了一条记录，但是没有提交，此时事务A可以查询出未提交记录。造成脏读现象。未提交读是最低的隔离级别。

### 将客户端A的事务隔离级别设置为read committed(已提交读)

![img](https://mmbiz.qpic.cn/mmbiz/rtJ5Lhxxzwm1Mk8Ox5XvDSjhcZGicztZINTsSrC1etnzQtnux7wiaplEMibaicfh434bqxWRoaTDka8Z9N7GDWFlmw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

A：启动事务，此时数据为初始状态

![img](https://mmbiz.qpic.cn/mmbiz/rtJ5Lhxxzwm1Mk8Ox5XvDSjhcZGicztZIIHuDBibytlMCkx0UcerCcnvbmUKxHajKJ675GVzG7nJjOA3CQsWmDCA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

B：启动事务，更新数据，但不提交

![img](https://mmbiz.qpic.cn/mmbiz_jpg/TNUwKhV0JpTmtXibg7dLhBHCricaRjbz8TAWEDomDhy1tHUygFQdoBttl2Y8x4icOWQiawGxMsn8EfWSqYDdnic3vfQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

A：再次读数据，发现数据未被修改

![img](https://mmbiz.qpic.cn/mmbiz/rtJ5Lhxxzwm1Mk8Ox5XvDSjhcZGicztZIibvyezgtw4DHBRbsVF8ZPr7PYeKgE05P6Hb2LAyu7KHCkjibHGb21NAQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

B：提交事务

![img](https://mmbiz.qpic.cn/mmbiz/rtJ5Lhxxzwm1Mk8Ox5XvDSjhcZGicztZIr20Ndz3ya4yNKuhl41qaxbic5x9Jeku7YY4xPibDaYNTcMUlUzvYiaVpA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

A：再次读取数据，发现数据已发生变化，说明B提交的修改被事务中的A读到了，这就是所谓的“不可重复读”

![img](https://mmbiz.qpic.cn/mmbiz/rtJ5Lhxxzwm1Mk8Ox5XvDSjhcZGicztZItgcxcibzxj4NBejibbwd7to85tf923mI7gesnDAr3vkibmGW2qDrH7DhA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

经过上面的实验可以得出结论，已提交读隔离级别解决了脏读的问题，但是出现了不可重复读的问题，即事务A在两次查询的数据不一致，因为在两次查询之间事务B更新了一条数据。已提交读只允许读取已提交的记录，但不要求可重复读。

### 将A的隔离级别设置为repeatable read(可重复读)

![img](https://mmbiz.qpic.cn/mmbiz/rtJ5Lhxxzwm1Mk8Ox5XvDSjhcZGicztZIFuFPyib7ibhExyLfLQhlZ3eGicwlFD3UTnkys3qRFUS0P3WXAAoLS0k8w/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

A：启动事务，此时数据为初始状态

![img](https://mmbiz.qpic.cn/mmbiz/rtJ5Lhxxzwm1Mk8Ox5XvDSjhcZGicztZICGqiaZtrzGibiauh3VMxGKU0iaBrDyEg30hicWic4icuDUicZ9T59xubbQibChg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

B：启动事务，更新数据，但不提交

![img](https://mmbiz.qpic.cn/mmbiz/rtJ5Lhxxzwm1Mk8Ox5XvDSjhcZGicztZI3E4b9ic46DUD6F5VNuU0riaMMvQww9xY96ucsJJAn6v9wVeQFb2gaHaA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

A：再次读取数据，发现数据未被修改

![img](https://mmbiz.qpic.cn/mmbiz_jpg/TNUwKhV0JpTmtXibg7dLhBHCricaRjbz8TZRzLibMqAfypSkic1Gy7hlu9uea3YWuK38KQo8KJy4wttx9ggEPuHWHg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

B：提交事务

![img](https://mmbiz.qpic.cn/mmbiz/rtJ5Lhxxzwm1Mk8Ox5XvDSjhcZGicztZIr20Ndz3ya4yNKuhl41qaxbic5x9Jeku7YY4xPibDaYNTcMUlUzvYiaVpA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

A：再次读取数据，发现数据依然未发生变化，这说明这次可以重复读了

![img](https://mmbiz.qpic.cn/mmbiz_jpg/TNUwKhV0JpTmtXibg7dLhBHCricaRjbz8TZRzLibMqAfypSkic1Gy7hlu9uea3YWuK38KQo8KJy4wttx9ggEPuHWHg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

B：插入一条新的数据，并提交

![img](https://mmbiz.qpic.cn/mmbiz/rtJ5Lhxxzwm1Mk8Ox5XvDSjhcZGicztZI2jydicoeRELZOorgQ3zCxhibVwBIpfK6unHrLGvuEWUFcIPva41wLiaiaw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

A：再次读取数据，发现数据依然未发生变化，虽然可以重复读了，但是却发现读的不是最新数据，这就是所谓的“幻读”

![img](https://mmbiz.qpic.cn/mmbiz_jpg/TNUwKhV0JpTmtXibg7dLhBHCricaRjbz8TZRzLibMqAfypSkic1Gy7hlu9uea3YWuK38KQo8KJy4wttx9ggEPuHWHg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

A：提交本次事务，再次读取数据，发现读取正常了

![img](https://mmbiz.qpic.cn/mmbiz/rtJ5Lhxxzwm1Mk8Ox5XvDSjhcZGicztZIEmUa4OibSZ1x2nSIylPyynGtjCR5M3BZUsPkYYmD8As7SOmeTGkcOVg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

由以上的实验可以得出结论，可重复读隔离级别只允许读取已提交记录，而且在一个事务两次读取一个记录期间，其他事务部的更新该记录。但该事务不要求与其他事务可串行化。例如，当一个事务可以找到由一个已提交事务更新的记录，但是可能产生幻读问题(注意是可能，因为数据库对隔离级别的实现有所差别)。像以上的实验，就没有出现数据幻读的问题。

### 将A的隔离级别设置为可串行化(Serializable)

![img](https://mmbiz.qpic.cn/mmbiz/rtJ5Lhxxzwm1Mk8Ox5XvDSjhcZGicztZIWlBcSV0ZiaraEtQ6FjNc7S16Lsn8A1Zn3hvshNiaj1F7nSpmyNZ0Qgew/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

A：启动事务，此时数据为初始状态

![img](https://mmbiz.qpic.cn/mmbiz/rtJ5Lhxxzwm1Mk8Ox5XvDSjhcZGicztZIUib5WKRsLU3q5NQcU9BnV1Gg1rJHnlAL7zm7be5dCCoDpSE8oRibSOjQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

B：发现B此时进入了等待状态，原因是因为A的事务尚未提交，只能等待（此时，B可能会发生等待超时）

![img](https://mmbiz.qpic.cn/mmbiz/rtJ5Lhxxzwm1Mk8Ox5XvDSjhcZGicztZIwZ6qHYPBTu2DQNWF9XicgKwfWxvHbY5YU2Ovgwbg0AnLibibeicJpByDCA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

A：提交事务

![img](https://mmbiz.qpic.cn/mmbiz/rtJ5Lhxxzwm1Mk8Ox5XvDSjhcZGicztZIpeAyYczm2lJ9rUW1XMYjxNguhN1fjtqj6ibeFywLfEamLsFQG0FcBjw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

B：发现插入成功

![img](https://mmbiz.qpic.cn/mmbiz/rtJ5Lhxxzwm1Mk8Ox5XvDSjhcZGicztZIicJial30Ogu6kagul3fqzHTYJXQ1eTUG6L3uRFYcX5HCRhwtNG7dWXhQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

serializable完全锁定字段，若一个事务来查询同一份数据就必须等待，直到前一个事务完成并解除锁定为止。是完整的隔离级别，会锁定对应的数据表格，因而会有效率的问题。

# 使用终端操作

## 数据库基本操作

#### --登录数据库

```mysql
mysql -u用户名 -p密码
mysql -uroot -p123456
```

#### --退出数据库

```
exit;
```

#### --查看数据库

```mysql
show databases;
```

#### --选择数据库

```
use 数据库名字;
```

#### --查看数据库中的表

```
show tables;
```

#### --查询表中的所有数据

```
select * from 表名 where.....;
```

#### --创建数据库

```
create database 名称;
```

#### --创建数据表

```
create table hjj(
name varchar(20),
birth varchar(20),
sex varchar(20)
);
```

![image-20200522222550720](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200522222550720.png)

#### --查看数据表结构详情

```
describe hjj;
```

![image-20200522222712555](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200522222712555.png)

#### --插入新的数据

```
insert into hjj
values('hjj','971024','boy');
```

#### --数据类型

MySQL支持多种类型，大致可以分为三类：数值、日期/时间和字符串(字符)类型。

##### 	数值类型

| 类型         | 大小                                     | 范围（有符号）                                               | 范围（无符号）                                               |      用途       |
| :----------- | :--------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :-------------: |
| TINYINT      | 1 byte                                   | (-128，127)                                                  | (0，255)                                                     |    小整数值     |
| SMALLINT     | 2 bytes                                  | (-32 768，32 767)                                            | (0，65 535)                                                  |    大整数值     |
| MEDIUMINT    | 3 bytes                                  | (-8 388 608，8 388 607)                                      | (0，16 777 215)                                              |    大整数值     |
| INT或INTEGER | 4 bytes                                  | (-2 147 483 648，2 147 483 647)                              | (0，4 294 967 295)                                           |    大整数值     |
| BIGINT       | 8 bytes                                  | (-9,223,372,036,854,775,808，9 223 372 036 854 775 807)      | (0，18 446 744 073 709 551 615)                              |   极大整数值    |
| FLOAT        | 4 bytes                                  | (-3.402 823 466 E+38，-1.175 494 351 E-38)，0，(1.175 494 351 E-38，3.402 823 466 351 E+38) | 0，(1.175 494 351 E-38，3.402 823 466 E+38)                  | 单精度 浮点数值 |
| DOUBLE       | 8 bytes                                  | (-1.797 693 134 862 315 7 E+308，-2.225 073 858 507 201 4 E-308)，0，(2.225 073 858 507 201 4 E-308，1.797 693 134 862 315 7 E+308) | 0，(2.225 073 858 507 201 4 E-308，1.797 693 134 862 315 7 E+308) | 双精度 浮点数值 |
| DECIMAL      | 对DECIMAL(M,D) ，如果M>D，为M+2否则为D+2 | 依赖于M和D的值                                               | 依赖于M和D的值                                               |     小数值      |

##### 	日期和时间类型

| 类型      | 大小 ( bytes) | 范围                                                         | 格式                |           用途           |
| :-------- | :------------ | :----------------------------------------------------------- | :------------------ | :----------------------: |
| DATE      | 3             | 1000-01-01/9999-12-31                                        | YYYY-MM-DD          |          日期值          |
| TIME      | 3             | '-838:59:59'/'838:59:59'                                     | HH:MM:SS            |     时间值或持续时间     |
| YEAR      | 1             | 1901/2155                                                    | YYYY                |          年份值          |
| DATETIME  | 8             | 1000-01-01 00:00:00/9999-12-31 23:59:59                      | YYYY-MM-DD HH:MM:SS |     混合日期和时间值     |
| TIMESTAMP | 4             | 1970-01-01 00:00:00/2038结束时间是第 **2147483647** 秒，北京时间 **2038-1-19 11:14:07**，格林尼治时间 2038年1月19日 凌晨 03:14:07 | YYYYMMDD HHMMSS     | 混合日期和时间值，时间戳 |

##### 	字符串类型

| 类型       | 大小                  |              用途               |
| :--------- | :-------------------- | :-----------------------------: |
| CHAR       | 0-255 bytes           |           定长字符串            |
| VARCHAR    | 0-65535 bytes         |           变长字符串            |
| TINYBLOB   | 0-255 bytes           | 不超过 255 个字符的二进制字符串 |
| TINYTEXT   | 0-255 bytes           |          短文本字符串           |
| BLOB       | 0-65 535 bytes        |     二进制形式的长文本数据      |
| TEXT       | 0-65 535 bytes        |           长文本数据            |
| MEDIUMBLOB | 0-16 777 215 bytes    |  二进制形式的中等长度文本数据   |
| MEDIUMTEXT | 0-16 777 215 bytes    |        中等长度文本数据         |
| LONGBLOB   | 0-4 294 967 295 bytes |    二进制形式的极大文本数据     |
| LONGTEXT   | 0-4 294 967 295 bytes |          极大文本数据           |

**char(n) 和 varchar(n) 中括号中 n 代表字符的个数，并不代表字节个数，比如 CHAR(30) 就可以存储 30 个字符。**

**CHAR 和 VARCHAR 类型类似，但它们保存和检索的方式不同。它们的最大长度和是否尾部空格被保留等方面也不同。在存储或检索过程中不进行大小写转换。**

#### --删除记录

```
delete from hjj where name='hjj';
```

#### --更新记录

```
update hjj set name='sas' where sex='girl';
```

## 约束条件

#### --主键约束primary key

能够唯一确定表中的一条记录，给某一个字段添加约束使得字段不重复且不为空

```
create table user(
id int primary key,
name varchar(20)
);
```

​		==联合主键==：可以给多个字段设置主键,每个字段都不能为空，并且需要他们加起来不一样

```
create table user1(
id int,
name varchar(20),
primary key(id,name)
);
```

#### --自增约束

#### --外键约束

#### --唯一约束

#### --非空约束

#### --默认约束

