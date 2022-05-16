# Spring

## 1. IoC控制反转
### 1.1 概念
IoC,Inversion of Control: 控制反转，指将传统上由程序代码直接操控的对象调用权交给容器，通过容器来实现对象的装配和管理。控制反转就是对对象控制权的转移，从程序代码本身反转到了外部容器。通过容器实现对象的创建，属性赋值，依赖的管理

### 1.2 IoC组成
+ 控制： 对象创建，属性赋值，对象生命周期管理
+ 反转：把开发人员管理对象的权限转移给了代码之外的容器实现。由容器完成对对象的管理
+ 正转（和反转相反）：开发人员在编写代码时，使用new构造方法创建对象。开发人员掌握了对象的创建，属性赋值，对象从开始到销毁的全部过程。开发人员有对对象的全部控制

通过容器，可以使用容器中的对象（容器已经创建了对象，对象属性已经赋值了，对象也组装好了），就像是以前自己在家吃饭需要买菜洗菜做菜，而现在可以通过外卖完成，买菜洗菜做菜就交给其他人来完成 

### 1.3 IoC的实现
依赖注入（DI， Dependency Injection）：程序只需要提供对象的名称就可以，对象如何创建，如何从容器中查找，获取都由容器内部自己实现。（举例，自己做菜变成了点外卖的过程）



## 2. Spring项目实现步骤

### 2.1 创建maven项目

### 2.2 加入依赖
+ spring-context：spring依赖
+ junit

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.3.16</version>
    </dependency>

    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.13.2</version>
        <scope>test</scope>
    </dependency>
</dependencies>

```
### 2.3 开发人员定义类：接口及其实现类或者一般的类
### 2.4 创建spring的配置文件（用于声明对象）
  + 把对象交给spring创建和管理
  + 使用`<bean>`表示对象声明，一个bean表示一个java对象

beans.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

</beans>
```
配置文件中<bean>标签属性
+ id：自定义对象名称，唯一值（可以不设置，spring可以提供默认名称）
+ class：类的全限定名称，spring通过反射机制创建对象，不能是接口

spring根据id和class创建对象，将对象放入spring的一个map对象
map.put(id, 对象)

### 2.5 使用容器中的对象
+ 创建一个spring容器对象`ApplicationContext`
+ 从容器中，根据名称获取对象，使用getBean("对象名称")

实例代码
```java
//1. 指定spring配置文件：从类路径（classpath）之下开始的路径（对应的是resources目录下）
String resouces = "beans.xml";

//2. 创建容器对象
ApplicationContext context = new ClassPathXmlApplicationContext(resouces);

//3. 从容器中获取指定名称的对象，使用getBean()方法
SomeService someService = (SomeService) context.getBean("someService");

someService.doSome();
```



### 2.6 spring容器创建对象的特点

+ **spring默认使用无参构造方法创建对象，因此如果定义了有参构造方法，同时也需要定义无参构造方法**

+ **创建spring容器对象的时候（获取ApplicationContext对象的时候），会读取配置文件，创建文件中的java对象（如果对象有无参构造方法，会执行无参构造方法），以及执行相关属性值的赋值**
+ **在创建容器对象的时候，会创建好配置文件中的所有对象，放在map对象中** 

## 2.7 DI
spring调用类的无参构造器创建对象，对象创建后给属性赋值
给属性赋值可以使用

+ xml配置文件中的标签和属性
+ 使用注解

DI的分类
+ set注入，也叫设值注入
+ 构造注入

### 2.7.1 基于xml的DI
xml配置文件中的标签和属性，完成对象创建，属性赋值
##### （1） 简单类型的set注入
spring调用类中的set方法，在set方法中可以完成属性赋值，推荐使用
spring调用类的set方法，通过set方法完成属性赋值

```xml
<bean id="xxx" class="yyy" >
	<property name="属性名" value="简单类型属性值" />
	...
</bean>
```
**注意**：这里的属性名是根据setter方法后面部分决定的，例如setName()方法可以推断属性名为name，而setStudentName()方法可以推断属性名为studentName，和定义的成员字段无关

示例代码

```xml
<bean id="mySchool" class="com.ecifics.spring.bean2.School">
    <property name="name" value="PKU" />
    <property name="address" value="BJ" />
</bean>
```



#### （2）引用类型的set注入

语法 

```xml
<bean id="xxx" class="yyy">
	<property name="属性名" ref="bean的id" />
    ...
</bean>
```

示例代码

