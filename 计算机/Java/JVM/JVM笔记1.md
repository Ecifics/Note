# JVM

[TOC]

## 一、JVM 概述

### 1.1 JVM整体结构

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/jvm/JVM%E6%95%B4%E4%BD%93%E7%BB%93%E6%9E%84.jfif" align="left" alt="JVM整体结构" width="700">



<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/jvm/JVM%E6%95%B4%E4%BD%93%E7%BB%93%E6%9E%84%E7%BB%86%E8%8A%82.png" align="left" alt="JVM整体结构" width="1000">



<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/jvm/Java%E7%A8%8B%E5%BA%8F%E7%BC%96%E8%AF%91%E8%BF%90%E8%A1%8C%E8%BF%87%E7%A8%8B.png" align="left" alt="Java程序编译运行过程">



## 二、类的加载过程

**注意这里是类的加载过程，不是对象的加载过程，类加载类变量（也就是加上了static修饰的变量）和类方法**

Java虚拟机把描述类的数据从Class文件加载到内存，并对数据进行校验、转换解析和初始化，最终形成可以被虚拟机直接使用的Java类型，这个过程被称作虚拟机的**类加载机制**

在Java语言里，类型的加载、蓝屏和初始化过程都是在程序运行期间完成的

类加载的目的：

+ 类加载器子系统负责从文件系统或者网络中加载Class文件，class文件在文件开头有特定的文件标识。
+ ClassLoader只负责class文件的加载，至于它是否可以运行，则由Execution Engine决定。
+ 加载的类信息存放于一块称为方法区的内存空间。除了类的信息外，方法区中还会存放运行时常量池信息，可能还包括字符串字面量和数字常量（这部分常量信息是Class文件中常量池部分的内存映射）



一个类从被夹在到虚拟机内存中开始，到卸载处内存位置，整个声明周期将会经历下面几个阶段

+ 加载（Loading）
+ 验证（Verification）
+ 准备（Preparation）
+ 解析（Resolution）
+ 初始化（Initialization）
+ 使用（Using）
+ 卸载（Unloading）

其中**加载、验证、准备、初始化和卸载这五个阶段顺序是确定的**，类的加载过程必须按照这个顺序开始

而解析阶段则不一定，为了支持Java语言的动态绑定特定，可以在初始化阶段之后再开始

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

加载阶段使用类加载器加载对象

加载的过程：

+ 通过一个类的全限定名获取定义此类的二进制字节流（例如Class文件）
+ 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构
+ 在内存中生成一个代表这个类的java.lang.Class对象，作为方法区这个类的各种数据的访问入口

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

- 为类变量（静态变量）分配内存并且设置该类变量的`默认初始值`，即`零值`
- 这里不包含用final修饰的static，因为final在编译的时候就会分配好了默认值，准备阶段会显式初始化
- **注意：这里不会为实例变量分配初始化，类变量会分配在方法区中，而实例变量是会随着对象一起分配到Java堆中**



#### 解析

+ 将常量池内的符号引用转换为直接引用的过程
+ 现在调用方法hello()，这个方法的地址是1234567，那么hello就是符号引用，1234567就是直接引用
+ 事实上，解析操作往往会伴随着JVM在执行完初始化之后再执行
+ 符号引用就是一组符号来描述所引用的目标。符号引用的字面量形式明确定义在《java虚拟机规范》的class文件格式中。直接引用就是直接指向目标的指针、相对偏移量或一个间接定位到目标的句柄
+ 解析动作主要针对类或接口、字段、类方法、接口方法、方法类型等。对应常量池中的CONSTANT Class info、CONSTANT Fieldref info、CONSTANT Methodref info等

### 2.4 初始化

\<client\>()方法：

+ 初始化阶段就是执行类构造器方法`<clinit>()`的过程
+ 此方法不需定义，是javac编译器自动收集类中的所有类变量的赋值动作和静态代码块中的语句合并而来。也就是说，**当我们代码中包含static变量的时候，就会有`<clinit>`( )方法**；如果当前类不存在static变量，那么它的字节码文件是不会存在`<clinit>( )`
+ `<clinit>()`方法中的指令按语句在源文件中出现的顺序执行
+ `<clinit>()`不同于类的构造器。（关联：构造器是虚拟机视角下的`<init>()`）
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

在\<init>()方法中会将a和b进行赋值，字节码如下

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

**注意图中的红字，加载器之间一般不是以继承的关系来实现的，而是通常使用组合关系来复用父类的代码**

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



### 2.8 系统类加载器（应用程序加载器7）

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

## 三、Java内存区域

Java虚拟机定义了若干种程序运行期间会使用到的运行时数据区，其中有一些会随着虚拟机启动而启动，随着虚拟机退出而退出，例如方法区和堆，这些区域可以由多个线程共用

一个进程拥有一个JVM实例，一个JVM实例拥有一个运行时数据区，进程中的线程共享运行时数据区中的堆和方法区，每一个线程各自用用一套程序计数器、本地方法栈和虚拟机栈

另一些则是与线程一一对应，这些与线程对应的数据区会随着线程的开始和结束而创建和销毁，例如程序计数器、本地方法栈和虚拟机栈

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/jvm/Java%E8%99%9A%E6%8B%9F%E6%9C%BA%E8%BF%90%E8%A1%8C%E6%97%B6%E6%95%B0%E6%8D%AE%E5%8C%BA.png" align="left" alt="Java运行时数据区">



### 3.1 程序计数器（Program Counter Register）

JVM中的程序计数寄存器并不是广义上的物理寄存器，将其命名为PC计数器或者程序计数器更贴切，JVM中的PC寄存器是对物理PC寄存器的一种抽象模拟

#### 作用

+ 程序计数器用来存储指令下一条指令的地址（字节码编号），也就是即将执行的指令代码，由执行引擎读取下一条指令

+ 程序计数器是程序控制流的指示器，分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖程序计数器来完成
+ 字节码解释器工作时就是通过这个计数器的值来选取下一条需要执行的字节码指令
+ 它是唯一一个在Java虚拟机规范中没有规定OutOfMemoryError情况的区域



#### 特点

+ 程序计数器是一块很小的内存空间，几乎可以忽略补剂，也是运行最快的存储区域
+ 在JVM规范中，每个线程都有它自己的程序计数器，是线程私有的，声明周期与线程的声明周期保持一致
+ 任何时间一个线程都只有一个方法在执行，也就是所谓的当前方法。程序计数器会存储当前正在执行的Java方法的JVM指令地址；或者如果实在执行native方法，则是未指定值



### 3.2 Java虚拟机栈

#### 内存中的堆和栈

栈是运行时的单位，而堆是存储的单位，也就是说，栈解决程序的运行问题，即程序如何执行，或者说如何处理数据，而堆解决的是数据存储的问题，即数据怎么放、放在哪儿



#### 概念

在每个线程创建时都会创建一个虚拟机栈，其内部有一个个的栈帧（栈中存储数据的基本单位），一个栈帧对应一个方法调用

栈顶的栈帧表示当前执行的方法，并且栈是线程私有的，不共享

Java虚拟机栈的生命周期和线程一样

**栈中没有GC垃圾回收**



#### 作用

Java虚拟机栈主管Java程序的运行，它保存方法的局部变量、部分结果，并参与方法的调用和返回



#### 设置栈的大小

可以通过`-Xss`参数来设置线程的最大栈空间，见Oracle官方文档的描述

Sets the thread stack size (in bytes). Append the letter `k` or `K` to indicate KB, `m` or `M` to indicate MB, and `g` or `G` to indicate GB. The default value depends on the platform: Mb ps

- Linux/x64 (64-bit): 1024 KB
- macOS (64-bit): 1024 KB
- Oracle Solaris/x64 (64-bit): 1024 KB
- Windows: The default value depends on virtual memory

The following examples set the thread stack size to 1024 KB in different units:

```
Copy-Xss1m
-Xss1024k
-Xss1048576
```



#### 栈中可能出现的异常

Java虚拟机规范允许Java虚拟机栈的大小是动态或者固定不变的，因此会有两种异常

+ 当采用固定大小的Java虚拟机栈，那每一个线程的Java虚拟机栈容量可以在线程创建的时候独立设置。如果线程请求分配的栈容量超过了Java虚拟机允许的最大容量，Java虚拟机将会抛出一个`StackOverflowError`异常
+ 当Java虚拟机栈可以动态扩展，并且在尝试扩展的时候无法申请到足够的内存，或者在创建新线程的时候没有足够的内存去创建对应的虚拟机栈，拿Java虚拟机将会抛出一个`OutOfMemoryError`异常



