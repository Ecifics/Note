# Mybatis

[TOC]

## 关键字
### 1. resultType和resultMap
	resultType:设置默认的映射关系（当数据库中的列名和实体类的字段名相同时，使用resultType）
	resultMap:设置自定义的映射关系（当数据库中的列名和实体类的字段名不同时，使用resultMap）

### 2. MySQL url
	jdbc:mysql://localhost:3306/mybatis?characterEncoding=utf8&useSSL=false&serverTimezone=UTC&rewriteBatchedStatements=true 

## 搭建Mybatis

### 1. mybatis配置文件
#### (1) jdbc.properties（放在resources目录下）
```properties
jdbc.driver=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/mybatis?characterEncoding=utf8&useSSL=false&serverTimezone=UTC&rewriteBatchedStatements=true 
jdbc.username=root
jdbc.password=000715
```

#### (2) mybatis-config.xml （放在resources目录下）
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <properties resource="jdbc.properties" />

    <typeAliases>
        <package name="" />
    </typeAliases>

    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <package name="" />
    </mappers>
</configuration>
```
其中`<properties resource="jdbc.properties" />`引入了jdbc.properties文件
引用properties文件中的数据需要使用`${property_key}`来引用properties文件中的键值，例如`<enviroments>`标签中`<property name="driver" value="${jdbc.driver}"/>`中引用了properties文件中jdbc.driver的值

### 2. 相关文件目录的创建
(1) 首先在java目录下创建一个Package，例如`com.ecifics`
(2) 创建mapper包，用于存放Mapper接口文件（对应的包名为`com.ecifics.mapper`）
(3) 在resources文件下创建Directory用于存放映射文件（接口文件对应的xml文件）。
	**注意**，在创建Directory时，输入的名字和创建Package方式不同，不同的Directory之间需要用`/`来作为分隔符（而创建Package时使用`.`来作为分隔符）。
	因此创建出来的包名为`com/ecifics/mapper`和上述Mapper接口文件包名相同

### 4. 日志
（1）引入log4j依赖
```xml
<!-- https://mvnrepository.com/artifact/log4j/log4j -->
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```
（2）配置log4j，在`resources`目录下创建配置文件`log4j.xml`，文件内容如下
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE log4j:configuration SYSTEM "log4j.dtd">
<log4j:configuration xmlns:log4j="http://jakarta.apache.org/log4j/">
    <appender name="STDOUT" class="org.apache.log4j.ConsoleAppender">
        <param name="Encoding" value="UTF-8" />
        <layout class="org.apache.log4j.PatternLayout">
            <param name="ConversionPattern" value="%-5p %d{MM-dd HH:mm:ss,SSS} %m  (%F:%L) \n" />
        </layout>
    </appender>
    <logger name="java.sql">
        <level value="debug" />
    </logger>
    <logger name="org.apache.ibatis">
        <level value="info" />
    </logger>
    <root>
        <level value="debug" />
        <appender-ref ref="STDOUT" />
    </root>
</log4j:configuration>
```


## Mybatis核心配置文件
### 1. 配置文档结构（必须按照下面这个顺序排列，否则会出错）

    configuration（配置）
        properties（属性）
        settings（设置）
        typeAliases（类型别名）
        typeHandlers（类型处理器）
        objectFactory（对象工厂）
        plugins（插件）
        environments（环境配置）
            environment（环境变量）
                transactionManager（事务管理器）
                dataSource（数据源）
        databaseIdProvider（数据库厂商标识）
        mappers（映射器）
### 2. properties（属性）

这些属性可以在外部进行配置，并可以进行动态替换。你既可以在典型的 Java 属性文件中配置这些属性，也可以在 properties 元素的子元素中设置。例如：

```properties
<properties resource="org/mybatis/example/config.properties">
  <property name="username" value="dev_user"/>
  <property name="password" value="F2Fa3!33TYyg"/>
</properties>
```

或者可以通过在mybatis-config.xml文件中在properties标签中的resource属性指明properties文件的位置

```xml
<properties resource="jdbc.properties" />
```



这样就可以在配置文件中通过`$`来使用properties文件中的属性了，例如配置enviroments

```java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <properties resource="jdbc.properties" />

    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>
</configuration>
```



### 3. settings（设置）

这是 MyBatis 中极为重要的调整设置，它们会改变 MyBatis 的运行时行为。 下表描述了设置中各项设置的含义、默认值等。

| 设置名                           | 描述                                                         | 有效值                                                       | 默认值                                                |
| :------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :---------------------------------------------------- |
| cacheEnabled                     | 全局性地开启或关闭所有映射器配置文件中已配置的任何缓存。     | true \| false                                                | true                                                  |
| lazyLoadingEnabled               | 延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。 特定关联关系中可通过设置 `fetchType` 属性来覆盖该项的开关状态。 | true \| false                                                | false                                                 |
| aggressiveLazyLoading            | 开启时，任一方法的调用都会加载该对象的所有延迟加载属性。 否则，每个延迟加载属性会按需加载（参考 `lazyLoadTriggerMethods`)。 | true \| false                                                | false （在 3.4.1 及之前的版本中默认为 true）          |
| multipleResultSetsEnabled        | 是否允许单个语句返回多结果集（需要数据库驱动支持）。         | true \| false                                                | true                                                  |
| useColumnLabel                   | 使用列标签代替列名。实际表现依赖于数据库驱动，具体可参考数据库驱动的相关文档，或通过对比测试来观察。 | true \| false                                                | true                                                  |
| useGeneratedKeys                 | 允许 JDBC 支持自动生成主键，需要数据库驱动支持。如果设置为 true，将强制使用自动生成主键。尽管一些数据库驱动不支持此特性，但仍可正常工作（如 Derby）。 | true \| false                                                | False                                                 |
| autoMappingBehavior              | 指定 MyBatis 应如何自动映射列到字段或属性。 NONE 表示关闭自动映射；PARTIAL 只会自动映射没有定义嵌套结果映射的字段。 FULL 会自动映射任何复杂的结果集（无论是否嵌套）。 | NONE, PARTIAL, FULL                                          | PARTIAL                                               |
| autoMappingUnknownColumnBehavior | 指定发现自动映射目标未知列（或未知属性类型）的行为。`NONE`: 不做任何反应`WARNING`: 输出警告日志（`'org.apache.ibatis.session.AutoMappingUnknownColumnBehavior'` 的日志等级必须设置为 `WARN`）`FAILING`: 映射失败 (抛出 `SqlSessionException`) | NONE, WARNING, FAILING                                       | NONE                                                  |
| defaultExecutorType              | 配置默认的执行器。SIMPLE 就是普通的执行器；REUSE 执行器会重用预处理语句（PreparedStatement）； BATCH 执行器不仅重用语句还会执行批量更新。 | SIMPLE REUSE BATCH                                           | SIMPLE                                                |
| defaultStatementTimeout          | 设置超时时间，它决定数据库驱动等待数据库响应的秒数。         | 任意正整数                                                   | 未设置 (null)                                         |
| defaultFetchSize                 | 为驱动的结果集获取数量（fetchSize）设置一个建议值。此参数只可以在查询设置中被覆盖。 | 任意正整数                                                   | 未设置 (null)                                         |
| defaultResultSetType             | 指定语句默认的滚动策略。（新增于 3.5.2）                     | FORWARD_ONLY \| SCROLL_SENSITIVE \| SCROLL_INSENSITIVE \| DEFAULT（等同于未设置） | 未设置 (null)                                         |
| safeRowBoundsEnabled             | 是否允许在嵌套语句中使用分页（RowBounds）。如果允许使用则设置为 false。 | true \| false                                                | False                                                 |
| safeResultHandlerEnabled         | 是否允许在嵌套语句中使用结果处理器（ResultHandler）。如果允许使用则设置为 false。 | true \| false                                                | True                                                  |
| mapUnderscoreToCamelCase         | 是否开启驼峰命名自动映射，即从经典数据库列名 A_COLUMN 映射到经典 Java 属性名 aColumn。 | true \| false                                                | False                                                 |
| localCacheScope                  | MyBatis 利用本地缓存机制（Local Cache）防止循环引用和加速重复的嵌套查询。 默认值为 SESSION，会缓存一个会话中执行的所有查询。 若设置值为 STATEMENT，本地缓存将仅用于执行语句，对相同 SqlSession 的不同查询将不会进行缓存。 | SESSION \| STATEMENT                                         | SESSION                                               |
| jdbcTypeForNull                  | 当没有为参数指定特定的 JDBC 类型时，空值的默认 JDBC 类型。 某些数据库驱动需要指定列的 JDBC 类型，多数情况直接用一般类型即可，比如 NULL、VARCHAR 或 OTHER。 | JdbcType 常量，常用值：NULL、VARCHAR 或 OTHER。              | OTHER                                                 |
| lazyLoadTriggerMethods           | 指定对象的哪些方法触发一次延迟加载。                         | 用逗号分隔的方法列表。                                       | equals,clone,hashCode,toString                        |
| defaultScriptingLanguage         | 指定动态 SQL 生成使用的默认脚本语言。                        | 一个类型别名或全限定类名。                                   | org.apache.ibatis.scripting.xmltags.XMLLanguageDriver |
| defaultEnumTypeHandler           | 指定 Enum 使用的默认 `TypeHandler` 。（新增于 3.4.5）        | 一个类型别名或全限定类名。                                   | org.apache.ibatis.type.EnumTypeHandler                |
| callSettersOnNulls               | 指定当结果集中值为 null 的时候是否调用映射对象的 setter（map 对象时为 put）方法，这在依赖于 Map.keySet() 或 null 值进行初始化时比较有用。注意基本类型（int、boolean 等）是不能设置成 null 的。 | true \| false                                                | false                                                 |
| returnInstanceForEmptyRow        | 当返回行的所有列都是空时，MyBatis默认返回 `null`。 当开启这个设置时，MyBatis会返回一个空实例。 请注意，它也适用于嵌套的结果集（如集合或关联）。（新增于 3.4.2） | true \| false                                                | false                                                 |
| logPrefix                        | 指定 MyBatis 增加到日志名称的前缀。                          | 任何字符串                                                   | 未设置                                                |
| logImpl                          | 指定 MyBatis 所用日志的具体实现，未指定时将自动查找。        | SLF4J \| LOG4J \| LOG4J2 \| JDK_LOGGING \| COMMONS_LOGGING \| STDOUT_LOGGING \| NO_LOGGING | 未设置                                                |
| proxyFactory                     | 指定 Mybatis 创建可延迟加载对象所用到的代理工具。            | CGLIB \| JAVASSIST                                           | JAVASSIST （MyBatis 3.3 以上）                        |
| vfsImpl                          | 指定 VFS 的实现                                              | 自定义 VFS 的实现的类全限定名，以逗号分隔。                  | 未设置                                                |
| useActualParamName               | 允许使用方法签名中的名称作为语句参数名称。 为了使用该特性，你的项目必须采用 Java 8 编译，并且加上 `-parameters` 选项。（新增于 3.4.1） | true \| false                                                | true                                                  |
| configurationFactory             | 指定一个提供 `Configuration` 实例的类。 这个被返回的 Configuration 实例用来加载被反序列化对象的延迟加载属性值。 这个类必须包含一个签名为`static Configuration getConfiguration()` 的方法。（新增于 3.2.3） | 一个类型别名或完全限定类名。                                 | 未设置                                                |

