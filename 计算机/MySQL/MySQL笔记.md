# MySQL

## 一、相关命令

### LIMIT

`LIMIT`命令用于限制返回的行数，`LIMIT 1`表示返回结果的第一行

`LIMIT row1, row2` 表示返回行row1（行0位第一行）开始的第一行

`LIMIT row1 OFFSET row2`，表示从行row2（行0为第一行）取row1行

故`LIMIT 3, 4` 和 `LIMIT 4 OFFSET 3`表示相同的意思



### ORDER BY

`ORDER BY`默认是升序排列，对于字母也就是按照A-Z的顺序排列，降序排列需要在后面加上`DESC`

对于多个列可以指定不同的排序方式，例如

```mysql
SELECT prod_id, prod_price, prod_name
FROM products.
```

搜索结果按照prod_price降序排列，当多个结果的prod_price相同时，根据其prod_name升序排序



### `AND`运算符和`OR`运算符的计算次序

当`AND`运算符和`OR`运算符同时出现时，SQL会有限处理AND操作符，例如下面SQL语句：

```mysql
SELECT prod_name, prod_price
FROM products
WHERE vend_id = 1002 OR vend_id = 1003 AND prod_price >= 0
```

等同于

```mysql
SELECT prod_name, prod_price
FROM products
WHERE vend_id = 1002 OR （vend_id = 1003 AND prod_price >= 0）
```



### 通配符

+ 通配符前接上`LIKE`操作符
+ `%`表示任何字符出现任意次数(包含了出现0次的情况)，`%`无法匹配**NULL**
+ `_`匹配单个字符
+ 不要过度使用通配符。如果其他操作符能达到相同的目的，应该使用其他操作符
+ 在确定使用通配符时，除非绝对有必要，否则不要把它们用在搜索模式的**开始处**。把通配符置于搜索模式的开始出，搜索起来是最慢的



### 正则表达式

+ 正则表达式需要使用`REGEXP`操作符
+ 当`LIKE`操作符后面没有通配符时，比如`WHERE id LIKE '1000'`，它会匹配是否有id**等**于1000的行；而`REGEXP`则会匹配id这一列中，是否有一列的值**包含**了1000这个字符串，
+ 使用正则表达式进行搜索时，如果要区分大小写，需要在`REGEXP`后面加上`BINARY`关键字，也就是`REGEXP BINARY`
+ `|`为正则表达式的OR操作符，他匹配其中之一，例如`WHERE id REGEXP '1000|2000'`表示查找id中有字符串1000或者2000的行
+ `[]`用于匹配几个字符之一，例如`[123]`表示匹配包含有1或2或者3的字符串
+ `.`表示匹配任意一个字符
+ 如果要匹配含有`.`的字符，因为`.`是通配符的原因，需要在前面加上`\\`，例如`WHERE name REGEXP '\\.'`



### 聚集函数

+ `AVG()`函数、`MAX()`函数、`MIN()`函数和`SUM()`函数**忽略**列值为NULL的行
+ 使用`COUNT(*)`对表中行的数目进行计数，不管表列中包含的是空值（NULL）还是非空值
+ 使用`COUNT(column)`对特定列中**具有值的行进行计数，忽略NULL值**



### 分组

+ **除了聚集函数外，SELECT语句中的每个列都必须出现在`GROUP BY`子句中**

+ 如果分组中具有NULL值，则NULL将作为一个分组返回。如果列中有多行NULL值，它们将分为一组
+ `GROUP BY`子句必须出现在`WHERE`子句之后，`ORDER BY`子句之前，因为`WHERE`是用来过滤行的，而`HAVING`应该出现在`ORDER BY`之后，`HAVING`语句是用来过滤分组的
+ `SELECT`子句顺序
  + SELECT
  + FROM
  + WHERE
  + GROUP BY
  + HAVING
  + ORDER BY
  + LIMIT
+ 

### 全文本搜索

+ MySQL最常使用的两个引擎`MyISAM`和`InnoDB`，前者支持全文本搜索，后者不支持
+ 使用全文本搜索（也就是使用`Match()`和`Against()`函数）相较于使用`LIKE`关键字来说，前者会根据搜索的字符串在文本中的顺序进行排序，搜索目标在文本中越靠前，优先级越高，最后显示地结果就越先返回





## 二、MySQL基本知识

### 2.1 客户端连接

启动客户端基本格式

```mysql
mysql -h主机名 -u用户名 -p密码
```



各个参数的意思

| 参数名 | 含义 |
| :---: | :------------- |
| -h | 表示服务器所在计算机的域名或者IP地址。如果服务器在本机运行的话，可以省略这个参数，或者输入`localhost`或者`127.0.0.1`；也可以写成`--host=主机名`的形式 |
| -u | 表示用户名；也可以写成`--user=用户名` |
| -p | 表示密码；也可以写成`--password=密码` |