#### 栈的特点

+ 栈是一种快速有效的分配储方式，访问速度仅次于程序计数器

+ JVM对Java虚拟机栈的操作只有两个：
  + 每个方法执行，伴随着进栈
  + 执行结束后的出栈工作
+ 对于栈来说不存在垃圾回收问题，但是会存在内存溢出（OOM）问题



#### 栈运行原理

+ JVM直接对Java虚拟机栈的操作只有两个，就是对栈帧的压栈和出栈
+ 在一个运行的线程中，在同一时间点上，只会有一个活动的栈帧。即只有当前正在执行的方法的栈帧才是有效的，这个栈帧被称为`当前栈帧`，与当前栈帧对应的方法称为`当前方法`，定义这个方法的类就是`当前类`
+ 执行引擎运行的所有字节码指令只对当前栈帧进行操作
+ 如果在该方法中调用了其他方法，对应的新栈帧会被创建出来，放在栈的顶端，成为新的当前栈帧
+ 不同线程中所包含的栈帧是不允许存在相互引用的，即不可能在一个栈帧内引用另一个线程的栈帧
+ 如果当前方法调用了其他方法，方法返回之际，当前栈帧会传回此方法的执行结果给前一个栈帧，接着虚拟机会丢弃当前栈帧，使得前一个栈帧重新称为当前栈帧
+ Java方法有两种返回函数的方式
  + 正常的函数返回，即使用return命令
  + 方法执行中出现未捕获处理的异常，以抛出异常的方式结束



#### 栈帧的内部结构

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/jvm/Java%E8%99%9A%E6%8B%9F%E6%9C%BA%E6%A0%88%E5%86%85%E9%83%A8%E7%BB%93%E6%9E%84.png" align="left" alt="Java虚拟机栈的内部结构">

+ 局部变量表（Local Variables）
+ 操作数栈（Operand Stack）
+ 动态链接（Dynamic Linking，指向运行时常量池的方法引用）
+ 方法返回地址（Return Address，方法正常退出或异常退出的定义）
+ 一些附加信息



#### 局部变量表（Local Variables）

局部变量表也叫`本地变量表`或`局部变量数组`

定义为一个数字数组，主要用于存储方法参数和定义在方法体内的局部变量，这些数据类型包括各类基本数据类型，对象引用，以及returnAddress类型

由于局部变量表是建立在线程的栈上，是线程的私有数据，因此不存在数据安全问题

局部变量表所需的容量大小是在编译期确定下来的，并保存的方法的Code属性的maximum local variables数据项中，在方法运行期间是不会修改局部变量表的大小

参数值的存放总是在局部变量数组的index0开始，到数组长度的-1的索引结束

在栈帧中，与性能调优关系最为密切的部分就是提到的局部变量表，在方法执行时，虚拟机使用局部变量表完成方法的传递

局部变量表中的变量也是重要的垃圾回收根节点，只要被局部变量表中直接或间接引用的对象都不会被回收



##### 局部变量表的特点

+ 方法嵌套调用的次数由栈的大小决定
  + 一般来说，栈越大，方法嵌套调用的次数越多。对于一个函数而言，它的参数和局部变量越多，使得局部变量表膨胀，它的栈帧就越大，以满足方法调用所需要传递的信息增大的需求。进而函数调用就会占用更多的栈空间，导致其嵌套调用次数就会减少
+ 局部变量表的变量只在当前方法调用中有效
  + 在方法执行时，虚拟机通过使用局部变量表完成参数值到参数变量列表的传递过程。当方法调用结束后，随着方法栈帧的销毁，局部变量表也会随之销毁



##### 关于Slot的理解

局部变量表的**最基本的存储单元为`Slot`（变量槽）**

局部变量表中存放编译器可知的各种数据类型、引用类型和returnAddress类型

在局部变量表里，32位以内的类型只占用一个slot（包括returnAddress类型），64位的类型（long和double）占用两个slot

+ byte、short和char在存储前被转换成int，boolean也被转换成int，0表示false，1表示true

+ long 和 double占用两个slot




JVM会为局部变量表中的每一个Slot都分配一个访问索引，通过这个索引即可成功访问到局部变量中指定的局部变量值

当一个实例方法被调用的时候，它的方法参数和方法内部定义的局部变量将会按照顺序被复制到局部变量表中的每一个slot上

**如果当前帧是由构造方法或者实例方法（也就是非static方法）创建的，那么该对象引用this将会存放在index为0的slot处，其余的参数按照参数表顺序排列**

栈帧中的局部变量表中的槽位是可以重复使用的，如果一个变量过了其作用域，那么在其作用域之后申明新的局部变量就很有可能复用过期局部变量的槽位，从而达到节省资源的目的



#### 操作数栈（Operand Stack）

操作数栈在方法执行的过程中，根据字节码指令，往栈中写入数据或提取数据，即入栈（push）或出栈（pop），主要是用于保存计算过程的中间结果，同时作为计算过程变量的临时存储空间

+ 有些字节码指令将值压入操作数栈，其余的字节码指令将操作数出栈，使用它们后再把它们压入栈中，例如复制、交换或者求和操作等

如果被调用的方法带有返回值的话，其返回值将会被压入当前栈帧的操作数栈中，并更新程序计数器中的下一条需要执行的字节码指令，返回值通过`aload`命令获取

操作数栈中的元素的数据类型必须与字节码指令的序列严格匹配，这由编译器在编译期间进行验证，同时在类加载过程中的类检验阶段的数据流分析阶段还要再次检验

操作数栈就是JVM执行引擎的一个工作区，当一个方法刚开始执行的时候，一个新的栈帧也会随之被创建出来，这个方法的操作数栈是空的

每一个操作数栈都会拥有一个明确的栈深度用于存储数值，所需的最大深度在编译器就定义好了，保存在方法的Code属性中，为max_stack的值

栈中的任何一个元素都可以是任意的Java数据类型

+ 32bit的类型占用一个栈单位深度
+ 64bit的类型占用两个栈单位深度



##### 相关指令

`bipush`:一个byte范围内的数据（大小在-128~127之间的数）压入操作数栈

`istore_index`：将操作数栈中栈顶元素放入局部变量表索引为index的元素，例如istore_1表示将操作数栈中栈顶元素放入局部变量表索引为1的位置

`iload_index`：将局部变量表中索引为index的元素压入操作数栈中

`iadd`：将操作数栈栈顶两个元素出栈之后相加，将结果压入栈



##### 栈顶缓存技术 0

由于完成一项操作的时候需要使用很多的入栈和出栈操作，这同时意味着更多的指令分派次数和内存读写次数

频繁的执行内存读/写操作必将影响执行速度，为了解决这个问题，HotSpot JVM的设计者提出了栈顶缓存（ToS，Top-of-Stack Caching）技术，**将栈顶元素全部缓存到物理CPU寄存器中，来降低对内存的读写次数，提升执行引擎的执行效率**



#### 动态链接（指向运行时常量池的方法引用）

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/jvm/%E5%8A%A8%E6%80%81%E9%93%BE%E6%8E%A5%E5%9B%BE%E7%A4%BA.png" align="left" alt="动态链接图示">

常量池，为了提供一些符号和常量，便于指令的识别

每一个栈帧内部都包含一个指向运行时常量池中该栈帧所属方法的引用。包含这个引用的目的就是为了支持当前方法的代码能够实现动态链接（Dynamic Linking），比如`invokedynamic`指令

在Java源文件被编译到字节码文件中时，所有的变量和方法引用都作为符号引用（Symbolic Reference）保存在class文件的常量池里。比如，描述一个方法调用了另外的其他方法时，就是通过常量池中指向方法的符号引用来表示的

例如下面字节码指令中的`#7`就是符号引用

```
invokevirtual #7
```

动态链接的作用就是为了将这些符号引用转换为调用方法的直接引用



#### 方法的调用

在JVM中，将符号引用转换成调用方法的直接引用与方法的绑定机制相关

##### 动态链接和静态链接
+ 静态链接
	+ 当一个字节码文件被装载进JVM内部时，如果被调用的目标方法在编译器可知，且运行期保持不变时，这种情况下将调用方法的符号引用转换为直接引用的过程称之为静态链接

+ 动态链接
	+ 如果被调用方法在编译器无法被确定下来，也就是说，只能够在程序运行期将调用方法的符号引用转换为直接引用，由于这种引用转换过程具备动态性，因此也就被称为动态链接


##### 早期绑定和晚期绑定
静态链接和动态链接对应的方法的绑定机制为：早期绑定（Early Binding）和晚期绑定（Late Binding）。绑定是一个字段、方法或者类在符号引用被替换为直接引用的过程，这仅仅发生一次。