一个配置完整的 settings 元素的示例如下：

```
<settings>
  <setting name="cacheEnabled" value="true"/>
  <setting name="lazyLoadingEnabled" value="true"/>
  <setting name="multipleResultSetsEnabled" value="true"/>
  <setting name="useColumnLabel" value="true"/>
  <setting name="useGeneratedKeys" value="false"/>
  <setting name="autoMappingBehavior" value="PARTIAL"/>
  <setting name="autoMappingUnknownColumnBehavior" value="WARNING"/>
  <setting name="defaultExecutorType" value="SIMPLE"/>
  <setting name="defaultStatementTimeout" value="25"/>
  <setting name="defaultFetchSize" value="100"/>
  <setting name="safeRowBoundsEnabled" value="false"/>
  <setting name="mapUnderscoreToCamelCase" value="false"/>
  <setting name="localCacheScope" value="SESSION"/>
  <setting name="jdbcTypeForNull" value="OTHER"/>
  <setting name="lazyLoadTriggerMethods" value="equals,clone,hashCode,toString"/>
</settings>
```

### 4. typeAliases（类型别名）

类型别名可为 Java 类型设置一个缩写名字。 它仅用于 XML 配置，意在降低冗余的全限定类名书写。例如：

```xml
<typeAliases>
  <typeAlias alias="Author" type="com.ecifics.pojo.Author"/>
  <typeAlias alias="Blog" type="com.ecifics.pojo.Blog"/>
  <typeAlias alias="Comment" type="com.ecifics.pojo.Comment"/>
  <typeAlias alias="Post" type="com.ecifics.pojo.Post"/>
  <typeAlias alias="Section" type="com.ecifics.pojo.Section"/>
  <typeAlias alias="Tag" type="com.ecifics.pojo.Tag"/>
</typeAliases>
```
也可以指定一个包名，MyBatis 会在包名下面搜索需要的 Java Bean，比如：
```xml
<typeAliases>
  <package name="com.ecifics.pojo"/>
</typeAliases>
```
每一个在包 com.ecifics.pojo 中的 Java Bean，在没有注解的情况下，会使用 Bean 的首字母小写的非限定类名来作为它的别名。 比如 com.ecifics.pojo.Author 的别名为 author

若有注解，则别名为其注解值，例如下面这个例子
```java
@Alias("author")
public class Author {
    ...
}
```

下面是一些为常见的 Java 类型内建的类型别名。它们都是不区分大小写的，注意，为了应对原始类型的命名重复，采取了特殊的命名风格。

| 别名       | 映射的类型 |
| :--------- | :--------- |
| _byte      | byte       |
| _long      | long       |
| _short     | short      |
| _int       | int        |
| _integer   | int        |
| _double    | double     |
| _float     | float      |
| _boolean   | boolean    |
| string     | String     |
| byte       | Byte       |
| long       | Long       |
| short      | Short      |
| int        | Integer    |
| integer    | Integer    |
| double     | Double     |
| float      | Float      |
| boolean    | Boolean    |
| date       | Date       |
| decimal    | BigDecimal |
| bigdecimal | BigDecimal |
| object     | Object     |
| map        | Map        |
| hashmap    | HashMap    |
| list       | List       |
| arraylist  | ArrayList  |
| collection | Collection |
| iterator   | Iterator   |

### 5. typeHandlers（类型处理器）

### 6. objectFactory（对象工厂）

### 7. plugins（插件）

### 8. environments（环境配置）
MyBatis 可以配置成适应多种环境，这种机制有助于将 SQL 映射应用于多种数据库之中， 现实情况下有多种理由需要这么做。例如，开发、测试和生产环境需要有不同的配置；或者想在具有相同 Schema 的多个生产数据库中使用相同的 SQL 映射。还有许多类似的使用场景。

**不过要记住：尽管可以配置多个环境，但每个 SqlSessionFactory 实例只能选择一种环境。**

所以，如果你想连接两个数据库，就需要创建两个 SqlSessionFactory 实例，每个数据库对应一个。而如果是三个数据库，就需要三个实例，依此类推，记起来很简单：

**每个数据库对应一个 SqlSessionFactory 实例**
为了指定创建哪种环境，只要将它作为可选的参数传递给 SqlSessionFactoryBuilder 即可。可以接受环境配置的两个方法签名是：
```java
SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(mybaitsConfigXmlFile, environment);
SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(mybaitsConfigXmlFile, environment, properties); 
```
如果忽略了环境参数，那么将会加载**默认环境**，如下所示：
```java
SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(mybaitsConfigXmlFile);
SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(mybaitsConfigXmlFile, properties);
```
environments 元素定义了如何配置环境。
```xml
<environments default="development">
  <environment id="development">
    <transactionManager type="JDBC">
      <property name="..." value="..."/>
    </transactionManager>
    <dataSource type="POOLED">
      <property name="driver" value="${driver}"/>
      <property name="url" value="${url}"/>
      <property name="username" value="${username}"/>
      <property name="password" value="${password}"/>
    </dataSource>
  </environment>
</environments>
```
注意一些关键点:
+ 默认使用的环境 ID（比如：default="development"）。
+ 每个 environment 元素定义的环境 ID（比如：id="development"）。
+ 事务管理器（transactionManager）的配置（比如：type="JDBC"）。
+ 数据源（dataSource）的配置（比如：type="POOLED"）。
  
#### transactionManager（事务管理器）
在 MyBatis 中有两种类型的事务管理器（也就是 type="[JDBC|MANAGED]"）：

+ JDBC – 这个配置直接使用了 JDBC 的提交和回滚设施，它依赖从数据源获得的连接来管理事务作用域。
+ MANAGED – 这个配置几乎没做什么。它从不提交或回滚一个连接，而是让容器（例如Spring）来管理事务的整个生命周期（比如 JEE 应用服务器的上下文）。 默认情况下它会关闭连接。然而一些容器并不希望连接被关闭，因此需要将 closeConnection 属性设置为 false 来阻止默认的关闭行为。例如:
```xml
<transactionManager type="MANAGED">
  <property name="closeConnection" value="false"/>
</transactionManager>
```
> 提示 
>
> 如果你正在使用 Spring + MyBatis，则没有必要配置事务管理器，因为 Spring 模块会使用自带的管理器来覆盖前面的配置。

#### dataSource（数据源）
dataSource 元素使用标准的 JDBC 数据源接口来配置 JDBC 连接对象的资源。
+ 大多数 MyBatis 应用程序会按示例中的例子来配置数据源。虽然数据源配置是可选的，但如果要启用延迟加载特性，就必须配置数据源。

有三种内建的数据源类型（也就是 type="[UNPOOLED|POOLED|JNDI]"）：
**UNPOOLED**– 这个数据源的实现会每次请求时打开和关闭连接。

运行步骤：

- TCP建立连接的三次握手（JDBC与MySQL的连接基于TCP协议）
- MySQL认证的三次握手
- 真正的SQL执行
- MySQL的关闭
- TCP的四次握手关闭

可以看到，为了执行一条SQL，却多了非常多我们不关心的网络交互。

虽然有点慢，但对那些数据库连接可用性要求不高的简单应用程序来说，是一个很好的选择。 性能表现则依赖于使用的数据库，对某些数据库来说，使用连接池并不重要，这个配置就很适合这种情形。UNPOOLED 类型的数据源仅仅需要配置以下 5 种属性：

