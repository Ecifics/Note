# JVM



## 一、JVM 概述

### 1.1 JVM整体结构

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/jvm/JVM%E6%95%B4%E4%BD%93%E7%BB%93%E6%9E%84.jfif" align="left" alt="JVM整体结构" width="700">



<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/jvm/JVM%E6%95%B4%E4%BD%93%E7%BB%93%E6%9E%84%E7%BB%86%E8%8A%82.png" align="left" alt="JVM整体结构" width="1000">



## 二、类的加载过程

**注意这里是类的加载过程，不是对象的加载过程，类加载类变量（也就是加上了static修饰的变量）和类方法**

Java虚拟机把描述类的数据从Class文件加载到内存，并对数据进行校验、转换解析和初始化，最终形成可以被虚拟机直接使用的Java类型，这个过程被称作虚拟机的**类加载机制**



类加载的目的：

+ 类加载器子系统负责从文件系统或者网络中加载Class文件，class文件在文件开头有特定的文件标识。
+ ClassLoader只负责class文件的加载，至于它是否可以运行，则由Execution Engine决定。
+ 加载的类信息存放于一块称为方法区的内存空间。除了类的信息外，方法区中还会存放运行时常量池信息，可能还包括字符串字面量和数字常量（这部分常量信息是Class文件中常量池部分的内存映射）

### 2.1 类的加载流程图

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/jvm/%E7%B1%BB%E7%9A%84%E5%8A%A0%E8%BD%BD%E6%B5%81%E7%A8%8B.png" align="left" alt="JVM整体结构" width="800">

类的加载分为三个步骤

+ 加载
+ 连接（图中有误）
  + 验证
  + 准备
  + 解析
+ 初始化



### 2.2 加载

加载的过程：

+ 通过一个类的全限定名获取定义此类的二进制字节流（例如Class文件）
+ 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构
+ `在内存中生成一个代表这个类的java.lang.Class对象`，作为方法区这个类的各种数据的访问入口



> 加载.class文件的方式
>
> + 从本地系统中直接加载
> + 通过网络获取，典型场景：Web Applet
> + 从zip压缩包中读取，成为日后jar、war格式的基础
> + 运行时计算生成，使用最多的是：动态代理技术
> + 由其他文件生成，典型场景：JSP应用从专有数据库中提取.class文件，比较少见
> + 从加密文件中获取，典型的防Class文件被反编译的保护措施

### 2.3 连接

#### 验证

+ 目的在于确保Class文件的字节流中包含信息符合当前虚拟机要求，保证被加载类的正确性，不会危害虚拟机自身安全
+ 主要包括四种验证，文件格式验证，元数据验证，字节码验证，符号引用验证。
+ 使用 BinaryViewer 查看字节码文件，其开头均为 CAFE BABE ，如果出现不合法的字节码文件，那么将会验证不通过

#### 准备

- 为类变量分配内存并且设置该类变量的`默认初始值`，即`零值`
- `这里不包含用final修饰的static，因为final在编译的时候就会分配好了默认值，准备阶段会显式初始化`
- 注意：`这里不会为实例变量分配初始化`，类变量会分配在方法区中，而实例变量是会随着对象一起分配到Java堆中



#### 解析

+ 将常量池内的符号引用转换为直接引用的过程
+ 事实上，解析操作往往会伴随着JVM在执行完初始化之后再执行
+ 符号引用就是一组符号来描述所引用的目标。符号引用的字面量形式明确定义在《java虚拟机规范》的class文件格式中。直接引用就是直接指向目标的指针、相对偏移量或一个间接定位到目标的句柄
+ 解析动作主要针对类或接口、字段、类方法、接口方法、方法类型等。对应常量池中的CONSTANT Class info、CONSTANT Fieldref info、CONSTANT Methodref info等

### 2.4初始化

\<client\>()方法：

+ 初始化阶段就是执行类构造器方法`<clinit>()`的过程
+ 此方法不需定义，是javac编译器自动收集类中的所有类变量的赋值动作和静态代码块中的语句合并而来。也就是说，**当我们代码中包含static变量的时候，就会有`<clinit>`( )方法**；如果当前类不存在static变量，那么它的字节码文件是不会存在<clinit>( )
+ `<clinit>`()方法中的指令按语句在源文件中出现的顺序执行
+ `<clinit>`()不同于类的构造器。（关联：构造器是虚拟机视角下的`<init>()`）
+ 若该类具有父类，JVM会保证子类的`<clinit>()`执行前，父类的`<clinit>()`已经执行完毕
+ 虚拟机必须保证一个类的`<clinit>()`方法在多线程下被同步加锁