如果不想显示密码可以输入下面命令，按Enter后会要求输入密码

```mysql
mysql -hlocalhost -uroot -p
```



### 2.2 MySQL处理客户端请求

以查询请求执行为例，流程图如下

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/mysql/MySQL%E9%80%BB%E8%BE%91%E6%9E%B6%E6%9E%84%E5%9B%BE.webp" width="800" height="600" align="left" alt="查询请求执行流程">

#### 连接管理

每当有一个客户端连接到服务器进程时，服务器进程都会创建一个线程专门处理与这个客户端的交互；当客户端与服务器断开连接时，服务器并不会吧与该客户端交互的线程晓辉，而是把它缓存起来，在另一个新的客户端再进行连接时，把这个缓存的线程分配给该新客户端



#### 查询缓存

MySQL 拿到一个查询请求后，会先到查询缓存看看，之前是不是执行过这条语句。之前执行过的语句及其结果可能会以 key-value 对的形式，被直接缓存在内存中。key 是查询的语句，value 是查询的结果。

如果你的查询能够直接在这个缓存中找到 key，那么这个 value 就会被直接返回给客户端。如果语句不在查询缓存中，就会继续后面的执行阶段。执行完成后，执行结果会被存入查询缓存中。你可以看到，如果查询命中缓存，MySQL 不需要执行后面的复杂操作，就可以直接返回结果，这个效率会很高

MySQL的缓存系统会监测涉及的每张表，只要该表的结构或者数据被修改，比如对该表使用了INSERT、UPDATE、DELETE、TRUNCATE TABLE、ALTER TABLE、DROP TABLE或DROP DATABASE语句，则与该表有关的所有查询缓存都将变为无效并从查询缓存中删除

> 维护缓存可能会造成一些不可能的开销，因此从MySQL 5.7.20开始，不推荐使用查询缓存，在MySQL 8.0 中直接将其删除



#### 分析器

如果没有命中查询缓存，就要开始真正执行语句了。首先，MySQL 需要知道你要做什么，因此需要对 SQL 语句做解析。

分析器先会做“词法分析”。你输入的是由多个字符串和空格组成的一条 SQL 语句，MySQL 需要识别出里面的字符串分别是什么，代表什么。MySQL 从你输入的"select"这个关键字识别出来，这是一个查询语句。它也要把字符串“T”识别成“表名 T”，把字符串“ID”识别成“列 ID”。

做完了这些识别以后，就要做“语法分析”。根据词法分析的结果，语法分析器会根据语法规则，判断你输入的这个 SQL 语句是否满足 MySQL 语法。

如果你的语句不对，就会收到“You have an error in your SQL syntax”的错误提醒，比如下面这个语句 select 少打了开头的字母“s”。



#### 优化器

在分析器之后，服务器程序获得了需要的信息，比如要查的表和列是哪些，搜索条件是什么。

优化器会对我们的语句做出一些优化，如外连接转换为内连接，表达式简化、子查询转为连接等一堆东西。

优化的结果就是生成一个执行计划，这个执行计划表明了应该使用哪些索引进行查询，以及表之间的连接顺序是什么样的。

可以使用`EXPLAIN`语句来查询某个语句的执行计划



#### 执行器

MySQL 通过分析器知道了你要做什么，通过优化器知道了该怎么做，于是就进入了执行器阶段，开始执行语句。

开始执行的时候，要先判断一下你对这个表 T 有没有执行查询的权限，如果没有，就会返回没有权限的错误，如下所示 (在工程实现上，如果命中查询缓存，会在查询缓存返回结果的时候，做权限验证。

查询也会在优化器之前调用 precheck 验证权限)。

```mysql
mysql> select * from T where ID=10;

ERROR 1142 (42000): SELECT command denied to user 'b'@'localhost' for table 'T'
```

如果有权限，就打开表继续执行。打开表的时候，执行器就会根据表的引擎定义，去使用这个引擎提供的接口。

比如我们这个例子中的表 T 中，ID 字段没有索引，那么执行器的执行流程是这样的：调用 InnoDB 引擎接口取这个表的第一行，判断 ID 值是不是 10，如果不是则跳过，如果是则将这行存在结果集中；调用引擎接口取“下一行”，重复相同的判断逻辑，直到取到这个表的最后一行。

执行器将上述遍历过程中所有满足条件的行组成的记录集作为结果集返回给客户端。至此，这个语句就执行完成了。对于有索引的表，执行的逻辑也差不多。

第一次调用的是“取满足条件的第一行”这个接口，之后循环取“满足条件的下一行”这个接口，这些接口都是引擎中已经定义好的。你会在数据库的慢查询日志中看到一个 rows_examined 的字段，表示这个语句执行过程中扫描了多少行。