+ `driver` – 这是 JDBC 驱动的 Java 类全限定名（并不是 JDBC 驱动中可能包含的数据源类）。
+ `url` – 这是数据库的 JDBC URL 地址。
+ `username` – 登录数据库的用户名。
+ `password` – 登录数据库的密码。
+ `defaultTransactionIsolationLevel` – 默认的连接事务隔离级别。
+ `defaultNetworkTimeout `– 等待数据库操作完成的默认网络超时时间（单位：毫秒）。查看 java.sql.Connection#setNetworkTimeout() 的 API 文档以获取更多信息。

作为可选项，你也可以传递属性给数据库驱动。只需在属性名加上“driver.”前缀即可，例如：
+ `driver.encoding=UTF8`
这将通过 DriverManager.getConnection(url, driverProperties) 方法传递值为 UTF8 的 encoding 属性给数据库驱动。

**POOLED**– 这种数据源的实现利用“池”的概念将 JDBC 连接对象组织起来，避免了创建新的连接实例时所必需的初始化和认证时间。 这种处理方式很流行，能使并发 Web 应用快速响应请求。

**第一次访问的时候，需要建立连接。 但是之后的访问，均会复用之前创建的连接，直接执行SQL语句。**

除了上述提到 UNPOOLED 下的属性外，还有更多属性用来配置 POOLED 的数据源：

+ `poolMaximumActiveConnections` – 在任意时间可存在的活动（正在使用）连接数量，默认值：10
+ `poolMaximumIdleConnections` – 任意时间可能存在的空闲连接数。
+ `poolMaximumCheckoutTime` – 在被强制返回之前，池中连接被检出（checked out）时间，默认值：20000 毫秒（即 20 秒）
+ `poolTimeToWait` – 这是一个底层设置，如果获取连接花费了相当长的时间，连接池会打印状态日志并重新尝试获取一个连接（避免在误配置的情况下一直失败且不打印日志），默认值：20000 毫秒（即 20 秒）。
+ `poolMaximumLocalBadConnectionTolerance` – 这是一个关于坏连接容忍度的底层设置， 作用于每一个尝试从缓存池获取连接的线程。 如果这个线程获取到的是一个坏的连接，那么这个数据源允许这个线程尝试重新获取一个新的连接，但是这个重新尝试的次数不应该超过 poolMaximumIdleConnections 与 poolMaximumLocalBadConnectionTolerance 之和。 默认值：3（新增于 3.4.5）
+ `poolPingQuery` – 发送到数据库的侦测查询，用来检验连接是否正常工作并准备接受请求。默认是“NO PING QUERY SET”，这会导致多数数据库驱动出错时返回恰当的错误消息。
+ `poolPingEnabled` – 是否启用侦测查询。若开启，需要设置 poolPingQuery 属性为一个可执行的 SQL 语句（最好是一个速度非常快的 SQL 语句），默认值：false。
+ `poolPingConnectionsNotUsedFor` – 配置 poolPingQuery 的频率。可以被设置为和数据库连接超时时间一样，来避免不必要的侦测，默认值：0（即所有连接每一时刻都被侦测 — 当然仅当 poolPingEnabled 为 true 时适用）。

**JNDI** – 这个数据源实现是为了能在如 EJB 或应用服务器这类容器中使用，容器可以集中或在外部配置数据源，然后放置一个 JNDI 上下文的数据源引用。这种数据源配置只需要两个属性：

+ `initial_context` – 这个属性用来在 InitialContext 中寻找上下文（即，initialContext.lookup(initial_context)）。这是个可选属性，如果忽略，那么将会直接从 InitialContext 中寻找 data_source 属性。
+ `data_source` – 这是引用数据源实例位置的上下文路径。提供了 initial_context 配置时会在其返回的上下文中进行查找，没有提供时则直接在 InitialContext 中查找。

和其他数据源配置类似，可以通过添加前缀“env.”直接把属性传递给 InitialContext。比如：

+ `env.encoding=UTF8`

### 9. databaseIdProvider（数据库厂商标识）

### 10. mappers（映射器）

```xml
<!-- 使用相对于类路径的资源引用 -->
<mappers>
  <mapper resource="org/mybatis/builder/AuthorMapper.xml"/>
  <mapper resource="org/mybatis/builder/BlogMapper.xml"/>
  <mapper resource="org/mybatis/builder/PostMapper.xml"/>
</mappers>

<!-- 使用完全限定资源定位符（URL） -->
<mappers>
  <mapper url="file:///var/mappers/AuthorMapper.xml"/>
  <mapper url="file:///var/mappers/BlogMapper.xml"/>
  <mapper url="file:///var/mappers/PostMapper.xml"/>
</mappers>

<!-- 使用映射器接口实现类的完全限定类名 -->
<mappers>
  <mapper class="org.mybatis.builder.AuthorMapper"/>
  <mapper class="org.mybatis.builder.BlogMapper"/>
  <mapper class="org.mybatis.builder.PostMapper"/>
</mappers>

<!-- 将包内的映射器接口实现全部注册为映射器  具体注意点看下方-->
<mappers>
  <package name="org.mybatis.builder"/>
</mappers>
```

这些配置会告诉 MyBatis 去哪里找映射文件，剩下的细节就应该是每个 SQL 映射文件了

**注意**

如果要使用`<package>`的方式映入mapper对应的包的时候，需要保证两点
+ mapper所在的包需要和映射文件所在的包一致
+ mapper接口需要和映射文件的名字一致 



## Mybatis获取参数值的两种方式(掌握第5中和第六种方式)
### 1. 两种方式
+ `${}`，本质是字符串拼接（有可能发生SQL注入）
+ `#{}`，本质是占位符赋值

### 2. 当mapper接口方法的参数为单个的字面量类型
可以通过`${}`和`#{}`以任意的字符串获取参数值，但是`${}`记得在两边加上单引号

#### 示例代码
UserMapper.java

```java
public interface UserMapper {
    /**
     * 根据用户名查询用户信息
     * @return User
     * @param username
     */
    User getUserByUsername(String username);
}
```


UserMapper.xml

+ `${}`方式（注意${}两边的单引号，大括号内的值可以使任意字符串，不一定要和UserMapper接口中的参数名字一样，例如可以把下面的username替换成aaa一样可以成功）
```xml
<select id="getUserByUsername" resultType="com.ecifics.mybatis.pojo.User">
    SELECT *
    FROM mybatis.t_user
    WHERE username = '${username}';
</select>
```
+ `#{}`方式（大括号内的值可以使任意字符串，不一定要和UserMapper接口中的参数名字一样，例如可以把下面的username替换成aaa一样可以成功）
```java
<select id="getUserByUsername" resultType="com.ecifics.mybatis.pojo.User">
    SELECT *
    FROM mybatis.t_user
    WHERE username = #{username};
</select>
```

#### 测试代码
```java
@Test
public void testGetUserByUsername() {
    SqlSession sqlSession = SqlSessionUtils.getSqlSession();

    UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
    User user = userMapper.getUserByUsername("Ecifics");

    System.out.println(user);
}
```

### 3. 当mapper接口方法的参数多个时
此时mybatis会将这些参数放在一个map集合中，以两种方式存储
+ 以arg0, arg1... 为键， 以传入的参数为值
+ 以param1, param2.... 为键，以传入的参数位置
	因此只需要通过`#{}` 和 `${}`以键的方式访问即可，但需要主义${}的单引号问题
	**tips:**
	arg和param时成对应关系，也就是说arg0和param1对应的值相同，arg1和param2对应的值相同，在使用过程中可以混用

#### 示例代码
UserMapper.java
```java
public interface UserMapper {
    /**
     * 验证登录
     * @param username
     * @param password
     * @return User
     */
    User checkLogin(String username, String password);
}
```

UserMapper.xml
+ `${}`方式
```xml
<select id="checkLogin" resultType="com.ecifics.mybatis.pojo.User">
    SELECT *
    FROM mybatis.t_user
    WHERE username = '${param1}' AND password = '${param2}';
</select>
```
或者
```xml
<select id="checkLogin" resultType="com.ecifics.mybatis.pojo.User">
    SELECT *
    FROM mybatis.t_user
    WHERE username = '${arg0}' AND password = '${arg1}';
</select>
```
+ `#{}`方式
```xml
<select id="checkLogin" resultType="com.ecifics.mybatis.pojo.User">
    SELECT *
    FROM mybatis.t_user
    WHERE username = #{arg0} AND password = #{arg1};
</select>
```
或者
```xml
<select id="checkLogin" resultType="com.ecifics.mybatis.pojo.User">
    SELECT *
    FROM mybatis.t_user
    WHERE username = #{param1} AND password = #{param2};
</select>
```

#### 测试代码
```java
@Test
public void testCheckLogin() {
    SqlSession sqlSession = SqlSessionUtils.getSqlSession();

    UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
    User user = userMapper.checkLogin("Ecifics", "131352");

    System.out.println(user);
}
```
### 4. 当mapper接口方法的参数多个时，可以手动将这些参数放在一个map集合中存储
只需要通过`#{}`和`${}`两种方式访问自定义map中的键即可，但仍需要主义`${}`的单引号问题

#### 示例代码
UserMapper.java
```java
public interface UserMapper {
    /**
     * 验证登录（参数为map）
     * @param map
     * @return user
     */
    User checkLoginByMap(Map<String, Object> map);
}
```