```java
public class ClassLoaderTest {
    private static int num = 1;

    static {
        num = 2;
        number = 20;
    }

    private static int number = 10;

    public static void main(String[] args) {
        System.out.println(num);
        System.out.println(number);
    }
}
```

其中number变量在链接的准备阶段会被初始化为0，故在初始化时，先被static块中赋值为20，再被赋值成10



\<init\>()方法，即类的构造器

```java
public class InitTest {
    private int a;
    private int b;

    public InitTest() {
        a = 20;
        b = 30;
    }

    public static void main(String[] args) {
        int c = 20;
    }
}
```

在\<init`>()方法中会将a和b进行赋值，字节码如下

```java
 0 aload_0
 1 invokespecial #1 <java/lang/Object.<init> : ()V>
 4 aload_0
 5 bipush 20
 7 putfield #2 <com/ecifics/demo/InitTest.a : I>
10 aload_0
11 bipush 30
13 putfield #3 <com/ecifics/demo/InitTest.b : I>
16 return
```



### 2.5  类加载器

#### 类加载器概述

class file存在于本地硬盘上，可以理解为设计师画在纸上的模板，而最终这个模板在执行的时候是要加载到JVM当中来根据这个文件实例化出n个一模一样的实例。

class file加载到JVM中，被称为DNA元数据模板，放在方法区。

在.class文件 –> JVM –> 最终成为元数据模板，此过程就要一个运输工具（类装载器Class Loader），扮演一个快递员的角色。



<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/jvm/%E7%B1%BB%E5%8A%A0%E8%BD%BD%E5%99%A8%E5%8A%A0%E8%BD%BD%E6%B5%81%E7%A8%8B.png" align="left" alt="类加载器加载流程">

#### 类加载器分类

JVM支持两种类型的类加载器

+ 引导类加载器（Bootstrap ClassLoader）

+ 自定义类加载器（User-Defined ClassLoader）

从概念上来讲，自定义类加载器一般指的是程序中由开发人员自定义的一类类加载器，但是Java虚拟机规范却没有这么定义，而是将所有派生于抽象类ClassLoader的类加载器都划分为自定义类加载器
无论类加载器的类型如何划分，在程序中我们最常见的类加载器始终只有3个，如下所示



<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/jvm/%E7%B1%BB%E5%8A%A0%E8%BD%BD%E5%99%A8.png" align="left" alt="类加载器">

**注意图中的红字，加载器之间是上下级关系**

所有派生于抽象类ClassLoader的类加载器都划分为自定义类加载器



```java
/**
 * 注意：getParent() 只是获取上层的加载器，并不是继承关系
 * @author xiexu
 * @create 2020-11-20 10:49 上午
 */