这个值就是在执行器每次调用引擎获取数据行的时候累加的。在有些场景下，执行器调用一次，在引擎内部则扫描了多行，因此引擎扫描行数跟 rows_examined 并不是完全相同的。



#### 存储引擎

在逻辑上，表是由一行一行的记录组成，但在物理上如何表示记录，怎么从表中读取数据，以及怎么把数据写入具体的物理存储引擎上，都是存储引擎负责的事情

## 三、 InnoDB

### 3.1 InnoDB概述

+ InnoDB是一个将表中的数据存储到磁盘上的存储引擎
+ InnoDB将数据划分为若干个页，以页作为磁盘和内存之间交互的基本单位。InnoDB的页的大小默认为16KB



### 3.2 InnoDB记录存储结构

我们平时都是以记录为单位向表中插入数据的，这些记录在磁盘上的存放形式被称为**行格式**或**记录格式**，一条记录也就是表中的一行数据，InnoDB一共有4中不同类型的行格式（记录格式）

+ COMPACT
+ REDUNDANT
+ DYNAMIC
+ COMPRESSED



### 3.3 COMPACT行格式

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/mysql/COMPACT%E8%A1%8C%E6%A0%BC%E5%BC%8F%E7%A4%BA%E6%84%8F%E5%9B%BE.png" align="left" alt="COMPACT行格式" height="100">



+ 记录的额外信息

  + 变长字段长度列表
    + ​	变长字段是指一些变长的数据类型，例如`VARCHAR(M)`、`VARBINARY(M)`、`TEXT`类型和`BLOB`类型。

  + NULL值列表

  + 记录头信息

+ 记录的真实数据 

## 事务隔离

### 一致性问题

+ 脏写（Dirty Write）
  + 一个事务修改了另一个未提交事务修改过的数据
+ 脏读（Dirty Read）
  + 一个事务读到了另一个未提交事务修改过的数据
+ 不可重复读（Non-Repeatable Read）
  + 一个事务修改了另一个未提交事务读取的数据
+ 幻读（Phantom）
  + 一个事务先根据某些搜索条件查询出一些记录，在该事务未提交时，另一个事务写入了一些符合哪些搜索条件的记录（例如INSERT、DELETE和UPDATE操作）



### MySQL实现ACID

+ 一致性
  + 用undo log实现
+ 隔离性
  + 加锁
  + 隔离级别
+ 持久性
  + redo log



## 范式（Normal Form）

### 范式概述

在关系型数据库中，关于数据库表的设计的基本原则、规则就称为范式

目前关系型数据库有六种常见范式，按照范式级别，从低到高分别是

+ 第一范式（1NF）
+ 第二范式（2NF）
+ 第三范式（3NF）
+ 巴斯-科德范式（BCNF）
+ 第四范式（4NF）
+ 第五范式（5NF，又称完美范式）

数据库的范式设计越高阶，冗余度就越低，同时高阶范式一定符合低阶范式的要求，满足最低要求的范式是第一范式（1NF）



### 相关概念

如果球员表属性有：球员编号、姓名、身份证号、年龄、球队编号



+ 超键：能唯一标识元组（一行数据）的属性
  + 对于球员表，超键有（球员编号）、（球员编号，姓名）、（身份证号）、（球员编号，姓名，身份证号）等
+ 候选键（又称为码）：如果超键不包括多种多余的属性叫做超键
  + 对于球员表，候选键有球员编号和身份证号
+ 主键（又称为主码）：用户可以从候选键中选择一个作为主键
+ 外键：如果数据表R1中的某属性集不是R1的主键，而是另一个数据表R2的主键，那么这个属性集就是数据表R1的外键
+ 主属性：包含在任意候选键中的属性
+ 非主属性：不包含在任意一个候选键中的属性



### 第一范式（1st NF）

第一范式主要是确保数据表中的每个字段必须具有原子性，也就是说数据表中的每个字段的值是不可拆分的最小数据单元

但是属性的原子性是主观的，例如姓名可以拆分成first name 、middle name 和 last name三部分，至于是否要拆分，根据业务需求而定



### 第二范式

第二范式（2NF）要求实体的属性完全依赖于主关键字。所谓完全依赖是指不能存在仅依赖主关键字一部分的属性。如果存在，那么这个属性和主关键字的这一部分应该分离出来形成一个新的实体，新实体与原实体之间是一对多的关系。为实现区分通常需要为表加上一个列，以存储各个实例的唯一标识。

简而言之，第二范式就是非主属性非部分依赖于主关键字。**即当1NF消除了非主属性对码的部分函数依赖，则称为2NF**。

**2NF优化发生在联合主键情况下**，即一张表的主键为KEY（A,B），而表中属性C只依赖于A，则成为C对主键的部分依赖。**对于单主键的表，如果优化为1NF后，自然就已经满足2NF了。**

不符合第二范式的例子：

