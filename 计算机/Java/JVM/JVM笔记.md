# JVM

[TOC]

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

## 三、Java内存区域

Java虚拟机定义了若干种程序运行期间会使用到的运行时数据区，其中有一些会随着虚拟机启动而启动，随着虚拟机退出而退出，例如方法区和堆，这些区域可以由多个线程共用

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



#### 作用

Java虚拟机栈主观Java程序的运行，它保存方法的局部变量、部分结果，并参与方法的调用和返回



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

局部变量表所需的容量大小是在编译器确定下来的，并保存的方法的Code属性的maximum local variables数据项中，在方法运行期间是不会修改局部变量表的大小

参数值的存放总是在局部变量数组的index0开始，到数组长度的-1的索引结束



局部变量表的最基本的存储单元为`Slot`（变量槽）

局部变量表中存放编译器可知的各种数据类型、引用类型和returnAddress类型

在局部变量表里，32位以内的类型只占用一个slot（包括returnAddress类型），64位的类型（long和double）占用两个slot

+ byte、short和char在存储前被转换成int，boolean也被转换成int，0表示false，1表示true

+ long 和 double占用两个slot

  

##### 局部变量表的特点

+ 方法嵌套调用的次数由栈的大小决定
  + 一般来说，栈越大，方法嵌套调用的次数越多。对于一个函数而言，它的参数和局部变量越多，使得局部变量表膨胀，它的栈帧就越大，以满足方法调用所需要传递的信息增大的需求。进而函数调用就会占用更多的栈空间，导致其嵌套调用次数就会减少
+ 局部变量表的变量只在当前方法调用中有效
  + 在方法执行时，虚拟机通过使用局部变量表完成参数值到参数变量列表的传递过程。当方法调用结束后，随着方法栈帧的销毁，局部变量表也会随之销毁

#### 栈的优点

+ Java虚拟机规范允许Java虚拟机栈的大小是动态或者固定不变的

+ 栈是一种快速有效的分配存储方式，访问速度仅次于程序计数器
+ JVM对Java虚拟机栈的操作只有两个：
  + 每个方法执行，伴随着进栈
  + 执行结束后的出栈工作
+ 对于栈来说不存在垃圾回收问题，但是会存在内存溢出（OOM）问题



## 专业名词

+ 类变量也叫静态变量，也就是在变量前加了static 的变量
+ 实例变量也叫对象变量，即没加static 的变量



## 面试题

### 使用PC寄存器存储字节码指令地址有什么用？为什么使用PC寄存器记录当前线程的执行地址呢？

因为CPU需要不停的切换各个线程，这时候切换回来以后，就得知道接着从哪儿开始继续执行。

JVM的字节码解释器就是需要通过改变PC寄存器的值来明确下一条应该执行什么样的字节码指令