```xml
<bean id="myStudent" class="com.ecifics.spring.bean2.Student">
    <property name="name" value="Ecifics" />
    <property name="age" value="21" />
    <property name="school" ref="mySchool" />
</bean>

<bean id="mySchool" class="com.ecifics.spring.bean2.School">
    <property name="name" value="PKU" />
    <property name="address" value="BJ" />
</bean>
```

#### （3）构造注入
spring调用类的有参数构造方法，创建对象的同时给属性赋值
语法：

```xml
<bean id="xxx" class="xxx">
	<constructor-arg name="构造方法形参名" index="构造方法参数位置" value="简单类型的形参值" ref="引用类型的形参值" />
</bean>
```



+ 使用name属性注入，示例

```xml
<bean id="myStudent1" class="com.ecifics.spring.bean3.Student">
    <constructor-arg name="name" value="Ecifics" />
    <constructor-arg name="age" value="12" />
    <constructor-arg name="school" ref="mySchool" />
</bean>

<bean id="mySchool" class="com.ecifics.spring.bean3.School">
    <property name="name" value="PKU" />
    <property name="address" value="BJ" />
</bean>
```

+ 使用index属性注入，示例

```xml
<bean id="myStudent2" class="com.ecifics.spring.bean3.Student">
    <constructor-arg index="0" value="Ecifics" />
    <constructor-arg index="1" value="20" />
    <constructor-arg index="2" ref="mySchool" />
</bean>

<bean id="mySchool" class="com.ecifics.spring.bean3.School">
    <property name="name" value="PKU" />
    <property name="address" value="BJ" />
</bean>
```



+ 省略index属性注入（但是参数顺序必须按照构造器顺序来写），示例

```xml
<bean id="myStudent3" class="com.ecifics.spring.bean3.Student">
    <constructor-arg value="Ecifics" />
    <constructor-arg value="20" />
    <constructor-arg ref="mySchool" />
</bean>

<bean id="mySchool" class="com.ecifics.spring.bean3.School">
    <property name="name" value="PKU" />
    <property name="address" value="BJ" />
</bean>
```

#### （4） 引用类型的自动注入
spring可以跟某些规则给引用类型完成赋值（**只对引用类型有效**）
+ byName
  java类型中引用类型属性名称和spring容器中bean的id名称一样，且数据类型也一样，这些bean能赋值给引用类型

语法

```xml
<bean id="xxx" class="yyy" autowire="byName">

</bean>
```



示例代码

```xml
<bean id="school" class="com.ecifics.spring.bean4.School" >
    <property name="name" value="PKU" />
    <property name="address" value="BJ" />
</bean>

<bean id="myStudent" class="com.ecifics.spring.bean4.Student" autowire="byName">
    <property name="name" value="Ecifics" />
    <property name="age" value="21" />
</bean>
```

**注意**：必须保证bean的id和目标setter方法中属性名相同，例如setter方法名为setSchool（即属性名为school），那么bean的id必须是school


+ byType
  java类中引用类型的数据类型和spring容器中的bean的class值是同源关系，这行的bean可以赋值个引用类型
  + 同源关系：
  
    + java中引用类型的数据类型和bean中class值是一样的
    + java中引用类型的数据类型和bean中class值是父子类关系
    + java中引用类型的数据类型和bean中class值是接口和实现类关系
  + **注意**：上述同源关系中符合条件的对象只能有一个，例如父子类关系，子类只能有一个，多于一个会报错

语法

```xml
<bean id="xxx" class="yyy" autowire="byType">
    
</bean>
```

示例代码

```xml
<bean id="myStudent" class="com.ecifics.spring.bean5.Student" autowire="byType">
    <property name="name" value="Ecifics" />
    <property name="age" value="21" />
</bean>

<bean id="mySchool" class="com.ecifics.spring.bean5.School" >
    <property name="name" value="PKU" />
    <property name="address" value="BJ" />
</bean>
```

#### （5） 为应用程序制定多个Spring配置文件
项目中分多个配置文件的方式

+ 按功能模块分，一个模块一个配置文件（例如将菜单功能分在一块，注册功能分在一块）
+ 按类的功能分，数据库相关操作dao层在一个配置文件，service层在一个配置文件



Spring管理多个配置文件：常用的是包含关系的配置文件。项目中有一个总的配置文件，里面有`<import>`标签包含其他的多个配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <import resource="其他配置文件的路径1" />
    <import resource="其他配置文件的路径2" />
    