UserMapper.xml
+ `${}`方式
```xml
<select id="checkLoginByMap" resultType="com.ecifics.mybatis.pojo.User">
    SELECT *
    FROM mybatis.t_user
    WHERE username = '${username}' AND password = '${password}';
</select>
```

+ `#{}`方式
```xml
<select id="checkLoginByMap" resultType="com.ecifics.mybatis.pojo.User">
        SELECT *
        FROM mybatis.t_user
        WHERE username = #{username} AND password = #{password};
    </select>
```

#### 测试代码
```java
@Test
public void testCheckLoginByMap() {
    SqlSession sqlSession = SqlSessionUtils.getSqlSession();

    UserMapper userMapper = sqlSession.getMapper(UserMapper.class);

    Map<String, Object> map = new HashMap<>();
    map.put("username", "Ecifics");
    map.put("password", "131352");
    User user = userMapper.checkLoginByMap(map);

    System.out.println(user);
}
```
### 5. 当mapper接口方法的参数是实体类型的参数
只需要通过`#{}`和`${}`两种方式访问自定义实体类中的属性值（属性值不是成员变量，而是getter和setter方法中get或者set后面跟着那串，例如getName()那么name是属性值）即可，但仍需要主义`${}`的单引号问题

#### 示例代码 （${}方式省略）
UserMapper.java
```java
public interface UserMapper {
    /**
     * 添加用户
     * @param user
     * @return int
     */
    int insertUser(User user);
}
```

UserMapper.xml
```xml
<insert id="insertUser">
    INSERT INTO mybatis.t_user(id, username, password, age, sex, email)
    VALUES(null, #{username}, #{password}, #{age}, #{sex}, #{email});
</insert>
```
#### 测试代码
```java
@Test
public void testInsertUser() {
    SqlSession sqlSession = SqlSessionUtils.getSqlSession();

    User user = new User();
    user.setUsername("jack");
    user.setPassword("123456");
    user.setAge(21);
    user.setSex("男");
    user.setEmail("315523@qq.com");

    UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
    userMapper.insertUser(user);
}
```

### 6. 使用注解@Param命名参数
此时Mybatis会将参数放在一个map集合中，以两种方式进行存储
+ 以@Param注解中设定的值为键，参数为值
+ 以param1, param2...为键，参数为值

只需要使用`${}`和`#{}`通过相应的键访问其对应的值，`${}`需要注意单引号的问题

#### 示例代码
UserMapper.java
```java
public interface UserMapper {
    /**
     * 通过@param注解的方式来验证登录
     * @param username
     * @param password
     * @return user
     */
    User checkLoginByParam(@Param("username") String username, @Param("password") String password);
}
```

UserMapper.xml
```xml
<select id="checkLoginByParam" resultType="com.ecifics.mybatis.pojo.User">
    SELECT *
    FROM mybatis.t_user
    WHERE username = #{username} AND password = #{password};
</select>
```
或者
```xml
<select id="checkLoginByParam" resultType="com.ecifics.mybatis.pojo.User">
    SELECT *
    FROM mybatis.t_user
    WHERE username = #{param1} AND password = #{param2};
</select>
```

#### 测试代码
```java
@Test
public void testCheckLoginByParam() {
    SqlSession sqlSession = SqlSessionUtils.getSqlSession();

    UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
    User user = userMapper.checkLoginByParam("Ecifics", "131352");

    System.out.println(user);
}
```

## Mybatis各种查询功能
### 1. 查询处的数据只有一条
+ 可以通过实体类对象接收
+ 可以通过list集合接收
+ 可以通过map集合接收

### 2. 查询出的数据有多条
+ 可以通过实体类型的list集合接收 
+ 可以通过map类型的list集合接收
+ 可以在mapper接口方法上添加@MapKey注解，此时可以将查询到的每一条map集合作为值，@MapKey中指定的某个字段作为键，放在同一个map集合中（后面会详细演示）

### 3. 通过@MapKey注解的方式获取结果map集合
SelectMapper.java
这个例子中将User的id字段作为键，将查询到的用户信息存放的map作为值来返回
```java
public interface SelectMapper {
    /**
     * 查询所有用户信息并存入map集合
     * @return Map
     */
    @MapKey("id")
    Map<String, Object> getAllUserToMap();
}
```

SelectMapper.xml
```xml
<select id="getAllUserToMap" resultType="map">
    SELECT *
    FROM mybatis.t_user;
</select>
```

测试代码
```java
@Test
public void testGetAllUserToMap() {
    SqlSession sqlSession = SqlSessionUtils.getSqlSession();

    SelectMapper selectMapper = sqlSession.getMapper(SelectMapper.class);
    Map<String, Object> map = selectMapper.getAllUserToMap();

    System.out.println(map);
}
```



```xml
<select id="getUserByLike" resultType="com.ecifics.mybatis.pojo.User">
    SELECT *
    FROM mybatis.t_user
    WHERE username LIKE '%${username}%';
</select>
```



## Mybatis处理模糊查询
模糊查询使用`LIKE`关键字进行查询，其中在拼接`LIKE`关键值查询的字段中，如果有通配符（例如`%`）那么需要拼接，一共有三种方式
+ 一种是直接使用${}的方式
+ 使用#{}的方式，但需要使用Concat()函数来拼接
+ 使用`<bind>`方式来将拼接结果绑定到另外一个字符串上，再使用#{}的方式获取新字符串的值

### 示例代码
SQLMapper.java
```java
public interface SQLMapper {

    /**
     * 根据用户名模糊查询用户
     * @param username
     * @return list of user
     */
    List<User> getUserByLike(@Param("username") String username);
}
```

SQLMapper.xml （对应上面三种方式）
```xml
<select id="getUserByLike" resultType="com.ecifics.mybatis.pojo.User">
    SELECT *
    FROM mybatis.t_user
    WHERE username LIKE '%${username}%';
</select>
```

```xml
<select id="getUserByLike" resultType="com.ecifics.mybatis.pojo.User">
    SELECT *
    FROM mybatis.t_user
    WHERE username LIKE Concat('%', #{username}, '%');
</select>
```

```xml
<select id="getUserByLike" resultType="com.ecifics.mybatis.pojo.User">
    <bind name="user_name" value="'%' + username + '%'"/>
    SELECT *
    FROM mybatis.t_user
    WHERE username LIKE #{user_name};
</select>
```

### 测试代码
```java
@Test
public void testGetUserByLike() {
    SqlSession sqlSession = SqlSessionUtils.getSqlSession();
    SQLMapper mapper = sqlSession.getMapper(SQLMapper.class);

    List<User> userList = mapper.getUserByLike("a");

    for (User user : userList) {
    System.out.println(user);
    }
}
```

## Mybatis批量删除
批量删除可以考虑使用`IN`关键字来实现，`IN`后面括号中的参数通过mapper接口以String类型的变量进行传入，SQL语句通过`${}`的方式获取其中的值

### 示例代码
SQLMapper.java
```java
public interface SQLMapper {
    /**
     * 批量删除
     * @param ids
     * @return Integer
     */
    Integer deleteBatch(@Param("ids") String ids);
}
```
SQLMapper.xml
```xml
<delete id="deleteBatch">
    DELETE FROM mybatis.t_user
    WHERE id IN (${ids});
</delete>
```

### 测试代码
```java
@Test
public void testDeleteBatch() {
    SqlSession sqlSession = SqlSessionUtils.getSqlSession();
    SQLMapper mapper = sqlSession.getMapper(SQLMapper.class);

    Integer integer = mapper.deleteBatch("1, 2, 3");

    System.out.println(integer);
}
```

## Mybatis动态设置表名
动态设置表名只需要在`FROM`关键值后面加上传入的表名（String类型的字符串）即可，这个时候需要使用${}来获取表名，如果使用#{}来获取表名，得到的是一个带有单引号的字符串，而不是单独的表名

### 示例代码
SQLMapper.java
```java
public interface SQLMapper {
	/**
     * 通过表明查询表中的数据
     * @param tableName
     * @return list of user
     */
    List<User> getUserByTableName(@Param("tableName") String tableName);
}
```

SQLMapper.xml
```xml
<select id="getUserByTableName" 	resultType="com.ecifics.mybatis.pojo.User">
    SELECT *
    FROM ${tableName};
</select>
```

### 测试代码
```java
@Test
public void testGetUserByTableName() {
    SqlSession sqlSession = SqlSessionUtils.getSqlSession();
    SQLMapper mapper = sqlSession.getMapper(SQLMapper.class);

    List<User> userList = mapper.getUserByTableName("t_user");

    System.out.println(userList);
}
```

## Mybatis获取添加功能自增的主键
为了实现自增的功能，需要在mapper.xml文件条件语句中配置两个参数`userGeneratedKeys`（设置为true）和`keyProperty`(填入自增的主键名称)

### 实例代码
SQLMapper.java
```java
public interface SQLMapper {
    /**
     * 添加勇士信息
     * @param user
     */
    void insertUser(User user);
}
```

SQLMapper.xml
```xml
<insert id="insertUser" useGeneratedKeys="true" keyProperty="id">
    INSERT INTO mybatis.t_user(id, username, password, age, sex, email)
    VALUES(null, #{username}, #{password}, #{age}, #{sex}, #{email});
</insert>
```