+ 早期绑定
	+ 早期绑定就是指被调用的目标方法如果在编译期可知，且运行期保持不变时，即可将这个方法与所属的类型进行绑定，这样一来，由于明确了被调用的目标方法究竟是哪一个，因此也就可以使用静态链接的方式将符号引用转换为直接引用。
+ 晚期绑定
	+ 如果被调用的方法在编译期无法被确定下来，只能够在程序运行期根据实际的类型绑定相关的方法，这种绑定方式也就被称之为晚期绑定。

#### 虚方法和非虚方法
+ 如果方法在编译期就确定了具体的调用版本，这个版本在运行时是不可变的。这样的方法称为非虚方法。
+ 静态方法、私有方法、final 方法、实例构造器、父类方法都是非虚方法，其他方法称为虚方法。



##### 虚拟机中调用方法的指令

+ 普通调用指令
  + invokestatic：调用静态方法，解析阶段确定唯一方法版本
  + invokespecial：调用<init>方法、私有及父类方法，解析阶段确定唯一方法版本
  + invokevirtual：调用所有虚方法
  + invokeinterface：调用接口方法
+ 动态调用指令

+ invokedynamic：动态解析出需要调用的方法，然后执行
+ 区别
  + 前四条指令固化在虚拟机内部，方法的调用执行不可人为干预，而invokedynamic指令则支持由用户确定方法版本
  + 其中invokestatic指令和invokespecial指令调用的方法称为非虚方法，其余的（final修饰的除外）称为虚方法。



```java
/**
 * 解析调用中非虚方法、虚方法的测试
 *
 * invokestatic指令和invokespecial指令调用的方法称为非虚方法
 */
class Father {
    public Father() {
        System.out.println("father的构造器");
    }

    public static void showStatic(String str) {
        System.out.println("father " + str);
    }

    public final void showFinal() {
        System.out.println("father show final");
    }

    public void showCommon() {
        System.out.println("father 普通方法");
    }
}

public class Son extends Father {
    public Son() {
        //invokespecial 非虚方法
        super();
    }

    public Son(int age) {
        //invokespecial 非虚方法
        this();
    }

    //不是重写的父类的静态方法，因为静态方法不能被重写！
    public static void showStatic(String str) {
        System.out.println("son " + str);
    }

    private void showPrivate(String str) {
        System.out.println("son private" + str);
    }

    public void show() {
        //invokestatic 非虚方法
        showStatic("baidu.com");

        //invokestatic 非虚方法
        super.showStatic("good!");

        //invokespecial 非虚方法
        showPrivate("hello!");

        //invokevirtual
        //虽然字节码指令中显示为invokevirtual，但因为此方法声明有final，不能被子类重写，所以也认为此方法是非虚方法。
        showFinal();

        //invokespecial 非虚方法
        super.showCommon();

        //invokevirtual 虚方法
        //有可能子类会重写父类的showCommon()方法
        showCommon();
        
        //invokevirtual 虚方法
      	//info()是普通方法，有可能被重写，所以是虚方法
        info();

        MethodInterface in = null;
        //invokeinterface 虚方法
        in.methodA();
    }

    public void info() {

    }

    public void display(Father f) {
        f.showCommon();
    }

    public static void main(String[] args) {
        Son so = new Son();
        so.show();
    }
}

interface MethodInterface {
    void methodA();
}
```



#### 方法重写

步骤： 

+ 找到操作数栈的栈顶元素所执行的对象的实际类型，记作C
+ 如果在常量池中找到与类型C描述符和简单名称都相符的方法，则进行权限校验：
  + 如果通过则返回这个方法的直接引用，查找过程结束
  + 如果不通过，则返回`java.lang.IllegalAccessError`异常
+ 如果没有在常量池中没有找到相符合方法，则按照继承关系从下往上依次对C的各个父类进行第2步的搜索和验证过程
+ 如果始终没有找到合适的方法，则抛出`java.lang.AbstractMethodError`异常



##### 虚方法表

在面向对象的编程中，会频繁的使用动态分派，如果每次动态分派都要重新在类的元数据中搜索合适的目标的话可能会影响效率。

因此为了提高性能，JVM采用在**每个类**的方法区建立一个虚方法表，使用索引代替查找，其中存放着各个方法的入口

虚方法表会在类加载的链接阶段被创建并开始初始化，类的变量初始值准备完成之后，JVM会把该类的方法也初始化完毕



#### 方法返回地址

方法返回地址用来存放调用该方法的PC寄存器的值

方法退出（无论是正常退出还是出现未处理异常退出），方法退出后都会返回到该方法被调用的位置。

+ 方法正常退出时，调用者的pc寄存器的值作为返回地址，即调用该方法的指令的下一条指令
  + 方法正常调用完成后需要使用哪一个返回指令需要根据方法返回值的实际数据类型而定
    + `ireturn`：对应boolean、byte、short和int类型
    + `lreturn`：对应long类型
    + `freturn`：对应float类型
    + `dreturn`：对应double类型
    + `areturn`：对应引用类型
    + `return`：对应void类型方法、构造方法、雷和接口的
+ 方法异常退出时，返回地址要通过异常表来确定，栈帧中一般不会保存这部分信息
  + 方法执行过程中抛出异常时的异常处理，存储在一个异常处理表中，方便在发生异常的时候找到处理异常的代码
  + 如果这个异常没有在方法内处理，也就是本方法的异常表中没有搜索到匹配的异常处理，就会导致方法退出

本质上，方法的退出就是当前栈帧出栈的过程。此时，需要恢复上层方法的局部变量表、操作数栈、将返回值压入调用者的操作数栈、设置PC寄存器值等，让调用方法继续执行下去

正常退出和异常退出的区别在于：通过异常退出的不会向它的上层调用者产生任何返回值



### 3.3 本地方法（Native Method）接口

#### 概念

一个本地方法，也就是用`native`关键字修饰的方法是一个Java调用，非Java代码的接口，是由非Java代码实现的

本届方法接口的作用是融合不同的编程语言为Java所用，它的初衷是融合C/C++程序



### 3.4 本地方法栈



### 3.5 堆

#### 概述

+ 一个JVM实例只存有一个堆内存，堆也是Java内存管理的核心区域

+ Java堆区在JVM启动（运行程序，JVM使用启动类加载器加载类的时候，此时运行时数据区创建完成，堆也随之创建）的时候被创建

，起空间大小也随之确定，堆内存是JVM管理的最大一块内存空间

+ 堆内存的空间可以调节
+ JVM虚拟机规范规定，堆可以处在物理上不连续的内存空间，但在逻辑上它应该被视为连续的（操作系统虚拟化）
+ Java虚拟机规范中对Java堆的描述为：所有的对象实例都应该在运行时分配在堆上，实际上是大部分对象实例都在堆上分配内存（还有一部分通过逃逸分析优化之后分配到栈上）
+ 数组和对象永远不会存储在栈上，应为栈帧中保存的是引用，这个引用指向对象或数组在堆中的位置
+ 在方法结束后，堆中的对象不会马上被移除，而是在垃圾回收的时候才会被移除
  + 堆是执行垃圾回收的重点区域

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/jvm/JVM%E4%B8%AD%E6%A0%88%E5%A0%86%E6%96%B9%E6%B3%95%E5%8C%BA%E4%B9%8B%E9%97%B4%E7%9A%84%E8%81%94%E7%B3%BB.png" align="left" alt="JVM中堆栈方法区的联系">



#### 堆中内存结构

+ JDK 7

  + 逻辑上分成：新生区 + 养老区 + 永久区

+ JDK 8

  + 逻辑上分成：新生区 + 养老区 + 元空间

  

#### 堆空间大小

##### 手动设置

`-Xms`：用来设置堆空间（新生区和养老区）的初始内存大小，其中`-X`是JVM运行参数，`ms`是memory start，即起始内存大小

`-Xmx`：用来设置堆空间（新生区和养老区）的最大内存大小

例如`-Xms600m -Xmx600m`

##### 默认大小

初始内存大小： 电脑内存大小 / 64

最大内存大小：电脑内存大小 / 4

**开发中建议将初始内存大小和最大内存大小设置为一样的**，防止GC过程中调整堆空间大小造成性能下降



#### 新生区和养老区

存储在JVM中的Java对象可以被划分为两类：

+ 一类是生命周期较短的瞬时对象，这些对象的创建和消亡都非常迅速
+ 一类是对象的声明周期非常长，在某些极端的情况下还能够和JVM的生命周期保持一致