</beans>
```

在一个文件中通过import标签引入其他配置文件的内容时，需要使用类路径（classpath）到目标文件的路径

注：classpath是指字节码文件夹target目录下的classes目录



## 2.8 基于注解的DI

使用Spring提供的注解完成对象的创建，属性的赋值



### 2.8.1 使用注解之前的准备

想要使用注解，必须现在spring配置文件中声明组件扫描器



applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="com.ecifics.spring.bean1" />
</beans>
```

其中base-package中填写使用注解的类所在的包的名称



### 2.8.2 定义Bean的注解@Component

`@Component`：调用对象的**无参构造方法**创建对象，并将对象放入容器中。相当于`<bean>`标签

​	其中有一个属性value，表名类对象的名称，相当于bean标签中的id属性（value可以省略）

​	将@Component放在类的上方

如果`@component`没有提供value属性的值，那么Spring会使用默认值，也就是将类名称第一个字母小写后的样子，例如类名为Student，Spring提供的默认名称为student



示例代码

Student.java

```java
@Component(value = "myStudent")
public class Student {

    private String name;

    private int age;

    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```



### 2.8.3 和@Component功能相同的创建对象的注解

+ `@Repository`注解，放在dao层接口的实现类上，用于创建dao层对象
+ `@Service`注解，放在service层接口的实现类上，用于创建service层对象
+ `@Controller`注解，用于放在controller层中实体类上，用于创建controller对象

以上三个注解和`@Component`注解都能创建对象，但是上面三个注解表示的对象是分层的，其创建的对象是属于不同层的，具有额外的功能，如果一个对象不属于dao、service和controller层，使用`@Component`创建对象更加合适



### 2.8.4 扫描多个包中注解的三种方式

#### （1）使用多次组件扫描器

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="com.ecifics.spring.bean1" />
    <context:component-scan base-package="com.ecifics.spring.bean2" />
    <context:component-scan base-package="com.ecifics.spring.bean3" />
</beans>
```



#### （2）使用分隔符（使用;或,），指定多个包

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="com.ecifics.spring.bean1;com.ecifics.spring.bean2;com.ecifics.spring.bean3" />
</beans>
```



#### （4）使用父包

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="com.ecifics.spring" />
</beans>
```



### 2.8.5 简单类型属性注入@Value

在实体类属性上使用注解`@Value`，该注解的value属性用于注入制定的值

使用该注解完成属性注入时，类中无需setter方法。

如果属性有setter方法，也可以将注解加载setter方法上



示例代码

```java
@Component
public class School {

    @Value("PKU")
    private String name;

    @Override
    public String toString() {
        return "School{" +
                "name='" + name + '\'' +
                '}';
    }
}
```

或者

```java
@Component
public class School {
    
    private String name;

    @Override
    public String toString() {
        return "School{" +
                "name='" + name + '\'' +
                '}';
    }

    @Value("THU")
    public void setName(String name) {
        this.name = name;
    }
}
```



### 2.8.6 @Value注解使用外部属性配置文件

创建properties文件

myconf.properties

```properties
myname="ZJU"
```



在applicationContext.xml文件中配置读取外部属性文件

```xml
<context:property-placeholder location="classpath:myconf.properties" />
```

location属性用于表示属性文件位置



java实体类文件中使用`@Value`注解引用外部文件的语法格式为

```java
@Value("${key}")
```



School.java

```java
@Component
public class School {

    @Value("${myname}")
    private String name;

    @Override
    public String toString() {
        return "School{" +
                "name='" + name + '\'' +
                '}';
    }
}
```



### 2.8.7 @Autowired自动注入byType

`@Autowired`注解用于给**引用类型**赋值，使用自动注入原理，支持byName和byType，默认是byType

使用该注解完成属性注入时，类中无需setter方法；如果有setter方法，也可以直接放在setter方法上



示例代码

Student.java

```java
@Component
public class Student {

    @Value("Ecifics")
    private String name;

    @Value("23")
    private int age;

    @Autowired
    private School school;

    public Student() {
        System.out.println("Student方法的无参构造");
    }

    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", school=" + school +
                '}';
    }
}
```



School.java

```java
@Component("mySchool")
public class School {

    @Value("SWPU")
    private String name;

    @Value("CD")
    private String address;

    @Override
    public String toString() {
        return "School{" +
                "name='" + name + '\'' +
                ", address='" + address + '\'' +
                '}';
    }
}
```



### 2.8.8 @Autowired自动注入byName

通过byName的方式对属性进行赋值需要使用`@Autowired`和`Qualifier`两个注解。

`Qualifier`的value属性用于指定要匹配的Bean的id值



示例代码

Student.java

```java
@Component
public class Student {