### 测试代码
```java
@Test
public void testInsertUser() {
    SqlSession sqlSession = SqlSessionUtils.getSqlSession();
    SQLMapper mapper = sqlSession.getMapper(SQLMapper.class);
    User user = new User(null, "jack", "123456", 43, "男", "234125@qq.com");
    mapper.insertUser(user);

    System.out.println(user);
}
```

## 自定义映射ResultMap
### 引入问题
因为数据库（采用下划线分隔字符）和java（采用驼峰命名法）的命名规则不同，因此会造成数据库中字段的名字和java实体类字段名不同，为此，可以在Mapper.xml中使用`AS`关键字，将数据库中的字段重命名为java实体类字段的名字

### 全局配置解决实体类字段名和数据库中列名不一致问题
可以通过配置mybatis-config.xml文件自动将数据库中的列名转换成驼峰命名法，只需要添加下面的配置即可
```xml
<settings>
    <setting name="mapUnderscoreToCamelCase" value="true"/>
</settings>
```

### 通过ResultMap解决字段名和属性名的映射关系
需要在mapper.xml文件中定义resultMap标签，如下所示
```xml
<resultMap id="employeeResultMap" type="com.ecifics.mybatis.pojo.Employee">
    <id property="eid" column="eid" />
    <result property="empName" column="emp_name" />
    <result property="age" column="age" />
    <result property="sex" column="sex" />
    <result property="email" column="email" />
</resultMap>

<select id="getAllEmployee" resultMap="employeeResultMap">
    SELECT *
    FROM mybatis.t_employee;
</select>
```
在resultMap标签中，id用来这个resultMap的唯一标识，type用来设置java实体类的类型，id子标签是指定数据库表主键的映射关系，result标签是指定除主键之外，其他属性的映射关系，其中property指定的是java实体类中的字段名，column指定的是数据库中其对应的列名

### 多对一映射
对于多对一，例如员工对部门就是多对一的关系，如果需要利用员工的id查询员工及其所属部门的信息，则此时需要在员工实体类中添加部门字段，mapper.xml文件一共有三种配置方法
#### 1. 方法一
```xml
<resultMap id="employeeAndDepartmentResultMapOne" type="com.ecifics.mybatis.pojo.Employee">
    <id property="eid" column="eid" />
    <result property="empName" column="emp_name" />
    <result property="age" column="aeg" />
    <result property="sex" column="sex" />
    <result property="email" column="email" />
    <result property="department.did" column="did" />
    <result property="department.deptName" column="dept_name" />
</resultMap>

<select id="getEmployeeAndDepartmentById" resultMap="employeeAndDepartmentResultMapOne">
    SELECT *
    FROM mybatis.t_employee LEFT JOIN mybatis.t_dept
    ON mybatis.t_employee.did = mybatis.t_dept.did
    WHERE mybatis.t_employee.eid = #{eid};
</select>
```

### 方法二
通过\<association\>标签完成Employee类中department字段的映射
```xml
<resultMap id="employeeAndDepartmentResultMapTwo" type="com.ecifics.mybatis.pojo.Employee">
    <id property="eid" column="eid" />
    <result property="empName" column="emp_name" />
    <result property="age" column="aeg" />
    <result property="sex" column="sex" />
    <result property="email" column="email" />
    <association property="department" javaType="com.ecifics.mybatis.pojo.Department">
        <id property="did" column="did" />
        <result property="deptName" column="dept_name" />
    </association>
</resultMap>

<select id="getEmployeeAndDepartmentById" resultMap="employeeAndDepartmentResultMapTwo">
    SELECT *
    FROM mybatis.t_employee LEFT JOIN mybatis.t_dept
    ON mybatis.t_employee.did = mybatis.t_dept.did
    WHERE mybatis.t_employee.eid = #{eid};
</select>
```

### 方法三
分步查询：

+ 先通过员工id查出员工
+ 在通过员工中的部门id查询部门信息



EmployeeMapper.java

```java
/**
     * 通过分布查询查询员工以及员工所属的部门，第一步查询员工信息
     * @param eid
     * @return employee
     */
Employee getEmployeeAndDepartmentByIdByStepOne(@Param("eid") Integer eid);
```

DepartmentMapper.java

```java
/**
	* 通过分布查询查询员工以及员工所属的部门，第二步，通过did查询员工所对应的部门信息
   	* @param did
    * @return employee
    */
Department getEmployeeAndDepartmentByIdByStepTwo(@Param("did") Integer did);
```



DepartmentMapper.xml

```xml
<resultMap id="getEmployeeAndDepartmentByIdMap" type="com.ecifics.mybatis.pojo.Department">
    <id property="did" column="did" />
    <result property="deptName" column="dept_name" />
</resultMap>

<select id="getEmployeeAndDepartmentByIdByStepTwo" resultMap="getEmployeeAndDepartmentByIdMap">
    SELECT *
    FROM mybatis.t_dept
    WHERE did = #{did};
</select>
```



EmployeeMapper.xml

```xml
<resultMap id="employeeAndDepartmentByStepResult" type="com.ecifics.mybatis.pojo.Employee">
    <id property="eid" column="eid" />
    <result property="empName" column="emp_name" />
    <result property="age" column="aeg" />
    <result property="sex" column="sex" />
    <result property="email" column="email" />
    <association property="department"
                 select="com.ecifics.mybatis.mapper.DepartmentMapper.getEmployeeAndDepartmentByIdByStepTwo"
                 column="did"
                 fetchType="eager"></association>
</resultMap>

<select id="getEmployeeAndDepartmentByIdByStepOne" resultMap="employeeAndDepartmentByStepResult">
    SELECT *
    FROM mybatis.t_employee
    WHERE eid = #{eid};
</select>
```

见尚硅谷Mybatis P44

## 延迟加载
也就是按需加载，分步查询可以实现延迟加载，例如上面方法三中，通过员工id查询员工信息以及其所在部门信息，通过两步进行查询，那么如果我只需要查询结果中员工的姓名，那么第二步操作也就不需要执行，但是如果我需要获取这个员工的部门信息，则需要两步都执行

例如上面面例子中，我们通过EmployeeMapper类中的`getEmployeeAndDepartmentByIdByStepOne()`获取到了Employee对象，而我们后面只是用了员工的id信息，例如输出员工的id(System.out.println(emp.getId()))，那么这样只会查询员工表，不会继续查询Department表将Employee对象中的Department对象查询出来，如果后面用到了Department类中的属性，那么在这个时候才会去查询Department表获取部门信息，而不是在一开始就将其查出来

设置延迟加载需要在mybatis-config.xml中<settings>标签中设置属性
```xml
<setting name="lazyLoadingEnabled" value="true"/>
```
设置了延迟加载的属性后，为了是延迟加载这个功能更加灵活，可以在mapper.xml中的\<association\>标签**内**设置属性fetchType为lazy（表示延迟加载）和eager（立即加载）来完成相应SQL语句的控制

## 一对多
和上面多对一相对的是一对多，如果要查询一个部门及其员工的信息，此时一个部门包含多个员工，部门和员工是一对多的关系，此时需要在实体类Department中添加员工List集合，然后在mapper.xml文件中的\<resultMap\>标签中添加\<collection\>员工集合对象的映射，在\<collection\>标签内有两个属性property（用来制定需要映射的实体类集合字段）和ofType（用于制定集合中的实体类类型，本例中是Employee实体类）

下面是查询员工**及**部门信息

```xml
<resultMap id="departmentAndEmployeeResultMap" type="com.ecifics.mybatis.pojo.Department">
    <id property="did" column="did" />
    <result property="deptName" column="dept_name" />
    <collection property="employeeList" ofType="com.ecifics.mybatis.pojo.Employee">
        <id property="eid" column="eid" />
        <result property="empName" column="emp_name" />
        <result property="age" column="aeg" />
        <result property="sex" column="sex" />
        <result property="email" column="email" />
    </collection>
</resultMap>

<select id="getDepartmentAndEmployee" resultMap="departmentAndEmployeeResultMap">
    SELECT *
    FROM mybatis.t_dept LEFT JOIN mybatis.t_employee
    ON Mybatis.t_dept.did = mybatis.t_employee.did
    WHERE mybatis.t_dept.did = #{did};
</select>
```

### 分步查询
第一步，查询部门信息，第二部，根据did查询员工信息
见尚硅谷Mybatis P47

## 动态SQL
根据特定条件动态拼接SQL语句的功能，为了解决拼接SQL语句字符串时的痛点

例如网购，设置筛选条件

### 1. 动态SQL之if标签

if标签可以根据标签中test属性所对应的表达式决定标签中的内容是否需要拼接到SQL中去
其中test属性中，如果要想使用`&&`判断字符，可以将其替换为`and`，例如下面示例代码中的表达式

#### 示例
如果想通过部门员工信息查询到某个或某些员工，可以考虑使用if标签
DynamicMapper.java
```java
public interface DynamicMapper {
    /**
     * 多条件查询
     * @param employee
     * @return list of employee
     */
    List<Employee> getEmployeeByCondition(Employee employee);
}
```

DynamicMapper.xml
```xml
<select id="getEmployeeByCondition" resultType="com.ecifics.mybatis.pojo.Employee">
    SELECT *
    FROM t_employee
    WHERE 1 = 1
    <if test="empName != null and empName != ''">
    AND emp_name = #{empName}
    </if>

    <if test="age != null">
    AND age = #{age}
    </if>

    <if test="sex != null and sex != ''">
    AND sex = #{sex}
    </if>

    <if test="email != null and email != ''">
    AND email = #{email}
    </if>
</select>
```
**注意**：为了防止第一个if判断语句为空，造成后面判断标签中的AND造成语法错误，因此可以在WHERE后面加上一个恒等式`1 = 1`