Java堆中进一步可以细分为新生区（年轻代，YoungGen）和养老区（老年代，OldGen）

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/jvm/%E6%96%B0%E7%94%9F%E5%8C%BA%E5%92%8C%E5%85%BB%E8%80%81%E5%8C%BA.jfif" align="left" alt="新生区和老年区">

其中新生区可以划分为`Eden`空间、`Survivor0`空间和`Survivor1`空间（有时可以随机称survivor1和survivor2，空的可以叫做to区，不空的可以叫做from区）

在HotSpot中，新生区中Eden空间和另外两个Survivor空间占比为8:1:1，但实际项目中，因为自适应内存策略，占比不一定是8:1:1。当然，可以通过`-XX:SurvivorRation`来调整空间比例，例如`-XX:SurvivorRation=8`表示占用开间比例为8:1:1

**几乎所有Java对象都是在Eden区被new出来的**

绝大多数的Java对象的销毁都在新生区进行，很少在养老区进行，几乎不再永久区或者元空间进行



#### 设置新生区和养老区大小

+ 新生区和养老区占比
  + `-XX:NewRatio=？`，表示新生区和养老区的比例，例如`-XX:NewRatio=2`表示新生区占比为1，养老区占比为2，**默认值为2**
+ 新生区大小
  + `-Xmn`

注意，如果同时设置了`-XX:NewRatio`和`-Xmn`，那么最终以`-Xmn`设置的大小为准



#### 对象分配回收过程

+ new的对象先放`Eden`区。此区有大小限制。
+ 当`Eden`伊甸园的空间填满时，程序又需要创建对象，JVM的垃圾回收器将对伊甸园区进行垃圾回收（MinorGC或者YGC），将伊甸园区中的不再被其他对象所引用的对象进行销毁，再加载新的对象放到伊甸园区。
+ 然后将伊甸园中的剩余对象移动到`Survivor0`区。
+ 如果再次触发垃圾回收，此时将`Eden`区和`Survivor0`区进行垃圾回收，剩下的对象就会放到`Survivor1`区，并将对象的age值设置为1。
+ 如果再次经历垃圾回收，此时会重新放回幸存者`Survivor0`区，接着再去`Survivor1`区，并将对象的age值加一。
+ 啥时候能去养老区呢？可以设置次数。默认是15次，也就是对象的age值为15的时候。可以设置新生区进入养老区的年龄限制，设置 JVM 参数：`-XX:MaxTenuringThreshold=N` 进行设置
+ 在养老区，相对悠闲。当养老区内存不足时，再次触发GC：Major GC，进行养老区的内存清理
+ 若养老区执行了Major GC之后，发现依然无法进行对象的保存，就会产生OOM异常。



#### 图解

+ 我们创建的对象，一般都是存放在Eden区的，**当我们的Eden区满了后，就会触发GC操作**，一般被称为 `YGC / Minor GC`操作

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/jvm/%E5%A0%86%E4%B8%AD%E5%AF%B9%E8%B1%A1%E5%88%86%E9%85%8D%E8%BF%87%E7%A8%8B%E6%AD%A5%E9%AA%A4%E4%B8%80.png" align="left" alt="堆中对象分配过程步骤一">

+ 当我们进行一次垃圾收集后，红色的对象将会被回收，而绿色的独享还被占用着，存放在S0(Survivor From)区。同时我们给每个对象设置了一个年龄计数器，经过一次回收后还存在的对象，将其年龄加 1。
+ 同时Eden区继续存放对象，当Eden区再次存满的时候，又会触发一个MinorGC操作，此时GC将会把 Eden和Survivor From中的对象进行一次垃圾收集，把存活的对象放到 Survivor To区，同时让存活的对象年龄 + 1

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/jvm/%E5%A0%86%E4%B8%AD%E5%AF%B9%E8%B1%A1%E5%88%86%E9%85%8D%E8%BF%87%E7%A8%8B%E6%AD%A5%E9%AA%A4%E4%BA%8C.png" align="left" alt="堆中对象分配过程步骤二">

- 我们继续不断的进行对象生成和垃圾回收，当Survivor中的对象的年龄达到15的时候，将会触发一次 `Promotion 晋升`的操作，也就是将年轻代中的对象晋升到老年代中

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/jvm/%E5%A0%86%E4%B8%AD%E5%AF%B9%E8%B1%A1%E5%88%86%E9%85%8D%E8%BF%87%E7%A8%8B%E6%AD%A5%E9%AA%A4%E4%B8%89.png" align="left" alt="堆中对象分配过程步骤三"> 











对于Survivor1和Survivor2，复制之后会交换，谁空谁是To区



#### 特殊情况说明

##### Survivor区满了

- 在Eden区满了的时候，才会触发MinorGC，而**幸存者区满了后，不会触发MinorGC操作**
- 如果Survivor区满了后，将会触发一些特殊的规则，也就是可能直接晋升老年代



##### 对象分配的特殊情况

+ 如果来了一个新对象，先看看 Eden 是否放的下？
  + 如果 Eden 放得下，则直接放到 Eden 区
  + 如果 Eden 放不下，则触发 YGC ，执行垃圾回收，看看还能不能放下？
+ 将对象放到老年区又有两种情况：
  + 如果 Eden 执行了 YGC 还是无法放不下该对象，那没得办法，只能说明是超大对象，只能直接怼到老年代
  + 那万一老年代都放不下，则先触发重 GC ，再看看能不能放下，放得下最好，但如果还是放不下，那只能报 OOM 啦
+ 如果 Eden 区满了，将对象往幸存区拷贝时，发现幸存区放不下啦，那只能便宜了某些新对象，让他们直接晋升至老年区



<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/jvm/%E5%AF%B9%E8%B1%A1%E5%88%86%E9%85%8D%E6%B5%81%E7%A8%8B%E5%9B%BE.png" height="500" align="left" alt="对象分配流程图">



#### Minor GC（YGC）、Major GC与Full GC

我们都知道，JVM调优的一个环节，也就是垃圾收集，我们需要尽量的避免垃圾回收，因为在垃圾回收的过程中，容易出现STW（Stop the World）的问题，暂停其他线程，等垃圾回收结束之后，用户线程才恢复运行，而 Major GC 和 Full GC出现STW的时间，是Minor GC的10倍以上

JVM在进行GC时，并非每次都对上面三个内存( 新生代、老年代；方法区 )区域一起回收的，大部分时候回收的都是指新生代。针对Hotspot VM的实现，它里面的GC按照回收区域又分为两大种类型：一种是部分收集（Partial GC），一种是整堆收集（FullGC）

+ 部分收集：不是完整收集整个Java堆的垃圾收集。其中又分为：
  + 新生代收集（ Minor GC/Young GC ）：只是新生代( Eden、S0/S1 )的垃圾收集
  + 老年代收集（ Major GC/Old GC ）：只是老年代的垃圾收集。
    + 目前，只有CMS GC会有单独收集老年代的行为。
    + **注意，很多时候Major GC会和Full GC混淆使用，需要具体分辨是老年代回收还是整堆回收。**
  + 混合收集（Mixed GC）：收集整个新生代以及部分老年代的垃圾收集。
    + 目前，只有G1 GC会有这种行为
+ 整堆收集（Full GC）：收集整个java整个堆（包括新生区、养老区和元空间）和方法区的垃圾收集。



##### 新生区GC（Minor GC）触发机制

+ 当新生区空间不足时，就会触发Minor GC，这里的新生区满指的是Eden区满，Survivor区满不会触发GC。（每次Minor GC会清理年轻代的内存）
+ 因为Java对象大多都具备朝生夕灭的特性，所以Minor GC非常频繁，一般回收速度也比较快。
+ Minor GC会引发STW，暂停其它用户的线程，等待垃圾回收线程结束，用户线程才恢复运行

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/jvm/%E6%96%B0%E7%94%9F%E5%8C%BAGC.png" align="left" alt="新生区GC">

##### 养老区GC（MajorGC/Full GC）触发机制

+ 指发生在老年代的GC，对象从老年代消失时，我们说 Major GC或 Full GC发生了
+ 出现了MajorGC，经常会伴随至少一次的Minor GC，但非绝对的，在Parallel Scavenge收集器的收集策略里就有直接进行Major GC的策略选择过程，也就是说在老年代空间不足时，会先尝试触发Minor GC，如果之后空间还不足，则触发Major GC
+ **Major GC的速度一般会比Minor GC慢10倍以上，STW的时间更长**
+ 如果Major GC后，内存还不足，就报OOM了



##### Full GC

触发Full GC执行的情况有如下五种：