    @Value("Ecifics")
    private String name;

    @Value("23")
    private int age;

    @Autowired
    @Qualifier("mySchool")
    private School school;

    public Student() {
        System.out.println("Student方法的无参构造");
    }

    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", school=" + school +
                '}';
    }
}
```



School.java

```java
@Component("mySchool")
public class School {

    @Value("SCU")
    private String name;

    @Value("CD")
    private String address;

    @Override
    public String toString() {
        return "School{" +
                "name='" + name + '\'' +
                ", address='" + address + '\'' +
                '}';
    }
}
```



### 2.8.9 @Autowired注解的required属性

这个属性是boolean类型，默认为true

+ true：Spring在启动的时候，创建容器对象时候，会检查应用类型是否复制成功。如果赋值失败，终止程序执行，并报错
+ false：Spring在启动的时候，创建容器对象时候，会检查应用类型是否复制成功。如果赋值失败，程序正常执行，不报错，引用类型的值为null



### 2.8.10 JDK注解@Resource自动注入

Spring提供了对jdk中`@Resource`注解的支持

`@Resource`注解既可以按名称（byName）匹配Bean，也可以按类型（byType）库匹配Bean，**默认是按名称注入**

`@Resource`注解首先使用byName进行匹配，如果失败会使用byType进行匹配

如果只是用byName进行注入，可以使用`@Resource`注解的name属性，格式为

```java
@Resource(name = "bean的id")
```



#### 

**注意**，jdk1.8带有`@Resource`注解，高于jdk1.8的没有这个注解，需要导入依赖`javax.annotation-api`



## 3. AOP 面向切面编程

### 3.1 不适用AOP进行开发

例如业务层有一个方法doSome()处理用户登录信息，如果此时想在这个部门添加日志功能，只能在doSome()方法内部进行添加代码的操作，同理如果还需要其他工作，也需要在doSome()中添加工作，导致源代码改动较多，如果所有方法都需要添加日志功能，那么会造成重复代码过多，代码难于维护

当然，对于上述问题，也可以将重复的代码添加到一个处理类中，降低重复代码数量



### 3.2  AOP概念

#### 3.2.1 什么是AOP

AOP （Aspect Oriented Programming）：面向切面编程

Aspect：切面，给业务方法增加的的功能，叫做切面。切面一般是非业务功能，而切面功能一般是可以复用的，例如日志功能，事务功能，参数检查，统计信息等

面向切面编程：就是将交叉业务逻辑封装成切面，利用 AOP 容器的功能将切面织入到主业务逻辑中。所谓交叉业务逻辑是指，通用的、与主业务逻辑无关的代码，如安全检查、事务、日志、缓存等



#### 3.2.2 AOP的作用

+ 减少代码重复
+ 专注业务逻辑
+ 实现业务功能和其他非业务功能解耦合



#### 3.2.3 AOP中的术语

+ 切面（Aspect）：可复用的非业务功能，例如日志、缓存等
+ 连接点（JoinPoint）：程序执行的某个特定位置：如类开始初始化前、类初始化后、类某个方法调用前、调用后、方法抛出异常后。一个类或一段程序代码拥有一些具有边界性质的特定点，这些点中的特定点就称为“连接点”。Spring仅支持方法的连接点，即仅能在方法调用前、方法调用后、方法抛出异常时以及方法调用前后这些程序执行点织入增强。连接点由两个信息确定：第一是用方法表示的程序执行点；第二是用相对点表示的方位。
+ 切入点（PointCut）：表示一组 joint point，这些 joint point 或是通过逻辑关系组合起来，或是通过通配、正则表达式等方式集中起来
+ target：目标对象，给那个对象增加切面功能，这个对象就是目标对象
+ Advice：Advice 定义了在 `Pointcut` 里面定义的程序点具体要做的操作，它通过 before、after 和 around 来区别是在每个 joint point 之前、之后还是代替执行的代码

AOP重要的三要素：Aspect、Pointcut、Advice。在Advice时间，在Pointcut位置，执行Aspect



### 3.3 AspectJ对AOP的实现

#### 3.3.1 AspectJ的通知类型以及其对应的注解

+ 前置通知 （@Before）
+ 后置通知 （@AfterReturning）
+ 环绕通知 （@Around）
+ 异常通知 （@AfterThrowing）
+ 最终通知（@After）

#### 3.3.2 AspectJ 的切入点表达式

AspectJ 定义了专门的表达式用于指定切入点。表达式的原型是： 

	execution(modifiers-pattern? ret-type-pattern declaring-type-pattern?name-pattern(param-pattern) throws-pattern?)

其中：

+ `modifiers-pattern]` 访问权限类型 

+ `ret-type-pattern` 返回值类型 （**必须设置**）

+ `declaring-type-pattern` 包名类名 

+ `name-pattern(param-pattern)` 方法名(参数类型和参数个数)  （**必须设置**）

+ `throws-pattern` 抛出异常类型 

+ `？`表示可选的部分

故上方表达式可以转换成

	execution(访问权限 方法返回值 方法声明(参数) 异常类型)



具体实例，例如`execution(public void com.ecifics.spring.service.SomeService.doSome(String,Integer))`



表达式中的符号

| 符号 | 意义              |
| :--: | :---------------: |
| *    | 0或者多个任意字符 |
|..|（1）用在方法参数中，表示任意多个参数 （2）用在包名后面，表示当前包及其子包路径|
|+|（1）用在类名后，表示当前类及其子类（2）用在接口后，表示当前接口及其实现类|



举例

+ execution(public * *(..)) 	指定切入点为：任意公共方法。 *
+ execution(* set*(..)) 	指定切入点为：任何一个以“set”开始的方法。 
+ execution(* com.xyz.service.*.*(..))	 指定切入点为：定义在 service 包里的任意类的任意方法。 
+ execution(* com.xyz.service..*.*(..)) 	指定切入点为：定义在 service 包或者子包里的任意类的任意方法。“..”出现 在类名中时，后面必须跟“*”，表示包、子包下的所有类。 
+ execution(* *..service.*.*(..)) 	指定所有包下的 serivce 子包下所有类（接口）中所有方法为切入点
+ execution(* *.service.*.*(..))	 指定只有一级包下的 serivce 子包下所有类（接口）中所有方法为切入点 
+ execution(* *.ISomeService.*(..)) 	指定只有一级包下的 ISomeSerivce 接口中所有方法为切入点 execution(* *..ISomeService.*(..)) 	指定所有包下的 ISomeSerivce 接口中所有方法为切入点
+ execution(* com.xyz.service.IAccountService.*(..)) 	指定切入点为：IAccountService 接口中的任意方法。 *
+ *execution(* com.xyz.service.IAccountService+.*(..)) 指定切入点为：IAccountService 若为接口，则为接口中的任意方法及其所有 实现类中的任意方法；若为类，则为该类及其子类中的任意方法。 *
+ *execution(* joke(String,int))) 	指定切入点为：所有的 joke(String,int)方法，且 joke()方法的第一个参数是 String，第二个参数是 int。如果方法中的参数类型是 java.lang 包下的类，可 以直接使用类名，否则必须使用全限定类名，如 joke( java.util.List, int)。 
+ execution(* joke(String,*)))	 指定切入点为：所有的 joke()方法，该方法第一个参数为 String，第二个参数 可以是任意类型，如 joke(String s1,String s2)和 joke(String s1,double d2) 都是，但 joke(String s1,double d2,String s3)不是。*
+ execution(* joke(String,..))) 	指定切入点为：所有的 joke()方法，该方法第一个参数为 String，后面可以有 任意个参数且参数类型不限，如 joke(String s1)、joke(String s1,String s2)和 joke(String s1,double d2,String s3)都是。 
+ execution(* joke(Object)) 	指定切入点为：所有的 joke()方法，方法拥有一个参数，且参数是 Object 类 型。joke(Object ob)是，但，joke(String s)与 joke(User u)均不是。 
+ execution(* joke(Object+))) 	指定切入点为：所有的 joke()方法，方法拥有一个参数，且参数是 Object 类型或该类的子类。不仅 joke(Object ob)是，joke(String s)和 joke(User u)也 是。



### 3.3 AspectJ框架的使用

#### 3.3.1 Spring使用AspectJ框架的前置准备

（1）导入相应的依赖

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.2.5.RELEASE</version>
</dependency>

<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
    <version>5.2.5.RELEASE</version>
</dependency>
```