### 2. 动态SQL之where标签
where标签解决了if标签中AND和WHERE相遇造成语法错误的情况
where可以自动解决后面出现AND和OR关键字的情况，因此不用担心写在where标签的if标签中的SQL语句中的AND标签和Or标签会造成语法错误，where标签会根据情况自动去除多余的AND和OR
**注意**：where标签不能将拼接SQL语句后面的AND或OR去掉（也就是下面的这种情况），因此需要将AND和OR标签写在拼接SQL语句的前面

```xml
<where>
    <if test="empName != null and empName != ''">
        emp_name = #{empName} AND
    </if>

    <if test="age != null">
        age = #{age} AND
    </if>

    <if test="sex != null and sex != ''">
        sex = #{sex} AND
    </if>

    <if test="email != null and email != ''">
        email = #{email}
    </if>
</where>
```



上面if标签的例子中的mapper.xml文件可以改写成下面的形式
```xml
<select id="getEmployeeByCondition" resultType="com.ecifics.mybatis.pojo.Employee">
    SELECT *
    FROM t_employee
    <where>
        <if test="empName != null and empName != ''">
        AND emp_name = #{empName}
        </if>

        <if test="age != null">
        AND age = #{age}
        </if>

        <if test="sex != null and sex != ''">
        AND sex = #{sex}
        </if>

        <if test="email != null and email != ''">
        AND email = #{email}
        </if>
    </where>

</select>
```

### 3. 动态SQL之trim标签
trim标签可以解决上面where标签中将AND或OR写在每一个拼接SQL语句后面的情况
trim标签有四个属性
	+ prefix | suffix：在trim标签中的内容前面和后面添加指定的内容（例如where标签）
	+ prefixOverrides | suffixOverrides：将trim标签中内容前面和后面去掉指定的内容
若trim标签中没有内容，则trim标签没有任何效果

上述例子的xml文件可以改写为下面示例
```xml
<select id="getEmployeeByCondition" resultType="com.ecifics.mybatis.pojo.Employee">
    SELECT *
    FROM t_employee
    <trim prefix="where" suffixOverrides="and|or">
        <if test="empName != null and empName != ''">
            emp_name = #{empName} AND
        </if>

        <if test="age != null">
            age = #{age} AND
        </if>

        <if test="sex != null and sex != ''">
            sex = #{sex} AND
        </if>

        <if test="email != null and email != ''">
            email = #{email}
        </if>
    </trim>
</select>
```

### 4. 动态SQL值choose、when、otherwise
choose、when、otherwise，其中choose是其他两个标签的父标签，也就是说其他两个标签需要写在choose标签内，when相当于if 和 else if， 而otherwise相当于else

#### 示例代码
DynamicMapper.java
```java
public interface DynamicMapper {
    /**
     * 测试choose、when、otherwise
     * @param employee
     * @return list of employee
     */
    List<Employee> getEmployeeByChoose(Employee employee);
}
```

DynamicMapper.xml
```xml
<select id="getEmployeeByChoose" resultType="com.ecifics.mybatis.pojo.Employee">
    select *
    from t_employee
    <where>
        <choose>
            <when test="empName != null and empName != ''">
                emp_name = #{empName}
            </when>

            <when test="age != null">
                age = #{age}
            </when>

            <when test="sex != null and sex != ''">
                sex = #{sex}
            </when>

            <when test="email != null and email != ''">
                email = #{email}
            </when>

            <otherwise>
                did = 1
            </otherwise>
        </choose>
    </where>
</select>
```



### 5. 动态SQL之foreach标签

foreach的属性

+ collection：设置需要循环的集合和数组（在mapper接口中用`@Param`注解进行标注）
+ item：用户自己设置一个名字用于表示数组和集合中的每一个数据
+ separator：循环体之间的分隔符
+ open：表示foreach标签所有内容的开始符
+ close：表示foreach标签所有内容的结束符

#### 1. 如果我想根据一个eid数组批量删除员工的数据，则可以使用foreach标签

示例代码
DynamicMapper.java
```java
public interface DynamicMapper {
    /**
     * 通过数组实现批量删除
     * @return int
     */
    int batchDeleteByArray(@Param("eids") Integer[] eids);
}
```

DynamicMapper.xml

用`IN`关键字进行删除

```xml
<delete id="batchDeleteByArray">
    delete from t_employee
    where eid in
    <foreach collection="eids" item="eid" separator="," open='(' close=")">
        #{eid}
    </foreach>
</delete>
```

用`OR`关键字进行删除

```xml
<delete id="batchDeleteByArray">
    delete from t_employee
    where
    <foreach collection="eids" item="eid" separator="or">
        eid = #{eid}
    </foreach>
</delete>
```
#### 2. 实现批量添加员工

通过传入需要添加的员工信息list集合进行添加



示例代码

DynamicMapper.java

```java
public interface DynamicMapper {
    /**
     * 通过List集合实现批量添加员工信息
     * @param employeeList
     * @return batch
     */
    int batchInsertByArray(@Param("employeeList") List<Employee> employeeList);
}
```



DynamicMapper.xml

```xml
<update id="batchInsertByArray">
    insert into t_employee(eid, emp_name, age, sex, email, did)
    values
    <foreach collection="employeeList" item="employee" separator=",">
        (null, #{employee.empName}, #{employee.age}, #{employee.sex}, #{employee.email}, null)
    </foreach>
</update>
```
### 6. 动态SQL之sql
sql标签用于将常用的SQL片段单独放在一个标签内，当需要使用的时候，可以直接引用这样的sql片段，例如SELECT * FROM..语句中*代表的那些数据库字段可以单独拿出来，以后需要使用的时候，通过\<include\>标签中的refid引用对应的sql片段即可

实例代码
未修改前
```xml
<select id="getEmployeeByCondition" resultType="com.ecifics.mybatis.pojo.Employee">
    SELECT *
    FROM t_employee
    <trim prefix="where" suffixOverrides="and|or">
        <if test="empName != null and empName != ''">
            emp_name = #{empName} AND
        </if>

        <if test="age != null">
            age = #{age} AND
        </if>

        <if test="sex != null and sex != ''">
            sex = #{sex} AND
        </if>

        <if test="email != null and email != ''">
            email = #{email}
        </if>
    </trim>
</select>
```

修改后

```xml
<sql id="employeeColumn">
    eid, emp_name, age, sex, email, did
</sql>

<select id="getEmployeeByCondition" resultType="com.ecifics.mybatis.pojo.Employee">
    SELECT <include refid="employeeColumn" />
    FROM t_employee
    <trim prefix="where" suffixOverrides="and|or">
        <if test="empName != null and empName != ''">
            emp_name = #{empName} AND
        </if>

        <if test="age != null">
            age = #{age} AND
        </if>

        <if test="sex != null and sex != ''">
            sex = #{sex} AND
        </if>

        <if test="email != null and email != ''">
            email = #{email}
        </if>
    </trim>
</select>
```

## Mybatis缓存 （缓存只针对查询功能有效，缓存是将数据存在内存中）
### 1. Mybatis一级缓存
一级缓存是SqlSession级别的，通过**同一个**SqlSession查询的数据会被缓存，下次查询相同的数据，就会从缓存中直接获取，不会从数据库重新访问

#### 示例
通过员工id查询员工信息

CacheMapper.java
```java
public interface CacheMapper {

    /**
     * 根据员工id查询员工信息
     * @param eid
     * @return employee
     */
    Employee getEmployeeById(@Param("eid") Integer eid);
}
```

CacheMapper.xml
```xml
<select id="getEmployeeById" resultType="com.ecifics.mybatis.pojo.Employee">
    select *
    from t_employee
    where eid = #{eid}
</select>
```

测试程序
```java
@Test
public void testGetEmployeeById() {
    SqlSession sqlSession = SqlSessionUtils.getSqlSession();
    CacheMapper mapper = sqlSession.getMapper(CacheMapper.class);

    Employee employee1 = mapper.getEmployeeById(1);
    System.out.println(employee1);

    Employee employee2 = mapper.getEmployeeById(1);
    System.out.println(employee2);
}
```



测试输出结果

```
DEBUG 03-10 13:52:29,717 ==>  Preparing: select * from t_employee where eid = ?  (BaseJdbcLogger.java:137) 
DEBUG 03-10 13:52:29,762 ==> Parameters: 1(Integer)  (BaseJdbcLogger.java:137) 
DEBUG 03-10 13:52:29,794 <==      Total: 1  (BaseJdbcLogger.java:137) 
Employee{eid=1, empName='张三', age=12, sex='男', email=' 234252@qq.com', department=null}
Employee{eid=1, empName='张三', age=12, sex='男', email=' 234252@qq.com', department=null}
```

可以看出只输出了一次SQL，如果用相同的SqlSession创建两个mapper查询同一个员工对象，输出结果也和上面相同



#### 使一级缓存失效的情况
+ 不同的SqlSession对应不同的一级缓存
+ 同一个SqlSession但是查询条件不同
+ 同一个SqlSession两次查询期间执行了任意一次增删改
+ 同一个SqlSession两次查询期间手动清空了缓存（使用SqlSession的clearCache()方法）