+ 调用System.gc( )时，系统建议执行Full GC，但是不必然执行
+ 老年代空间不足
+ 方法区空间不足
+ 通过Minor GC后进入老年代的对象大小大于老年代的可用内存
+ 由Eden区、survivor space0（From Space）区 向survivor space1（To Space）区复制时，对象大小大于To Space可用内存，则把该对象转存到老年代，且老年代的可用内存 小于 该对象大小

**说明：Full GC 是开发或调优中尽量要避免的。这样STW时间会短一些**



#### 堆分代思想

分代的目的是为了优化GC的性能，如果没有分代，那所有的对象都在一起。需要GC的时候需要对堆的进行整体扫描，而很多对象生命周期都是非常短，因此将新创建对象放到某个区域，进行GC的时候先把这些生命周期非常短的对象进行回收，这样就可以腾出非常大的空间了



#### 内存分配策略（对象提升规则）

如果对象在`Eden`区经过第一次Minor GC存活下来，并且能背Survivor容纳的话，将被移动到Survivor空间中，并将age字段设置为1。

对象每在Survivor中经历一次Minor GC，age会加一，当它的age超过阈值（默认为15，每个JVM、GC有所不同），就会被提升到老年代中

对象提升到老年代的阈值可以通过`-XX:MaxTenuringThreshold`来设置



##### 针对不同age的对象分配原则

+ 优先分配到`Eden`
+ 大对象直接分配到老年代
  + 尽量避免程序中出现过多的大对象
+ 长期存活的对象分配到老年代
+ 动态年龄判断
  + 如果Survivor区中相同年龄的所有对象大小的综合大于Survivor空间的一半，年龄大于或等于该年龄的对象可以直接进入老年代，无需等到MaxTenuringThreshold中要求的年龄
+ 空间分配担保`-XX:HandlePromotionFailure`
  - 在发生Minor GC之前，虚拟机会检查老年代最大可用的连续空间是否大于新生代所有对象的总空间
    + 如果大于，此次Minor GC是安全的
    + 如果小于，则虚拟机会查看`-XX:HandlePromotionFailure`设置是否允许担保失败
      + 如果`-XX:HandlePromotionFailure=true`，那么会继续检查老年代最大可用连续空间是否大于历次提升到老年代对象的平均总大小
        + 如果大于，则尝试进行一个 Minor GC，但这次 MInor GC 依然是有风险的	
        + 如果小于，则进行一次 Full GC 
      + 如果`-XX:HandlePromotionFailure=false`，则进行一次 Full GC

**JDK7之后，HandlePromotionFailure参数不在会影响到虚拟机的空间分配担保策略，在JDK7之后，只要老年代的连续空间大于新生代对象总大小或者历次提升的平均对象总大小就会进行 Minor GC，否则进行 Full GC**



#### TLAB（Thread Local Allocation Buffer）

##### 为什么会有TLAB？

+ 堆区是线程共享区域，任何线程都可以访问到堆区中的共享数据
+ 由于对象实例的创建在JVM中非常频繁，因此在并发环境从堆区中划分内存空间是线程不安全的
+ 为了避免多个线程操作同一个地址，需要使用加锁机制，而加锁会影响分配速度



##### 什么是TLAB？

+ 从内存模型而不是垃圾手机的角度，对`Eden`区域继续进行划分，JVM为每个线程分配了一个私有缓存区域，它包含在`Eden`空间内
+ 多线程同时分配内存时，使用TLAB可以避免一系列的非线程安全问题，同时还能提升内存分配的吞吐量，因此我们可以将这种内存分配方式称为**快速分配策略**
+ 尽管不是所有的对象实例都能够在TLAB中成功分配内存，但JVM将TLAB作为内存分配的首选
+ 在程序中，可以通过`-XX:UseTLAB`设置开启TLAB空间
+ 默认情况下，TLAB空间的内存非常小，仅占整个`Eden`空间的1%，当然我们可以通过选项`-XX:TLABWasteTargetPercent`设置TLAB空间所占用Eden空间的百分比大小
+ 一旦对象在TLAB空间分配内存失败，JVM就会尝试着通过**使用加锁机制**确保数据操作的原子性，从而直接在Eden空间中分配内存



#### 常用参数设置

+ -XX:+PrintFlagsInitial：查看所有的参数的默认初始值
+ -XX:+PrintFlagsFinal：查看所有的参数的最终值（可能会存在修改，不再是初始值）
+ -Xms：初始堆空间内存（默认为物理内存的1/64）
+ -Xmx：最大堆空间内存（默认为物理内存的1/4）
+ -Xmn：设置新生代的大小（初始值及最大值）
+ -XX:NewRatio：配置新生代与老年代在堆结构的占比
+ -XX:SurvivorRatio：设置新生代中Eden和S0/S1空间的比例
+ -XX:MaxTenuringThreshold：设置新生代垃圾的最大年龄
+ -XX:+PrintGCDetails：输出详细的GC处理日志
+ -XX:+PrintGC 或 -verbose:gc ：打印gc简要信息
+ -XX:HandlePromotionFalilure：是否设置空间分配担保





#### 逃逸分析

##### 堆是分配对象存储的唯一选择吗？ 

在Java虚拟机中，对象是在Java堆中分配内存的，这是一个普遍的常识。但是，有一种特殊情况，那就是如果经过逃逸分析（Escape Analysis）后发现，一个对象并没有逃逸出方法的话，那么就可能被优化成栈上分配。这样就无需在堆上分配内存，也无须进行垃圾回收了。这也是最常见的堆外存储技术



##### 逃逸分析概述

+ 这是一种可以有效减少Java程序中同步负载和内存堆分配压力的跨函数全局数据流分析算法。
+ 通过逃逸分析，Java Hotspot编译器能够分析出一个新的对象的引用的使用范围从而决定是否要将这个对象分配到堆上。
+ 逃逸分析的基本行为就是分析对象动态作用域：
  + 当一个对象在方法中被定义后，对象只在方法内部使用，则认为没有发生逃逸。
  + 当一个对象在方法中被定义后，它被外部方法所引用，则认为发生逃逸。例如作为调用参数传递到其他地方中。



##### 代码分析

+ 没有发生逃逸的对象，则可以分配到栈上，随着方法执行的结束，栈空间就被移除

  ```java
  public void my_method() {
      V v = new V();
      // use v
      // ....
      v = null;
  }
  ```

+ 下面代码中的 StringBuffer sb 发生了逃逸

  ```java
  public static StringBuffer createStringBuffer(String s1, String s2) {
      StringBuffer sb = new StringBuffer();
      sb.append(s1);
      sb.append(s2);
      return sb;
  }
  ```

+ 如果想要StringBuffer sb不发生逃逸，可以这样写

  ```java
  public static String createStringBuffer(String s1, String s2) {
      StringBuffer sb = new StringBuffer();
      sb.append(s1);
      sb.append(s2);
      return sb.toString();
  }
  ```
  

##### 逃逸分析参数设置

+ 在JDK 1.7 版本之后，HotSpot中默认就已经开启了逃逸分析
  + 如果使用的是较早的版本，开发人员则可以通过：
    + 选项“-XX:+DoEscapeAnalysis"显式开启逃逸分析
    + 通过选项“-XX:+PrintEscapeAnalysis"查看逃逸分析的筛选结果



##### 利用逃逸分析进行代码优化

+ 栈上分配
  + 将堆分配转化为栈分配。如果一个对象在子程序中被分配，要使指向该对象的指针永远不会发生逃逸，对象可能是栈上分配的候选，而不是堆上分配
  + JIT编译器在编译期间根据逃逸分析的结果，发现如果一个对象没有逃逸出方法的话，就可能被优化成栈上分配。分配完成后，继续在调用栈内执行，最后线程结束，栈空间被回收，局部变量对象也被回收，这样就不需要进行垃圾回收
  
+ 同步省略
  + 如果一个对象被发现只有一个线程被访问到，那么对于这个对象的操作可以不考虑同步。
  
  + 线程同步的代价是相当高的，同步的后果是降低并发性和性能。
  
  + 在动态编译同步块的时候，JIT编译器可以借助逃逸分析来判断同步块所使用的锁对象是否只能够被一个线程访问而没有被发布到其他线程。
  
  + 如果没有，那么JIT编译器在编译这个同步块的时候就会取消对这部分代码的同步。这样就能大大提高并发性和性能。这个取消同步的过程就叫同步省略，也叫锁消除。
  
  + 例如下面的智障代码，根本起不到锁的作用
  
    ```java
    public void f() {
        Object hellis = new Object();
        synchronized(hellis) {
            System.out.println(hellis);
        }
    ```
    代码中对hellis这个对象加锁，但是hellis对象的生命周期只在f( )方法中，并不会被其他线程所访问到，所以在JIT编译阶段就会被优化掉，优化成：
    ```java
    public void f() {
      	Object hellis = new Object();
    		System.out.println(hellis);
    }
    ```
  