（2）创建业务接口及其实现类

SomeService.java

```java
package com.ecifics.spring.service;

/**
 * @author Ecifics
 * @Description TODO
 * @date 3/13/2022-8:32 PM
 */
public interface SomeService {
    void doSome(String name, Integer age);
}
```



SomeServiceImpl.java

```java
package com.ecifics.spring.service.impl;

import com.ecifics.spring.service.SomeService;

/**
 * @author Ecifics
 * @Description TODO
 * @date 3/13/2022-8:33 PM
 */
public class SomeServiceImpl implements SomeService {
    @Override
    public void doSome(String name, Integer age) {
        System.out.println("业务方法doSome()");
    }
}
```





（3）定义切面类，用来想业务类添加功能，需要在类上加上@Aspect，在方法上面加上AspectJ框架中的通知注解，例如@Before(value = "切入点表达式")

```java
package com.ecifics.spring.handler;

import org.aspectj.lang.annotation.Aspect;

/**
 * @author Ecifics
 * @Description TODO
 * @date 3/13/2022-8:35 PM
 */
@Aspect
public class MyAspect {
    
}
```



（4）创建spring配置文件

+ 声明目标对象
+ 声明切面类对象
+ 声明自动代理生成器

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans 
       http://www.springframework.org/schema/beans/spring-beans.xsd 
       http://www.springframework.org/schema/aop 
       https://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean id="someService" class="com.ecifics.spring.service.impl.SomeServiceImpl" />

    <bean id="myAspect" class="com.ecifics.spring.handler.MyAspect" />

    <aop:aspectj-autoproxy />