| 货物类型 | 货物ID | 货物名称   | 注意事项           |
| -------- | ------ | ---------- | ------------------ |
| 瓷碗     | 1      | 白色瓷碗   | 易碎品             |
| 瓷碗     | 2      | 青花瓷碗   | 易碎品             |
| 瓷碗     | 3      | 雕花瓷碗   | 易碎品             |
| 三合板   | 1      | 普通三合板 | 易燃物品，注意防火 |

　　在该表中主键为（货物类型，货物ID），货物名称字段完全依赖于这个主键，换句话说，货物的名称完全是取决于这个主键的值的。但“注意事项”这一列，仅依赖于一个主键中”货物类型“这一个属性。根据第二范式规定，既然表中存在一个对主键不是完全依赖的字段，那么我们就可以确定，该表不符合第二范式。



### 第三范式

任何非主属性不依赖于其它非主属性。

存在一个部门信息表，其中每个部门有部门编号（dept_id）、部门名称、部门简介等信息。那么员工信息表中列出部门编号后就不能再将部门名称、部门简介等与部门有关的信息再加入员工信息表中。如果不存在部门信息表，则根据第三范式（3NF）也应该构建它，否则就会有大量的数据冗余。即：A（员工ID）、B（部门ID）、C（部门名称），三列中A->B，A->（B->）C，满足第二范式，但是由于A->C是通过B来传递的，因此不符合第三范式。

## MySQL安装

### Step 1 – Enable MySQL Repository

First of all, You need to enable MySQL 5.7 community release yum repository on your system. The rpm packages for yum repository configuration are available on MySQL’s official website.

First of all, import the latest MySQL GPG key to your system.

```
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022 
```

Now, use one of the below commands to configure the Yum repository as per your operating system version.

- On CentOS & RHEL 7:

  ```
  sudo yum localinstall https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm 
  ```

- On Fedora 36

  ```
  sudo dnf install https://dev.mysql.com/get/mysql57-community-release-fc27-11.noarch.rpm 
  ```

- On Fedora 35

  ```
  sudo dnf install https://dev.mysql.com/get/mysql57-community-release-fc26-11.noarch.rpm 
  ```

### Step 2 – Installing MySQL 5.7 Server

As you have successfully enabled MySQL yum repository on your system. Now, install MySQL 5.7 community server using the following commands as per your operating system version.

- On CentOS & RHEL 7:

  ```
  sudo yum install mysql-community-server 
  ```

- On Fedora 36/35:

  ```
  sudo dnf install mysql-community-server 
  ```

The above command will install the MySQL community server and other dependencies on your system. During the installation process of packages, a temporary password is created and logged to MySQL log files. Use the following command to find your temporary MySQL password.

After installing RPMs, use the following command to start MySQL Service.

```
sudo systemctl start mysqld 
```

During the first start, MySQL stores the root account password in log file, That can be found with the followign command.

```
grep 'A temporary password' /var/log/mysqld.log |tail -1 
```

Sample output:

```
2017-03-30T02:57:10.981502Z 1 [Note] A temporary password is generated for root@localhost: Nm(!pKkkjo68e
```



### 使用临时密码登录MySQL，并修改密码

```
mysql -uroot -p  
```

输入上面的临时密码

修改密码规则（mysql5.7默认密码策略要求密码必须是大小写字母数字特殊字母的组合，至少8位，嫌麻烦可以先修改密码规则策略）

```
set global validate_password_policy=0;
set global validate_password_length=1;
```

修改密码

```
ALTER USER 'root'@'localhost' IDENTIFIED BY 'demo2020';
```

### 授权其他机器远程登录

```
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'demo2020' WITH GRANT OPTION;
```

**注意**：不要加上`FLUSH PRIVILEGES;`

### 设置开机自启动

先退出MySQL

```
mysql> exit
```
设置开机启动，然后重新加载新的unit 配置文件  
```
systemctl enable mysqld     
systemctl daemon-reload
```



### 设置MySQL的字符集为UTF-8，另其支持中国

```
vim /etc/my.cnf
```

```
# For advice on how to change settings please see
# http://dev.mysql.com/doc/refman/5.7/en/server-configuration-defaults.html

[mysqld]
#
# Remove leading # and set to the amount of RAM for the most important data
# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
# innodb_buffer_pool_size = 128M
#
# Remove leading # to turn on a very important data integrity option: logging
# changes to the binary log between backups.
# log_bin
#
# Remove leading # to set options mainly useful for reporting servers.
# The server defaults are faster for transactions and fast SELECTs.
# Adjust sizes as needed, experiment to find the optimal values.
# join_buffer_size = 128M
# sort_buffer_size = 2M
# read_rnd_buffer_size = 2M
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
default-storage-engine=INNODB
character_set_server=utf8
```

加上最后这两行

重启MySQL

```
systemctl restart mysqld;
```