### 2. Mybatis二级缓存

二级缓存是SqlSessionFactory级别，通过同一个SqlSessionFactory创建的SqlSession查询的结果会被缓存；此后若再次执行相同的查询语句，结果就会从缓存中获取

#### 二级缓存开启的条件
+ 在核心配置文件中，设置全局配置属性`cacheEnable="true"`，默认为true，故不需要设置
+ 在映射文件(mapper.xml)中设置标签<cache />
+ 二级缓存必须在SqlSession**关闭**或**提交**后才有效
+ 查询的数据转换的实体类型必须实现序列化的接口（Serializable）

#### 二级缓存失效的情况
两次查询期间执行了任意的增删改，会使一级和二级缓存同时失效



#### 二级缓存的相关配置

在mapper配置文件中添加的cache标签可以设置一些属性

+ `eviction`属性：缓存回收策略
    + LRU （Least Recently Used） - 最近最少使用：移除最长时间不被使用的对象
    + FIFO（First In First Out）- 先进先出：按对象进入缓存在顺序来移除它们
    + SOFT - 软引用：移除基于垃圾回收器状态和软引用规则的对象
    + WEAK - 弱引用：更积极地移除基于垃圾收集器状态和弱引用规则的对象
+ `flushInterval`属性：刷新间隔（清空二级缓存），单位毫秒
	默认情况时设置，也就是没有刷新间隔，缓存仅仅调用（增删改）语句时刷新
+ `size`属性：引用数目，正整数
	代表缓存最多可以存储多少个对象，太大容易导致内存溢出
+ `readOnly`属性：只读 true | false，默认是false
	+ true：只读缓存，会给所有调用者返回缓存对象的相同实例。因此这些对象不能被修改。这提供了很重要的性能优势
	+ false：读写缓存；会返回缓存对象的拷贝（通过序列化）。这会慢一些，但是安全

### 3. Mybatis缓存查询的顺序
+ 先查询二级缓存，因为二级缓存中可能会有其他程序已经查出来的数据，可以拿来直接使用
+ 如果二级缓存没有命中，再查询一级缓存
+ 如果一级缓存也没有命中，则查询数据库
+ SqlSession关闭之后，一级缓存中的数据会写入二级缓存

### Mybatis整合第三方缓存（二级缓存）Ehcache
### 1. 添加依赖

```xml
<!-- Mybatis EHCache整合包 -->
<dependency>
	<groupId>org.mybatis.caches</groupId>
	<artifactId>mybatis-ehcache</artifactId>
	<version>1.2.1</version>
</dependency>
<!-- slf4j日志门面的一个具体实现 -->
<dependency>
	<groupId>ch.qos.logback</groupId>
	<artifactId>logback-classic</artifactId>
	<version>1.2.3</version>
</dependency>
```

### 2. 各个jar包的功能

| jar包名称       | 作用                            |
| --------------- | ------------------------------- |
| mybatis-ehcache | Mybatis和EHCache的整合包        |
| ehcache         | EHCache核心包                   |
| slf4j-api       | SLF4J日志门面包                 |
| logback-classic | 支持SLF4J门面接口的一个具体实现 |

### 3. 创建EHCache的配置文件ehcache.xml

- 名字必须叫`ehcache.xml`

```xml
<?xml version="1.0" encoding="utf-8" ?>
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="../config/ehcache.xsd">
    <!-- 磁盘保存路径 -->
    <diskStore path="D:\atguigu\ehcache"/>
    <defaultCache
            maxElementsInMemory="1000"
            maxElementsOnDisk="10000000"
            eternal="false"
            overflowToDisk="true"
            timeToIdleSeconds="120"
            timeToLiveSeconds="120"
            diskExpiryThreadIntervalSeconds="120"
            memoryStoreEvictionPolicy="LRU">
    </defaultCache>
</ehcache>
```

### 4. 设置二级缓存的类型

- 在xxxMapper.xml文件中设置二级缓存类型

```xml
<cache type="org.mybatis.caches.ehcache.EhcacheCache"/>
```

### 5. 加入logback日志

- 存在SLF4J时，作为简易日志的log4j将失效，此时我们需要借助SLF4J的具体实现logback来打印日志。创建logback的配置文件`logback.xml`，名字固定，不可改变

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration debug="true">
    <!-- 指定日志输出的位置 -->
    <appender name="STDOUT"
              class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <!-- 日志输出的格式 -->
            <!-- 按照顺序分别是：时间、日志级别、线程名称、打印日志的类、日志主体内容、换行 -->
            <pattern>[%d{HH:mm:ss.SSS}] [%-5level] [%thread] [%logger] [%msg]%n</pattern>
        </encoder>
    </appender>
    <!-- 设置全局日志级别。日志级别按顺序分别是：DEBUG、INFO、WARN、ERROR -->
    <!-- 指定任何一个日志级别都只打印当前级别和后面级别的日志。 -->
    <root level="DEBUG">
        <!-- 指定打印日志的appender，这里通过“STDOUT”引用了前面配置的appender -->
        <appender-ref ref="STDOUT" />
    </root>
    <!-- 根据特殊需求指定局部日志级别 -->
    <logger name="com.atguigu.crowd.mapper" level="DEBUG"/>
</configuration>
```

### 6. EHCache配置文件说明

| 属性名                          | 是否必须 | 作用                                                         |
| ------------------------------- | -------- | ------------------------------------------------------------ |
| maxElementsInMemory             | 是       | 在内存中缓存的element的最大数目                              |
| maxElementsOnDisk               | 是       | 在磁盘上缓存的element的最大数目，若是0表示无穷大             |
| eternal                         | 是       | 设定缓存的elements是否永远不过期。 如果为true，则缓存的数据始终有效， 如果为false那么还要根据timeToIdleSeconds、timeToLiveSeconds判断 |
| overflowToDisk                  | 是       | 设定当内存缓存溢出的时候是否将过期的element缓存到磁盘上      |
| timeToIdleSeconds               | 否       | 当缓存在EhCache中的数据前后两次访问的时间超过timeToIdleSeconds的属性取值时， 这些数据便会删除，默认值是0,也就是可闲置时间无穷大 |
| timeToLiveSeconds               | 否       | 缓存element的有效生命期，默认是0.,也就是element存活时间无穷大 |
| diskSpoolBufferSizeMB           | 否       | DiskStore(磁盘缓存)的缓存区大小。默认是30MB。每个Cache都应该有自己的一个缓冲区 |
| diskPersistent                  | 否       | 在VM重启的时候是否启用磁盘保存EhCache中的数据，默认是false   |
| diskExpiryThreadIntervalSeconds | 否       | 磁盘缓存的清理线程运行间隔，默认是120秒。每个120s， 相应的线程会进行一次EhCache中数据的清理工作 |
| memoryStoreEvictionPolicy       | 否       | 当内存缓存达到最大，有新的element加入的时候， 移除缓存中element的策略。 默认是LRU（最近最少使用），可选的有LFU（最不常使用）和FIFO（先进先出 |

## MyBatis的逆向工程

- 正向工程：先创建Java实体类，由框架负责根据实体类生成数据库表。Hibernate是支持正向工程的
- 逆向工程：先创建数据库表，由框架负责根据数据库表，反向生成如下资源：  
 - Java实体类  
   - Mapper接口  
   - Mapper映射文件

### 创建逆向工程的步骤

#### 添加依赖和插件

```xml
<dependencies>
	<!-- MyBatis核心依赖包 -->
	<dependency>
		<groupId>org.mybatis</groupId>
		<artifactId>mybatis</artifactId>
		<version>3.5.9</version>
	</dependency>
	<!-- junit测试 -->
	<dependency>
		<groupId>junit</groupId>
		<artifactId>junit</artifactId>
		<version>4.13.2</version>
		<scope>test</scope>
	</dependency>
	<!-- MySQL驱动 -->
	<dependency>
		<groupId>mysql</groupId>
		<artifactId>mysql-connector-java</artifactId>
		<version>8.0.27</version>
	</dependency>
	<!-- log4j日志 -->
	<dependency>
		<groupId>log4j</groupId>
		<artifactId>log4j</artifactId>
		<version>1.2.17</version>
	</dependency>
</dependencies>
<!-- 控制Maven在构建过程中相关配置 -->
<build>
	<!-- 构建过程中用到的插件 -->
	<plugins>
		<!-- 具体插件，逆向工程的操作是以构建过程中插件形式出现的 -->
		<plugin>
			<groupId>org.mybatis.generator</groupId>
			<artifactId>mybatis-generator-maven-plugin</artifactId>
			<version>1.3.0</version>
			<!-- 插件的依赖 -->
			<dependencies>
				<!-- 逆向工程的核心依赖 -->
				<dependency>
					<groupId>org.mybatis.generator</groupId>
					<artifactId>mybatis-generator-core</artifactId>
					<version>1.3.2</version>
				</dependency>
				<!-- 数据库连接池 -->
				<dependency>
					<groupId>com.mchange</groupId>
					<artifactId>c3p0</artifactId>
					<version>0.9.2</version>
				</dependency>
				<!-- MySQL驱动 -->
				<dependency>
					<groupId>mysql</groupId>
					<artifactId>mysql-connector-java</artifactId>
					<version>8.0.27</version>
				</dependency>
			</dependencies>
		</plugin>
	</plugins>