</beans>
```



#### 3.3.2 @Before前置通知

执行时间在目标方法之前执行，不会影响目标方法的执行和执行结果



定义切面类

```java
@Aspect
public class MyAspect {

    @Before(value = "execution(* *..do*(..))")
    public void myBefore(JoinPoint joinPoint) {

        System.out.println("前置方法中获取目标方法的定义：" + joinPoint.getSignature());

        System.out.println("前置方法中获取目标方法的名称：" + joinPoint.getSignature().getName());

        Object[] args = joinPoint.getArgs();
        for (Object arg : args) {
            System.out.println("前置参数: " + arg);
        }

        System.out.println("前置通知" + new Date());
    }
}
```



JoinPoint：表示正在执行的业务方法，相当于反射中的Method，作为**第一个参数**传入@Before注解下的方法，用于获取方法执行时的信息，例如方法的名称、方法参数的集合



#### 3.3.3 @AfterReturning 后置通知

后置通知：在目标方法执行后执行

属性

+ value：切入点表达式
+ returning：自定义的变量，表示目标方法的返回值，**自定义变量名称必须和通知方法的形参名一样**

通知方法的参数是目标方法的返回值，由于接受目标方法的返回值



示例代码

```java
@Aspect
public class MyAspect {

    @AfterReturning(value = "execution(public String *..doOther(..))",
            returning = "res")
    public void myAfter(Object res) {
        System.out.println(res);
    }
}
```

其中returning属性的值和myAfter方法的参数名相同



#### 3.3.4 @Around 环绕通知

`@Around`

环绕方法的定义

+ 方法是public
+ 方法**必须**有返回值
+ 方法**必须**有ProceedingJoinPoint参数



标准格式代码实例：

```java
@Aspect
public class MyAspect {

    @Around(value = "execution(public String *..doFirst(..))")
    public Object myAround(ProceedingJoinPoint proceedingJoinPoint) {
        
    }
}
```

上方通知方法的返回值Object，表示调用方法希望得到的执行结果（不一定是目标方法自己的返回值）

ProceedingJoinPoint：接口ProceedingJoinPoint 其有一个 proceed()方法，用于执行目标方法



实例代码：

```java
@Aspect
public class MyAspect {

    @Around(value = "execution(public String *..doFirst(..))")
    public Object myAround(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        System.out.println("执行目标方法前，添加日志");
        Object obj = proceedingJoinPoint.proceed();
        System.out.println("执行目标方法后，添加日志");
        return obj;
    }
}
```

通过环绕通知，可以修改目标方法的返回值，例如上述返回了obj对象，也可以返回任意类型的对象，例如一串自定义的字符串

调用对应方法的执行结果

```java
执行目标方法前，添加日志
执行业务方法doFirst()
执行目标方法后，添加日志
```



#### 3.3.5 @After最终通知

`@After`属性

+ value：切入点表达式

特点：

+ 在目标方法后执行
+ 总是会被执行（包括发生异常），类似try-catch-finally中的finally
+ 可以用来程序最后的首位工作。例如清除临时数据，变量。清理内容



使用方法和@Before类似



#### 3.3.6 @Pointcut 定义切入点表达式

当较多的通知增强方法使用相同的`execution`切入点表达式时，编写、维护均较为麻烦。AspectJ 提供了`@Pointcut`注解，用于定义 `execution` 切入点表达式。
其用法是，将`@Pointcut `注解在一个方法之上，以后所有的 `execution` 的`value` 属性值均可使用该方法名作为切入点。代表的就是`@Pointcut` 定义的切入点。这个使用`@Pointcut` 注解的方法一般使用 private 的标识方法，即没有实际作用的方法。



示例代码

```java
@Aspect
public class MyAspect {