+ 分离对象或标量替换
  + 有的对象可能不需要作为一个连续的内存结构存在也可以被访问到，那么对象的部分（或全部）可以不存储在内存，而是存储在CPU寄存器中。
    + 其中标量是指一个无法再分解成更小的数据的数据，Java中的原始数据类型就是标量
    + 相对的，那些还可以分解的数据叫做聚合量，Java中的对象就是聚合量，因为它可以分解成其他聚合量和标量
    + 在JIT阶段，如果经过逃逸分析，发现一个对象不会被外界访问，那么经过JIT优化，就会把这个对象拆解成若干个其中包含的若干个成员变量来代替，这个过程就是标量替换
    
  + 举例
    ```java
    public static void main(String args[]) {
          alloc();
      }
      class Point {
          private int x;
          private int y;
      }
      private static void alloc() {
          Point point = new Point(1,2);
          System.out.println("point.x" + point.x + ";point.y" + point.y);
      }
      ````
      以上代码，经过标量替换后，就会变成
      ```java
      private static void alloc() {
          int x = 1;
          int y = 2;
          System.out.println("point.x = " + x + "; point.y=" + y);
      }
      ```



### 3.6 方法区

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/jvm/JVM%E4%B8%AD%E6%96%B9%E6%B3%95%E5%8C%BA.png" width="700px" align="left" alt="JVM方法区">

#### 栈、堆和方法区的交互关系

- Person 类的 .class 信息存放在方法区中
- person 变量存放在 Java 栈的局部变量表中
- 真正的 person 对象存放在 Java 堆中

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/jvm/%E5%A0%86%E6%A0%88%E5%92%8C%E6%96%B9%E6%B3%95%E5%8C%BA%E7%9A%84%E4%BA%A4%E4%BA%92.png" align="left" alt="堆栈和方法区的交互">

- 在 person 对象中，有个指针指向方法区中的 person 类型数据，表明这个 person 对象是用方法区中的 Person 类 new 出来的

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/jvm/Person%E5%AF%B9%E8%B1%A1%E5%9C%A8%E5%A0%86%E6%A0%88%E6%96%B9%E6%B3%95%E5%8C%BA%E4%B8%AD%E7%9A%84%E8%A1%A8%E7%A4%BA.jfif" align="left" alt="Person对象在堆栈方法区中的表示">



#### 方法区的位置

Java虚拟机规范里明确说明：“尽管所有的方法区在逻辑上是属于堆的一部分，但一些简单的实现可能不会选择去进行垃圾收集或者进行压缩”。

但对于HotSpot而言，方法区还有一个别名叫做Non-Heap，目的就是为了和堆分开



#### 方法区的特点

+ 方法区与堆一样，是由各个线程共享的区域
+ 方法区在JVM启动的时候被创建，在JVM关闭的时候释放，它的实际物理内存空间和Java堆区一样都可以是不连续的
+ 方法区的大小是可以选择固定大小和可扩展大小
+ 方法区的大小决定了系统可以保存多少个类，如果系统定义的类太多，导致方法区溢出，虚拟机会抛出内存溢出错误`java.lang.OutOfMemoryError: PermGen space`（JDK7之前）或者`java.lang.OutOfMemoryError:Metaspace`（JDK8之后）



#### HotSpot中方法区的演进

+ 在JDK7之前，习惯性吧方法区称为永久代，从JDK8开始，使用元空间取代了永久代。可以把方法区看成一个接口，而永久代和元空间看成是对方法区这个接口的实现
+ 本质上方法区和永久代并不等价，仅对HotSpot而言

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/jvm/%E6%B0%B8%E4%B9%85%E4%BB%A3%E5%92%8C%E5%85%83%E7%A9%BA%E9%97%B4%E7%A4%BA%E6%84%8F%E5%9B%BE.png" align="left" alt="永久代和元空间示意图">

+ 元空间与永久代的最大区别在于：元空间不在虚拟机设置的内存中，而是使用本地内存
+ 永久代和元空间不只是名字变了，内部结构也做了调整
+ 根据Java虚拟机规范指出，如果方法区无法满足新的内存分配需求时，将抛出OOM异常



#### 设置方法区大小

+ JDK7以及之前版本

  - 通过`-XX:Permsize`来设置永久代初始分配空间。默认值是20.75M

  - `-XX:MaxPermsize`来设定永久代最大可分配空间。32位机器默认是64M，64位机器模式是82M

  - 当JVM加载的类信息容量超过了这个值，会报异常`OutofMemoryError:PermGen space`

+ JDK8以及之后版本

  + 元数据区的大小可以使用参数`-XX:MetaspaceSize`和`-XX:MaxMetaspaceSize`指定

    + `-XX:MetaspaceSize`: The default size depends on the platform.Sets the size of the allocated class metadata space that will trigger a garbage collection the first time it is exceeded. This threshold for a garbage collection is increased or decreased depending on the amount of metadata used. 

      + 超过对应值会执行Full GC，重新设置该值，如果释放的空间太少，那么会在不超过`-XX:MaxMetaspaceSize`的情况下适当提高该值，如果释放空间过多，则应该适当降低该值

    + `-XX:MaxMetaspaceSize`: Sets the maximum amount of native memory that can be allocated for class metadata. By default, the size is not limited. The amount of metadata for an application depends on the application itself, other running applications, and the amount of memory available on the system.

      The following example shows how to set the maximum class metadata size to 256 MB:

      ```
      -XX:MaxMetaspaceSize=256m
      ```

  + 如果`-XX:MetaspaceSize`设置的值过小，则会发生多次垃圾回收并对该值做出调整，为了避免频繁的Full GC，应该将该值的初始值设置为较高的值



#### 方法区的内部结构

方法区用于存储已被虚拟机加载的类型信息、常量、静态变量、即时编译器（JIT）后的代码缓存等

类型信息包含了域信息和方法信息



#### 类型信息

对每个加载的类型（类class、接口interface、枚举类enum和注解类annotation），JVM必须在方法区中存储一下信息：

+ 这个类型的完整有效名称（包名+类名）
+ 这个类型的直接父类的完整有效名（对于interface或者java.lang.Object都没有父类）
+ 这个类型的修饰符（public、abstract或者final等等）
+ 这个类型直接接口的一个有序列表



#### 域（Field）信息

+ JVM必须在方法中保存类型的所有域的相关信息以及域的声明顺序
+ 域的相关信息包括
  + 域名称
  + 域类型
  + 域修饰符（public、private、protected、static、final、volatile、transient等）



##### 方法（Method信息）

JVM必须保存所有方法的以下信息，同域信息一样包括声明顺序

+ 方法名称
+ 方法的返回类型
+ 方法参数的数量和类型（按顺序）
+ 方法的修饰符
+ （public、private、protected、static、final、synchronized等）
+ 方法的字节码、操作数栈、局部变量表及大小（abstract和native方法除外）
+ 异常表（abstract和native方法除外）
  + 每个异常处理的开始位置、结束位置、代码处理在程序计数器中的偏移地址、被捕获的异常类的常量池索引



##### 类变量

静态变量和类相关联，随着类的加载而加载

用final修饰的类变量在编译阶段就完成了赋值而不用等到类加载的时候才赋值，没有final修饰的类变量则是在类加载过程中的准备阶段赋一个零值，初始化阶段才完成最终赋值



##### 运行时常量池

+ 运行时常量池是方法区的一部分
+ 常量池表是Class文件的一部分，用于存放编译器生成的各种字面量与符号引用，这部分内存将在类加载后存放到方法区的运行时常量池中
+ 运行时常量池，在加载类和接口到虚拟机后，就会创建对应的运行时常量池
+ JVM为每个已加载的类型（类或接口）都维护一个常量池，池中的数据项像数组项一样，通过索引进行访问
+ 运行时常量池中包含多种不同的常量，包括编译期就已经明确的数值字面量，也包括到运行期解析后才能获得的方法或者字段引用。此时不再是常量池中的符号地址，这里转换成真实地址
+ 运行时常量池类似于传统编程语言中的符号表，但是它所包含的数据却比符号表要更加丰富一些
+ 当创建类或接口的运行时常量池时，如果构造运行时常量池所需的内存空间超过了方法区所能提供的最大值，泽JVM会抛OutOfMemoryError异常



#### 字节码中常量池

在字节码文件中有常量池，通过类加载器将字节码文件加载到JVM内存中的时候，字节码中的常量池变成了运行时常量池。

一个有效的字节码文件中除了包含类的版本信息、字段、方法以及接口等描述信息外，还包含常量池表，包括各种字面量和对类型与和方法的符号应用

常量池可以看做是一张表，虚拟机指令根据这张表找到要执行的类名、方法名、参数类型、字面量等类型。



##### 常量池中有什么？

+ 数量值
+ 字符串值
+ 类引用
+ 字段引用
+ 方法引用



##### 为什么需要常量池？

一个java源文件中的类、接口，编译后产生一个字节码文件。而Java中的字节码需要数据支持，通常这种数据会很大以至于不能直接存到字节码里，换另一种方式，可以存到常量池，这个字节码包含了指向常量池的引用。





#### 方法区的垃圾回收



## 四、对象实例化内存布局与访问定位

### 4.1 对象实例化

#### 创建对象的方式

+ new
+ Class的newInstance()：反射的方式，只能调用空参的构造器，权限必须是public
+ Constructor的newInstance()：放射的方式，可以调用空参、带参的构造器，权限没有要求
+ 使用clone()：不调用任何构造器，当前类需要实现Cloneable接口，实现clone()
+ 使用反序列化：从文件、网络中获取二进制流
+ 第三方库Objenesis



#### 对象创建过程

+ 判断对象对应的类是否加载、连接和初始化
  + 虚拟机遇到一个new指令，首先去检查这个指令的参数是否能在Metaspace的常量池中定位到一个类的符号引用，并且检查这个符号引用代表的类是否已经被加载、解析和初始化。（即判断类元信息是否存在）。如果没有，那么在双亲委派模式下，使用当前类加载器以ClassLoader+包名+类名为Key进行查找对应的.class。如果没有找到文件，则抛出ClassNotFoundException异常，如果找到，则进行类加载，并生成Class类对象
+ 为对象分配内存
  + 首先计算对象占用的空间大小，接着在堆中划分一块内存给新对象。如果实例成员变量是引用变量，仅分配引用变量空间即可，即4字节大小
  + 选择那种分配方式由Java堆是否规整来决定，而Java堆是否规整又由所采用的的垃圾收集器是否带有压缩功能决定	
    + 内存规整
      + 虚拟机采用指针碰撞来为对象分配内存。也就是所有用过的内存在一边，空闲的内存在一边，用一个指针作为分界点的指示器，分配内存就是将指针向空闲的那边挪动一段与对象大小的距离。如果垃圾回收期选择的是Serial、ParNew这种基于压缩算法，虚拟机采用这种分配方式，一般带有compact过程的收集器时，使用指针碰撞

    + 内存不规整
      + 已使用的内存和未使用的内存相互交错，虚拟机将采用空闲列表的方法来为对象分配内存。在空闲列表上记录了哪些内存块是可用的，分配的时候找到一块足够大的空间分配给对象实例，并更新表上的内存，这种分配方式称为“空闲列表”


+ 处理并发安全问题
  + 采用CAS失败重试、区域加锁保证更新的原子性
  + 每个线程预先分配一块TLAB（通过`-XX:+/-UseTLAB`来设置）
+ 初始化分配到的空间
  + 所有属性设置默认值，保证对象实例字段在不赋值时可以直接使用
+ 设置对象的对象头
  + 将对象的所属类（即类的元数据信息）、对象的HashCode和对象的GC信息、锁信息等数据存储在对象的对象头中。这个过程的具体设置方式取决于JVM实现
+ 执行init方法进行初始化
  + 在Java程序的视角看来，初始化才正式开始。初始化成员变量，执行实例化代码块，调用类的构造方法，并把堆内对象的首地址赋值给引用变量
  + 因此一般来说（由字节码中跟随 invokespecial 指令所决定），new指令之后会接着就是执行init方法，把对象按照程序员的意愿进行初始化，这样一个真正可用的对象才算完成创建出来



#### 对象的内存布局

##### 对象头

- 运行时元数据
  - 哈希值（HashCode），可以看作是堆中对象的地址
  - GC分代年龄（年龄计数器）
  - 锁状态标志
  - 线程持有的锁
  - 偏向线程ID
  - 偏向时间戳
- 类型指针
  - 指向类元数据 InstanceKlass，确定该对象所属的类型。指向的其实是方法区中存放的类元信息



##### 实例数据

+ 它是对象真正存储而有效信息，包括程序代码中定义的各种类型的字段（包括从父类继承下来的和本身拥有的字段）
+ 规则
  + 相同宽度的字段总是被分配在一起
  + 父类中定义的变量会出现在子类之前
  + 如果CompactFields参数为true（默认为true），子类的窄变量可能插入到父类变量的空隙



##### 对齐填充

不是必须的，没有特别含义，起到占位符的作用



示例代码

```java
public class Customer {
    int id = 8001;
    String name;
    Account acct;
    
