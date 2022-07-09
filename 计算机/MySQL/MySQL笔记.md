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

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/mysql/%E6%9F%A5%E8%AF%A2%E8%AF%B7%E6%B1%82%E6%89%A7%E8%A1%8C%E8%BF%87%E7%A8%8B.png" width="700" height="900" align="left" alt="查询请求执行流程">

#### 2.2.1 连接管理

每当有一个客户端连接到服务器进程时，服务器进程都会创建一个线程专门处理与这个客户端的交互；当客户端与服务器断开连接时，服务器并不会吧与该客户端交互的线程晓辉，而是把它缓存起来