    @Before(value = "myExecutionExpression()")
    public void myBefore() {
        System.out.println("前置通知myBefore");
    }

    @After(value = "myExecutionExpression()")
    public void myAfter() {
        System.out.println("后置通知myAfter");
    }

    @Pointcut(value = "execution(* *..doSome(..))")
    private void myExecutionExpression() {

    }
}
```



## 4. 事务

### 4.1 事务的概念

**事务**是一系列sql语句的集合，作为一个整体执行



### 4.2 Spring事务管理

事务原本是数据库中的概念，在 Dao 层。但一般情况下，需要将事务提升 到业务层，即 Service 层。这样做是为了能够使用事务的特性来管理具体的业务。 

在 Spring 中通常可以通过以下两种方式来实现对事务的管理：

+ 使用 Spring 的事务注解管理事务 
+ 使用 AspectJ 的 AOP 配置管理事务



### 4.3 Spring事务管理器

#### 4.3.1 事务管理器接口（PlatformTransactionManager）

事务管理器是 PlatformTransactionManager 接口对象。其主要用于完成 事务的提交、回滚，及获取事务的状态信息



事务管理器有很多实现类

+ DataSourceTransactionManager：使用 JDBC 或 MyBatis 进行数据库操作时使用。 
+ HibernateTransactionManager：使用 Hibernate 进行持久化数据时使用。



#### 4.3.2 Spring的回滚方式

Spring 事务的默认回滚方式是：发生运行时异常和 error 时回滚，未发生异常时时提交



### 4.4 事务定义接口 

事务定义接口 TransactionDefinition 中定义了事务描述相关的三类常量： 

+ 事务隔离级别
+ 事务传播行为
+ 事务默认超时时限



事务特征

+ 原子性：事务是最小的执行单位，不允许分割。事务的原子性确保动作要么全部完成，要么完全不起作用；

+ 一致性：执行事务前后，数据保持一致；

+ 隔离性：并发访问数据库时，一个用户的事物不被其他事物所干扰，各并发事务之间数据库是独立的；

+ 持久性: 一个事务被提交之后。它对数据库中数据的改变是持久的，即使数据库发生故障也不应该对其有任何影响。



#### 4.4.1 事务隔离级别

事务在多事务操作之间不会产生影响，这就叫隔离性

三个读问题：

+ 脏读：一个未提交的事务读取到另一个未提交事务的数据
+ 不可重复读：在事务过程中，行被检索两次，行之间的值在读取之间不同，也就是一个事务读到了另一个提交事务修改的数据
+ 虚读（幻读）：一个未提交的事务读取到另一个提交事务添加的数据



+ `DEFAULT`： 采 用 DB 默 认 的 事 务 隔 离 级 别 。 MySql 的默认为 REPEATABLE_READ； Oracle 默认为 READ_COMMITTED。 
+ `READ_UNCOMMITTED`：读未提交。未解决任何并发问题。 
+ `READ_COMMITTED`：读已提交。解决脏读，存在不可重复读与幻读。 
+ `REPEATABLE_READ`：可重复读。解决脏读、不可重复读，存在幻读 
+ `SERIALIZABLE`：串行化。不存在并发问题。

||脏读|不可重复读|幻读|
|:-:|:-:|:-:|:-:|
| READ_UNCOMMITTED | 有   | 有   | 有 |
|READ_COMMITTED|无|有|有|
|REPEATABLE_READ|无|无|有|
|SERIALIZABLE|无|无|无|



#### 4.4.2 事务传播行为

处于不同事务中的方法在相互调用时，执行期间事务方法的如何进行。

例如，A 事务中的方法 doSome()调用 B 事务中的方法 doOther()，在调用执行期间事务的如何使用，就称为事务传播行为。事务传播行为是加在方法上的



事务传播行为常量

+ **PROPAGATION_REQUIRED **
+ **PROPAGATION_REQUIRES_NEW **
+ **PROPAGATION_SUPPORTS **
+ PROPAGATION_MANDATORY 
+ PROPAGATION_NESTED 
+ PROPAGATION_NEVER 
+ PROPAGATION_NOT_SUPPORTED



（1）PROPAGATION_REQUIRED 

Spring默认传播行为，方法在调用的时候，如果存在事务则使用当前的事务，若没有事务，则新建事务，方法在新事务中进行

（2）PROPAGATION_REQUIRES_NEW

方法需要一个新事务，调用方法时，如果存在一个事务，那么原来的事务暂停，直到新事务执行完毕；如果方法没有事务，则新建一个事务，在新事物中执行

（3）PROPAGATION_SUPPORTS 

方法中有无事务都可以进行，例如查询操作



#### 4.4.3 事务超时时限

超时时间，以秒为单位，默认为-1

表示一个业务方法最长的执行关系，如果到达时间没有执行完毕，spring回滚事务



### 4.5 使用 Spring 的事务注解@Transactional管理事务

#### 4.5.1 @Transactional 注解

通过`@Transactional`注解方式，可将事务织入到相应**public方法**中，实现事务管理

属性：

+ `propagation`：用于设置事务传播属性。该属性类型为 Propagation 枚举， 默认值为 Propagation.REQUIRED。 
+ `isolation`：用于设置事务的隔离级别。该属性类型为 Isolation 枚举，默认 值为 Isolation.DEFAULT。 
+ `readOnly`：用于设置该方法对数据库的操作是否是只读的。该属性为 boolean，默认值为 false。 
+ `timeout`：用于设置本操作与数据库连接的超时时限。单位为秒，类型为 int， 默认值为-1，即没有时限。 
+ `rollbackFor`：指定需要回滚的异常类。类型为 Class[]，默认值为空数组。 当然，若只有一个异常类时，可以不使用数组。 
+ `rollbackForClassName`：指定需要回滚的异常类类名。类型为 String[]，默 认值为空数组。当然，若只有一个异常类时，可以不使用数组。 
+ `noRollbackFor`：指定不需要回滚的异常类。类型为 Class[]，默认值为空数 组。当然，若只有一个异常类时，可以不使用数组。 
+ `noRollbackForClassName`：指定不需要回滚的异常类类名。类型为 String[]， 默认值为空数组。当然，若只有一个异常类时，可以不使用数组



需要注意的是，@Transactional 若用在方法上，**只能用于 public 方法上**。

对于其他非public方法，如果加上了注解@Transactional，虽然 Spring 不会报错，但不会将指定事务织入到该方法中。**因为Spring会忽略掉所有非public方法上的@Transactional 注解**



#### 4.5.2 @Transactional注解的使用步骤

（1）在spring配置文件中声明事务

+ 声明事务管理器
+ 声明使用注解管理事务



声明事务管理器

```xml
<bean id="transactionManager" 
      class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
	<property name="dataSource" ref="dataSource" />