    {
        name = "匿名客户";
    }
    
    public Customer() {
        acct = new Account();
    }
}
```



<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/jvm/Customer%E7%B1%BB%E7%9A%84%E5%86%85%E5%AD%98%E5%B8%83%E5%B1%80.png" align="left" alt="Customer类的内存布局">

### 4.2 对象访问定位

JVM是如何通过栈帧中的对象引用访问到其内部的对象实例呢？

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/jvm/%E9%80%9A%E8%BF%87%E5%AF%B9%E8%B1%A1%E5%BC%95%E7%94%A8%E8%AE%BF%E9%97%AE%E5%AF%B9%E8%B1%A1%E5%AE%9E%E4%BE%8B%E8%BF%87%E7%A8%8B.png" align="left" alt="通过对象引用访问对象实例">



#### 句柄访问

一个对象对应一个句柄，一个句柄包含两部分

+ 到对象**实例**数据的指针
+ 到对象**类型**数据的指针



优点：reference中存储稳定句柄地址，对象被移动（垃圾收集时移动对象很普遍）时只会改变句柄中实例数据指针即可，reference本身不需要被修改

缺点：在堆空间中开辟了一块空间作为句柄池，句柄池本身也会占用空间；通过两次指针访问才能访问到堆中的对象，效率低



<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/jvm/%E5%8F%A5%E6%9F%84%E8%AE%BF%E9%97%AE.png" align="left" alt="句柄访问">

#### 直接指针（HotSpot采用）

优点：直接指针是局部变量表中的引用，直接指向堆中的实例，在对象实例中有类型指针，指向的是方法区中的对象类型数据

缺点：对象被移动（垃圾收集时移动对象很普遍）时需要修改 reference 的值

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/jvm/%E7%9B%B4%E6%8E%A5%E6%8C%87%E9%92%88.png" align="left" alt="直接指针">



## 五、直接内存

本地内存也就是系统的内存包含了直接内存和元数据区，直接内存是程序向操作系统申请的额外内存，是Java堆外内存（直接受到操作系统的管理，不会受到垃圾回收的影响）

#### 概述

+ 不是虚拟机运行时数据区的一部分，也不是《Java虚拟机规范》中定义的内存区域
+ 直接内存是在Java堆外的、直接向系统申请的内存区间
+ 来源于NIO，通过存在堆中的DirectByteBuffer操作Native内存
+ 通常，访问直接内存的速度会优于Java堆。即读写性能高
  + 因此出于性能考虑，读写频繁的场合可能会考虑使用直接内存
  + Java的NIO库允许Java程序使用直接内存，用于数据缓冲区



#### BIO和NIO

+ 非直接缓冲区（BIO），在读写本地文件时，我们需要从用户态切换成内核态

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/jvm/BIO.png" align="left" alt="BIO">

+ 直接缓冲区（NIO），操作系统划出的直接缓存区可以被Java代码直接访问，只有一份。NIO适合对大文件的读写操作

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/jvm/NIO.png" align="left" alt="NIO">



#### 直接内存与OOM

+ 直接内存也可能导致OutofMemoryError异常
+ 由于直接内存在Java堆外，因此它的大小不会直接受限于-Xmx指定的最大堆大小，但是系统内存是有限的，Java堆和直接内存的总和依然受限于操作系统能给出的最大内存
+ 直接内存的缺点为：
  + 分配回收成本较高
  + 不受JVM内存回收管理
    

#### 直接内存大小设置

直接内存大小可以通过`-XX:MaxDirectMemorySize`设置
如果不指定，默认与堆的最大值-Xmx参数值一致



## 六、执行引擎

### 执行引擎概述

java程序代码编译为字节码文件的过程叫做**前端编译**，从字节码文件编译成机器能够识别的机器指令之后让操作系统去执行的过程叫做**后端编译**

虚拟机是一个相对物理机的概念，这两种机器都有代码执行能力，其区别是物理机的执行引擎是直接建立在处理器、缓存、指令集和操作系统层面上的，而虚拟机的执行引擎则是由软件自行实现的，因此可以不受物理条件制约地定制指令集与执行引擎的结构体系，能够执行哪些不被硬件直接支持的指令集格式（通过执行引擎翻译成能被硬件直接支持的指令集格式）

JVM的主要任务是负责装载字节码到其内部，但字节码并不能够直接运行在操作系统上，因为字节码指令并非等价于本地机器指令，它内部包含的仅仅是一些能够被JVM所识别的字节码指令、符号表，以及其它辅助信息

那么，如果想要让一个Java程序运行起来，需要通过执行引擎将字节码指令解释/编译（后端编译）为对应平台上的本地机器指令才可以，执行引擎充当了高级语言和机器语言之间的翻译官



##### 执行引擎工作流程

+ 执行引擎将PC寄存器所指明的字节码翻译成机器码
+ 每当执行完一项指令操作后，PC寄存器就会更新下一条需要被执行的指令地址
+ 方法区执行的过程中，执行引擎可能会通过存储在局部变量表中的对象引用准确定位到存储在Java堆区中的对象实例信息，以及通过对象头中的元数据指针定位到目标对象的类型信息

#### Java代码编译和执行过程

前端编译器执行过程（源代码 -> Java字节码）：

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/jvm/Java%E4%BB%A3%E7%A0%81%E7%BC%96%E8%AF%91%E8%BF%87%E7%A8%8B.png" align="left" alt="Java代码编译过程">

后端编译器执行过程（Java字节码 -> 机器指令）

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/jvm/Java%E5%AD%97%E8%8A%82%E7%A0%81%E8%BD%AC%E6%8D%A2%E6%88%90%E6%9C%BA%E5%99%A8%E6%8C%87%E4%BB%A4.png" align="left" alt="Java字节码转换成机器指令">

#### 机器码、指令和汇编语言



#### 解释器（Interpreter）

JVM设计者的初衷知识为了满足java程序跨平台的特性，因此避免了静态编译直接生成本地机器指令，在早期使用解释器在运行时逐行解释字节码执行程序的想法

当Java虚拟机启动时会根据预定义的规范对字节码采用逐行解释的方式执行，将每条字节码文件中的内容通过解释器“翻译”为对应平台的本地机器指令执行，当前指令执行完成后，接着根据PC寄存器中记录的下一条需要被执行的字节码指令执行解释操作

但是基于解释器执行已经沦为了低效的代名词，为了解决低效的问题，就出现了JIT编译器40.



#### JIT编译器（Just In Time Compiler）

在部分商用虚拟机中（如HotSpot），Java程序最初是通过解释器（Interpreter）进行解释执行的，当虚拟机发现某个方法或代码块的运行特别频繁时，就会把这些代码认定为“*热点代码*”。为了提高热点代码的执行效率，在运行时，虚拟机将会把这些代码编译成与本地平台相关的机器码，并进行各种层次的优化，完成这个任务的编译器称为即时编译器



##### 热点代码

根据代码被调用执行的频率，那些需要频繁被编译为本地代码的字节码，被称为热点代码。一个被调用多次的方法或是一个方法体内部循环次数较多的循环体都可以被称为热点代码

JIT编译器会将热点代码编译成的机器指令存入codeCache中，当下次执行时，再遇到这段代码，就会从codeCache中读取机器码，直接执行，以此来提升程序运行的性能，因为这种编译发式发生在方法的执行过程中，因此也被称为栈上替换（On Stack Replacement）



##### 热点探测方式

HotSpot VM所采用的的热点探测方式是基于计数器的热点探测，HotSpot VM将会为每一个方法都建立2个不同类型的计数器

+ 方法调用计数器
+ 回边计数器
  + 统计循环体执行的循环次数



##### 方法调用计数器

+ 统计方法的调用次数
+ 默认阈值在Client模式下为1500次，在Server模式下是10000次。超过这个阈值，会触发JIT编译
+ 阈值可以通过`-XX:CompileThreshold`来设置
+ 如果一个方法被调用，会检查该方法是否被JIT编译过，如果是，则会使用编译后的本地指令来执行，如果不存在，会将方法的调用计数器加一，然后判断方法计数器与回边计数器之和是否超过方法调用计数器的阈值。如果超过阈值，会触发JIT编译器提交一个该方法的代码编译请求

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/jvm/%E6%96%B9%E6%B3%95%E8%B0%83%E7%94%A8%E8%AE%A1%E6%95%B0%E5%99%A8%E6%B5%81%E7%A8%8B%E5%9B%BE.png" align="left" alt="方法调用计数器流程图">





热度衰减

+ 如果不做任何设置，方法调用计数器统计的并不是方法被调用的绝对次数，而是一个相对的执行频率，即一段时间之间方法被调用的次数。当超过一定的时间限度，如果方法的调用次数仍然不足以让它提交给JIT编译器，那么这个方法的调用计数器就会被减少一半，这个过程称为方法调用计数器热度的衰减，而这段时间就成为方法调用计数器的半衰周期
+ 进行热度衰减的动作是在虚拟机进行垃圾回收时顺便进行的，可以调用虚拟机参数`-XX:UseCounterDecay`来关闭热度衰减，让方法计数器统计方法调用的绝对次数，这样，只要系统运行时间足够长，绝大部分方法都会被编译成本地代码
+ 可以使用`-XX:CounterHalfLifeTime`参数设置半衰周期的时间，单位是秒



##### 回边计数器

统计一个方法中循环体代码执行的次数，在字节码遇到控制流向后跳转的指令称为回边

<img src="https://notetuchuang-1305953527.cos.ap-chengdu.myqcloud.com/images/jvm/%E5%9B%9E%E8%BE%B9%E8%AE%A1%E6%95%B0%E5%99%A8%E6%B5%81%E7%A8%8B%E5%9B%BE.png" align="left" alt="回边计数器流程图">



#### HotSpot VM中JIT分类

HotSpot VM中有两种JIT编译器

+ C1编译器：Client Compiler
+ C2编译器：Server Compiler

开发人员可以通过对应的命令决定使用哪一种JIT编译器（64位操作系统只支持为C2，无法修改）

+ `-client`：指定Java虚拟机在Client模式下，使用C1编译器
+ `-server`：指定Java虚拟机运行在server模式下，使用C2编译器



##### C1编译器优化策略

C1编译器会对字节码进行简单和可靠的优化，耗时短，已到达更快的编译速度

+ 方法内联
  + 将引用的函数代码编译到引用点处，这样可以减少栈帧的生成，减少参数传递以及跳转过程
+ 去虚拟机化
  + 对唯一的实现类进行内联（是一种将函数体直接展开到调用处的一种优化技术）

+ 冗余消除
  + 在运行期间把一些不会执行的代码折叠



##### C2编译器优化策略

C2编译器优化是基于逃逸分析，会进行耗时较长的优化，以及激进优化，但优化效率更高

+ 标量替换
  + 用标量值代替聚合对象的属性值
+ 栈上分配
  + 对于未逃逸的对象分配对象在栈上而不是堆
+ 同步消除
  + 清除同步操作，通常指synchronized



##### 分层编译

程序执行优化（不开启性能监控）可触发C1编译器将自己码翻译成机器码，可以进行简单优化，也可以加上性能监控，C2编译器会根据性能监控信息进行激进优化

在Java7之后，一旦开发人员显示使用命令`-server`时，默认会开启分层编译策略，有C1编译器和C2编译器相互协作共同来执行编译任务







<img src="" align="left" alt="新生区GC">





<img src="" align="left" alt="新生区GC">





<img src="" align="left" alt="新生区GC">



<img src="" align="left" alt="新生区GC">



<img src="" align="left" alt="新生区GC">





<img src="" align="left" alt="新生区GC">





<img src="" align="left" alt="新生区GC">



<img src="" align="left" alt="新生区GC">

## 专业名词

+ 类变量也叫静态变量，也就是在变量前加了static 的变量
+ 实例变量也叫对象变量，即没加static 的变量



## 面试题

### 使用PC寄存器存储字节码指令地址有什么用？为什么使用PC寄存器记录当前线程的执行地址呢？

因为CPU需要不停的切换各个线程，这时候切换回来以后，就得知道接着从哪儿开始继续执行。

JVM的字节码解释器就是需要通过改变PC寄存器的值来明确下一条应该执行什么样的字节码指令