public class ClassLoaderTest {
    public static void main(String[] args) {

        //获取系统类加载器
        ClassLoader systemClassLoader = ClassLoader.getSystemClassLoader();
        System.out.println(systemClassLoader); //sun.misc.Launcher$AppClassLoader@18b4aac2

        //获取其上层：扩展类加载器
        ClassLoader extClassLoader = systemClassLoader.getParent();
        System.out.println(extClassLoader); //sun.misc.Launcher$ExtClassLoader@61bbe9ba

        //获取其上层：获取不到引导类加载器
        ClassLoader bootstrapClassLoader = extClassLoader.getParent();
        System.out.println(bootstrapClassLoader); //null

        //对于用户自定义类来说：默认使用系统类加载器进行加载
        ClassLoader classLoader = ClassLoaderTest.class.getClassLoader();
        System.out.println(classLoader); //sun.misc.Launcher$AppClassLoader@18b4aac2

        //String类使用引导类加载器进行加载的。--> Java的核心类库都是使用引导类加载器进行加载的。
        ClassLoader classLoader1 = String.class.getClassLoader();
        System.out.println(classLoader1); //null

    }
}
```



+ 我们尝试获取引导类加载器，获取到的值为 null ，这并不代表引导类加载器不存在，因为引导类加载器是由 C/C++ 语言构成的，所以我们是获取不到
+ 两次获取系统类加载器的值都相同：sun.misc.Launcher$AppClassLoader@18b4aac2 ，这说明系统类加载器是全局唯一的



### 2.6 启动类加载器（引导类加载器Boostrap Classloader）

+ 这个类加载使用C/C++语言实现的，嵌套在JVM内部
+ 它用来加载Java的核心库（JAVA_HOME / jre / lib / rt.jar、resources.jar 或 sun.boot.class.path 路径下的内容），用于提供JVM自身需要的类
+ 并不继承自java.lang.ClassLoader，没有父加载器
+ 加载扩展类和应用程序类加载器，并作为他们的父类加载器（当他俩的爹）
+ 出于安全考虑，Bootstrap启动类加载器只加载包名为java、javax、sun等开头的类



### 2.7 扩展类加载器（Extension ClassLoader）

+ Java语言编写，由sun.misc.Launcher$ExtClassLoader实现
+ 派生于ClassLoader类
+ 父类加载器为启动类加载器
+ 从java.ext.dirs系统属性所指定的目录中加载类库，或从JDK的安装目录的 jre / lib / ext子目录（扩展目录）下加载类库。如果用户创建的 JAR 放在此目录下，也会自动由扩展类加载器加载



### 2.8 系统类加载器

+ Java语言编写，由sun.misc.LaunchersAppClassLoader实现
+ 派生于ClassLoader类
+ 父类加载器为扩展类加载器
+ 它负责加载环境变量 classpath 或 系统属性java.class.path指定路径下的类库
+ 该类加载是程序中默认的类加载器，一般来说，Java应用的类都是由它来完成加载的
+ 通过classLoader.getSystemclassLoader( )方法可以获取到该类加载器



### 2.9 用户自定义类加载器

为什么需要自定义类加载器？

+ 在Java的日常应用程序开发中，类的加载几乎是由上述3种类加载器相互配合执行的，在必要时，我们还可以自定义类加载器，来定制类的加载方式。



那为什么还需要自定义类加载器？

+ 隔离加载类
+ 修改类加载的方式
+ 扩展加载源
+ 防止源码泄露



如何自定义类加载器？

+ 开发人员可以通过继承抽象类java.lang.ClassLoader类的方式，实现自己的类加载器，以满足一些特殊的需求
+ 在JDK1.2之前，在自定义类加载器时，总会去继承ClassLoader类并重写loadClass( )方法，从而实现自定义的类加载类，但是在JDK1.2之后已不再建议用户去覆盖loadClass( )方法，而是建议把自定义的类加载逻辑写在findclass( )方法中
+ 在编写自定义类加载器时，如果没有太过于复杂的需求，可以直接继承URIClassLoader类，这样就可以避免自己去编写findclass( )方法及其获取字节码流的方式，使自定义类加载器编写更加简洁。

### 2.10 双亲委派机制

#### 双亲委派机制原理

Java虚拟机对 class 文件采用的是按需加载的方式，也就是说当需要使用该类时才会将它的 class 文件加载到内存中生成 class 对象。而且加载某个类的class文件时，Java虚拟机采用的是双亲委派模式，即把请求交由父类处理，它是一种任务委派模式

+ 如果一个类加载器收到了类加载请求，它并不会自己先去加载，而是把这个请求委托给父类的加载器去执行；
+ 如果父类加载器还存在其父类加载器，则进一步向上委托，依次递归，请求最终将到达顶层的启动类加载器；
+ 如果父类加载器可以完成类加载任务，就成功返回，倘若父类加载器无法完成此加载任务，子加载器才会尝试自己去加载，这就是双亲委派模式。
+ 父类加载器一层一层往下分配任务，如果子类加载器能加载，则加载此类，如果将加载任务分配至系统类加载器也无法加载此类，则抛出异常

#### 双亲委派机制的优势

- 避免类的重复加载
- 保护程序安全，防止核心API被随意篡改
  - 自定义类：java.lang.String 没有调用
  - 自定义类：java.lang.ShkStart（报错：阻止创建 java.lang开头的类）



### 2.11 沙箱安全机制

+ 自定义String类时：在加载自定义String类的时候会率先使用引导类加载器加载，而引导类加载器在加载的过程中会先加载jdk自带的文件（rt.jar包中java.lang.String.class），报错信息说没有main方法，就是因为加载的是rt.jar包中的String类。
+ 这样可以保证对java核心源代码的保护，这就是沙箱安全机制。

### 三、Java内存区域

Java虚拟机定义了若干种程序运行期间会使用到的运行时数据区，其中有一些会随着虚拟机启动而启动，随着虚拟机退出而退出，例如方法区和堆，这些区域可以由多个线程共用

另一些则是与线程一一对应，这些与线程对应的数据区会随着线程的开始和结束而创建和销毁，例如程序计数器、本地方法栈和虚拟机栈





## 专业名词

+ 类变量也叫静态变量，也就是在变量前加了static 的变量
+ 实例变量也叫对象变量，即没加static 的变量



## 面试题

### 使用PC寄存器存储字节码指令地址有什么用？为什么使用PC寄存器记录当前线程的执行地址呢？

因为CPU需要不停的切换各个线程，这时候切换回来以后，就得知道接着从哪儿开始继续执行。

JVM的字节码解释器就是需要通过改变PC寄存器的值来明确下一条应该执行什么样的字节码指令