</bean>
```



开启注解驱动

```xml
<tx:annotation-driven transaction-manager="transactionManager" />
```



（2）在业务类的**public方法**中，加入@Transactional

如果没有设置`@Transactional`注解的属性，那么会使用各个属性的默认值





### 4.6 使用 AspectJ的AOP配置管理事务

所有操作都在配置文件中配置



#### 4.6.1 使用步骤

（1）在pom.xml文件中加入`spring-aspects`依赖



（2）在spring的配置文件中声明事务的内容

+ 声明事务管理器

```xml
<bean id="transactionManager"       class="org.springframework.jdbc.datasource.DataSourceTransactionManager">    
    <property name="dataSource" ref="dataSource" />
</bean>
```

+ 声明业务方法需要的事务属性（隔离级别，超时时限，传播行为）

  + name：业务方法名称

  + propagation：传播行为

  + isolation：隔离级别

  + read-only：是否只读
  + rollback-for：回滚的异常类列表，使用的异常类全限定名称

```java
<tx:advice id="" transaction-manager="transactionManager">
    <tx:attributes>
        <tx:method name="" propagation="" isolation="" read-only="" timeout="" rollback-for="" />
    </tx:attributes>
</tx:advice>
```



+ 声明切入点表达式并关联事务通知
  + id：切入点表达式唯一名称
  + expression：切入点表达式

```java
<aop:config>
	<aop:pointcut id="" expression="" />
    <aop:advisor advice-ref="serviceAdvice" pointcut-ref="" />
</aop:config>
```