</build>
```

#### 创建MyBatis的核心配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <properties resource="jdbc.properties"/>
    <typeAliases>
        <package name=""/>
    </typeAliases>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <package name=""/>
    </mappers>
</configuration>
```

#### 创建逆向工程的配置文件

- 文件名必须是：`generatorConfig.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
    <!--
    targetRuntime: 执行生成的逆向工程的版本
    MyBatis3Simple: 生成基本的CRUD（清新简洁版）
    MyBatis3: 生成带条件的CRUD（奢华尊享版）
    -->
    <context id="DB2Tables" targetRuntime="MyBatis3Simple">
        <!-- 数据库的连接信息 -->
        <jdbcConnection driverClass="com.mysql.cj.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/mybatis"
                        userId="root"
                        password="123456">
        </jdbcConnection>
        <!-- javaBean的生成策略-->
        <javaModelGenerator targetPackage="com.atguigu.mybatis.pojo" targetProject=".\src\main\java">
            <property name="enableSubPackages" value="true" />
            <property name="trimStrings" value="true" />
        </javaModelGenerator>
        <!-- SQL映射文件的生成策略 -->
        <sqlMapGenerator targetPackage="com.atguigu.mybatis.mapper"
                         targetProject=".\src\main\resources">
            <property name="enableSubPackages" value="true" />
        </sqlMapGenerator>
        <!-- Mapper接口的生成策略 -->
        <javaClientGenerator type="XMLMAPPER"
                             targetPackage="com.atguigu.mybatis.mapper" targetProject=".\src\main\java">
            <property name="enableSubPackages" value="true" />
        </javaClientGenerator>
        <!-- 逆向分析的表 -->
        <!-- tableName设置为*号，可以对应所有表，此时不写domainObjectName -->
        <!-- domainObjectName属性指定生成出来的实体类的类名 -->
        <table tableName="t_emp" domainObjectName="Emp"/>
        <table tableName="t_dept" domainObjectName="Dept"/>
    </context>
</generatorConfiguration>
```

#### 执行MBG插件的generate目标

- ![](D:/Browser Download/MyBatis/Resources/执行MBG插件的generate目标.png)
- 如果出现报错：`Exception getting JDBC Driver`，可能是pom.xml中，数据库驱动配置错误
 - dependency中的驱动![](D:/Browser Download/MyBatis/Resources/dependency中的驱动.png)
   - mybatis-generator-maven-plugin插件中的驱动![](D:/Browser Download/MyBatis/Resources/插件中的驱动.png)
   - 两者的驱动版本应该相同
- 执行结果![](D:/Browser Download/MyBatis/Resources/逆向执行结果.png)

## QBC

### 查询

- `selectByExample`：按条件查询，需要传入一个example对象或者null；如果传入一个null，则表示没有条件，也就是查询所有数据
- `example.createCriteria().xxx`：创建条件对象，通过andXXX方法为SQL添加查询添加，每个条件之间是and关系
- `example.or().xxx`：将之前添加的条件通过or拼接其他条件
  ![](D:/Browser Download/MyBatis/Resources/example的方法.png)

```java
@Test public void testMBG() throws IOException {
	InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
	SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
	SqlSessionFactory sqlSessionFactory = sqlSessionFactoryBuilder.build(is);
	SqlSession sqlSession = sqlSessionFactory.openSession(true);
	EmpMapper mapper = sqlSession.getMapper(EmpMapper.class);
	EmpExample example = new EmpExample();
	//名字为张三，且年龄大于等于20
	example.createCriteria().andEmpNameEqualTo("张三").andAgeGreaterThanOrEqualTo(20);
	//或者did不为空
	example.or().andDidIsNotNull();
	List<Emp> emps = mapper.selectByExample(example);
	emps.forEach(System.out::println);
}
```

![](D:/Browser Download/MyBatis/Resources/example测试结果.png)

#### 增改

- `updateByPrimaryKey`：通过主键进行数据修改，如果某一个值为null，也会将对应的字段改为null
 - `mapper.updateByPrimaryKey(new Emp(1,"admin",22,null,"456@qq.com",3));`
   - ![](D:/Browser Download/MyBatis/Resources/增删改测试结果1.png)
- `updateByPrimaryKeySelective()`：通过主键进行选择性数据修改，如果某个值为null，则不修改这个字段
 - `mapper.updateByPrimaryKeySelective(new Emp(2,"admin2",22,null,"456@qq.com",3));`
   - ![](D:/Browser Download/MyBatis/Resources/增删改测试结果2.png)

## 分页插件

### 分页插件使用步骤

#### 添加依赖

```xml
<!-- https://mvnrepository.com/artifact/com.github.pagehelper/pagehelper -->
<dependency>
	<groupId>com.github.pagehelper</groupId>
	<artifactId>pagehelper</artifactId>
	<version>5.2.0</version>
</dependency>
```

#### 配置分页插件

- 在MyBatis的核心配置文件（mybatis-config.xml）中配置插件
- ![](D:/Browser Download/MyBatis/Resources/配置分页插件.png)

```xml
<plugins>
	<!--设置分页插件-->
	<plugin interceptor="com.github.pagehelper.PageInterceptor"></plugin>
</plugins>
```

### 分页插件的使用

#### 开启分页功能

- 在查询功能之前使用`PageHelper.startPage(int pageNum, int pageSize)`开启分页功能
 - pageNum：当前页的页码  
   - pageSize：每页显示的条数

```java
@Test
public void testPageHelper() throws IOException {
	InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
	SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
	SqlSessionFactory sqlSessionFactory = sqlSessionFactoryBuilder.build(is);
	SqlSession sqlSession = sqlSessionFactory.openSession(true);
	EmpMapper mapper = sqlSession.getMapper(EmpMapper.class);
	//访问第一页，每页四条数据
	PageHelper.startPage(1,4);
	List<Emp> emps = mapper.selectByExample(null);
	emps.forEach(System.out::println);
}
```

![](D:/Browser Download/MyBatis/Resources/分页测试结果.png)

#### 分页相关数据

##### 方法一：直接输出

```java
@Test
public void testPageHelper() throws IOException {
	InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
	SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
	SqlSessionFactory sqlSessionFactory = sqlSessionFactoryBuilder.build(is);
	SqlSession sqlSession = sqlSessionFactory.openSession(true);
	EmpMapper mapper = sqlSession.getMapper(EmpMapper.class);
	//访问第一页，每页四条数据
	Page<Object> page = PageHelper.startPage(1, 4);
	List<Emp> emps = mapper.selectByExample(null);
	//在查询到List集合后，打印分页数据
	System.out.println(page);
}
```

- 分页相关数据：

	```
	Page{count=true, pageNum=1, pageSize=4, startRow=0, endRow=4, total=8, pages=2, reasonable=false, pageSizeZero=false}[Emp{eid=1, empName='admin', age=22, sex='男', email='456@qq.com', did=3}, Emp{eid=2, empName='admin2', age=22, sex='男', email='456@qq.com', did=3}, Emp{eid=3, empName='王五', age=12, sex='女', email='123@qq.com', did=3}, Emp{eid=4, empName='赵六', age=32, sex='男', email='123@qq.com', did=1}]
	```

##### 方法二使用PageInfo

- 在查询获取list集合之后，使用`PageInfo<T> pageInfo = new PageInfo<>(List<T> list, intnavigatePages)`获取分页相关数据
 - list：分页之后的数据  
   - navigatePages：导航分页的页码数

```java
@Test
public void testPageHelper() throws IOException {
	InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
	SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
	SqlSessionFactory sqlSessionFactory = sqlSessionFactoryBuilder.build(is);
	SqlSession sqlSession = sqlSessionFactory.openSession(true);
	EmpMapper mapper = sqlSession.getMapper(EmpMapper.class);
	PageHelper.startPage(1, 4);
	List<Emp> emps = mapper.selectByExample(null);
	PageInfo<Emp> page = new PageInfo<>(emps,5);
	System.out.println(page);
}
```

- 分页相关数据：

	```
	PageInfo{
	pageNum=1, pageSize=4, size=4, startRow=1, endRow=4, total=8, pages=2, 
	list=Page{count=true, pageNum=1, pageSize=4, startRow=0, endRow=4, total=8, pages=2, reasonable=false, pageSizeZero=false}[Emp{eid=1, empName='admin', age=22, sex='男', email='456@qq.com', did=3}, Emp{eid=2, empName='admin2', age=22, sex='男', email='456@qq.com', did=3}, Emp{eid=3, empName='王五', age=12, sex='女', email='123@qq.com', did=3}, Emp{eid=4, empName='赵六', age=32, sex='男', email='123@qq.com', did=1}], 
	prePage=0, nextPage=2, isFirstPage=true, isLastPage=false, hasPreviousPage=false, hasNextPage=true, navigatePages=5, navigateFirstPage=1, navigateLastPage=2, navigatepageNums=[1, 2]}
	```

- 其中list中的数据等同于方法一中直接输出的page数据

##### 常用数据：

- pageNum：当前页的页码  
- pageSize：每页显示的条数  
- size：当前页显示的真实条数  
- total：总记录数  
- pages：总页数  
- prePage：上一页的页码  
- nextPage：下一页的页码
- isFirstPage/isLastPage：是否为第一页/最后一页  
- hasPreviousPage/hasNextPage：是否存在上一页/下一页  
- navigatePages：导航分页的页码数  
- navigatepageNums：导航分页的页码，\[1,2,3,4,5]