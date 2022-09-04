# JVM

[TOC]

## 七、String Table

### 7.1 String基本特性

+ String声明为final，不可被继承和修改，不可修改体现了不可变性
  + 当对字符串重新赋值时，需要重新申请一块内存区域进行赋值，不能使用原有的value进行赋值
  + 当对现有的字符串进行拼接时，也需要重新申请一块内存区域进行赋值，不能使用原有的value进行赋值
+ String实现了serializable接口和Comparable接口，表示字符串支持序列化和比较大小
+ JDK8中用final char[] value存储字符串数据，而JDK9改为final byte[] value
+ **通过字面量的方式给一个字符串赋值（String str = "hello"），此时的字符串值声明在字符串常量池中**

+ 字符串常量池不会存储相同内容的字符串
  + String Pool是一个固定大小的Hashtable，默认值长度大小为1009,。如果放入String Pool的字符串过多，会造成哈希冲突，调用String.intern性能会大幅下降
  + 通过`-XX:StringTableSize`可以设置StringTable长度
  + JDK6中，StringTable大小是固定的，为1009，可以任意设置StringTableSize的值
  + JDK7中，StringTable默认长度为60013，可以任意设置StringTableSize的值
  + JDK8开始，设置StringTable的大小必须大于等于1009



### String内存分配

为了让各种数据类型的访问速度更快、更节省内存，Java为八种数据类型和String都提供了常量池

String类型的常量池使用方法有两种

+ 使用双引号来声明String对象会直接存储在常量池中，例如 `String str = "Hello";`
+ 使用String提供的intern()方法



#### 内存分配

+ Java6，字符串常量池存放在永久代中
+ Java7，字符串常量池调整到堆中
  + 所有的字符串都保存在堆中，和其他普通对象一样，这样在调优的时候只需要调节堆大小即可
+ Java8，字符串常量在堆中的字符串常量池



### String基本操作



### 字符串拼接操作

- 常量与常量的拼接结果在常量池，原理是编译期优化
- 常量池中不会存在相同内容的变量
- 只要其中有一个是变量（如果是常量的话，还是采用编译器优化进行拼接，常量包括用final修饰的字符串对象），结果就在堆中，相当于通过`new String()`创建一个对象。变量拼接的原理是创建一个`StringBuilder`对象，然后调用append方法拼接两个字符串，最后通过toString()方法返回拼接后的字符串
- 如果拼接的结果调用intern( )方法，需要判断常量池中是否有该字符串
  - 如果有，直接返回字符串在常量池中的地址
  - 如果没有，需要将该字符串放入常量池中，并返回其在常量池中的地址



#### 示例代码

```java
@Test
public void test1() {
    String s1 = "a" + "b" + "c"; //编译期优化：等同于"abc"
    String s2 = "abc"; //"abc"一定是放在字符串常量池中，将此地址赋给s2
    /*
    * 最终.java编译成.class,再执行.class
    * String s1 = "abc";
    * String s2 = "abc"
    */
    System.out.println(s1 == s2); //true
    System.out.println(s1.equals(s2)); //true
}
```



```java
@Test
public void test2(){
    String s1 = "javaEE";
    String s2 = "hadoop";

    String s3 = "javaEEhadoop";
    String s4 = "javaEE" + "hadoop";//编译期优化
    //如果拼接符号的前后出现了变量，则相当于在堆空间中new String()，具体的内容为拼接的结果：javaEEhadoop
    String s5 = s1 + "hadoop";
    String s6 = "javaEE" + s2;
    String s7 = s1 + s2;

    System.out.println(s3 == s4);//true
    System.out.println(s3 == s5);//false
    System.out.println(s3 == s6);//false
    System.out.println(s3 == s7);//false
    System.out.println(s5 == s6);//false
    System.out.println(s5 == s7);//false
    System.out.println(s6 == s7);//false
    //intern():判断字符串常量池中是否存在javaEEhadoop值，如果存在，则返回常量池中javaEEhadoop的地址；
    //如果字符串常量池中不存在javaEEhadoop，则在常量池中加载一份javaEEhadoop，并返回此对象的地址。
    String s8 = s6.intern();
    System.out.println(s3 == s8);//true
}
```





#### 使用StringBuilder的append()方法拼接对象和使用加法来拼接对象效率比较

```java
// 通过加号品拼接两个字符串
public void method1(int highLevel){
    String src = "";
    for(int i = 0;i < highLevel;i++){
        src = src + "a";//每次循环都会创建一个StringBuilder、String
    }
}

// 通过StringBuilder拼接字符串
public void method2(int highLevel){
    //只需要创建一个StringBuilder
    StringBuilder src = new StringBuilder();
    for (int i = 0; i < highLevel; i++) {
        src.append("a");
    }
}
```

+ 通过StringBuilder的append()方法进行拼接字符串自始至终只需要创建一个StringBuilder对象；而通过加号的方式拼接字符串，在每一次拼接中都需要创建一个StringBuilder对象，拼接完成通过toString()方法来new一个String对象，占用内存很大，可能还需要额外时间去进行GC
+ 对于使用StringBuilder进行拼接，如果确定了拼接成的字符串长度，可使使用带参构造器，指定StringBuilder的容量大小，这样减少了扩容时造成的消耗

```java
public StringBuilder(int capacity) {
    super(capacity);
}
```



### intern()的使用

intern()方法会从字符串常量池中查询当前字符串是否存在，如果不存在会将该字符串放入常量池后，再返回该字符串在字符串常量池中的地址

intern()方法就是确保字符串在内存中只有一份拷贝，这样可以节省内存空间，加快字符串操作任务的执行速度



#### new String("abc")会创建几个对象？

答案为两个

+ 一个是new一个String对象
+ 另一个是在字符串常量池中创建对象"abc"



示例代码

```java
public static void main(String[] args) {
    String str = new String("abc");
}
```

字节码

```java
new #2 <java/lang/String>
dup
ldc #3 <abc>
invokespecial #4 <java/lang/String.<init> : (Ljava/lang/String;)V>
astore_1
return
```

> ldc 指令：Push item to operand stack from run-time constant pool



#### new String(“a”) + new String(“b”) 会创建几个对象

示例代码

```java
public class StringNewTest {
    public static void main(String[] args) {
        String str = new String("a") + new String("b");
    }
}
```



字节码

```java
 0 new #2 <java/lang/StringBuilder> //new StringBuilder()
 3 dup
 4 invokespecial #3 <java/lang/StringBuilder.<init> : ()V>
 7 new #4 <java/lang/String> //new String()
10 dup
11 ldc #5 <a> //常量池中的 “a”
13 invokespecial #6 <java/lang/String.<init> : (Ljava/lang/String;)V> //new String("a")
16 invokevirtual #7 <java/lang/StringBuilder.append : (Ljava/lang/String;)Ljava/lang/StringBuilder;> //append()
19 new #4 <java/lang/String> //new String()
22 dup
23 ldc #8 <b> //常量池中的 “b”
25 invokespecial #6 <java/lang/String.<init> : (Ljava/lang/String;)V> //new String("b")
28 invokevirtual #7 <java/lang/StringBuilder.append : (Ljava/lang/String;)Ljava/lang/StringBuilder;> //append()
31 invokevirtual #9 <java/lang/StringBuilder.toString : ()Ljava/lang/String;> //toString()里面会new一个String对象
34 astore_1
35 return
```



创建的对象：

+ 对象1：new StringBuilder()
+ 对象2：new String("a")
+ 对象3：常量池中的 “a”
+ 对象4：new String("b")
+ 对象5：常量池中的 “b”
+ 对象6：toString 中会创建一个 new String("ab")
  + **toString( )的调用，在字符串常量池中，没有生成"ab"**



#### JDK6 VS JDK7/8

```java
public void test() {
    String s1 = new String("1");
    s1.intern();
    String s2 = "1";
    System.out.println(s1 == s2); // jdk6/7/8:false

    String s3 = new String("1") + new String("1");
    s3.intern();
    String s4 = "11";
    System.out.println(s3 == s4); // jdk6:false, jdk7/8:true
}
```



第一部分中，在new String("1")的时候，就会在字符串常量池中生成字符串"1"，但是变量s1保存的是堆中字符串常量"1"的地址值，而不是堆中的字符串常量池中的字符串"1"的地址值，即使s1调用了intern()方法，但是他没有接收intern方法返回的字符串常量中的地址值，如果改成`s1 = s1.intern()`，那么最终结果为true



对于字符串变量s3，它是通过拼接的方式来创建字符串的，而在拼接的过程中最终会通过StringBuilder的toString()方法来创建字符串，但是并没有在字符串常量池中生成对应的字符串"11"，对于s3.intern()这一步，不同的jdk版本会有不同的处理方式

+ jdk6，会在字符串常量池中创建对象"11"
+ jdk7/8，字符串常量池中并不会创建对象"11"，而是创建一个指向堆中字符串"11"的地址值



注意如果交换了顺序，结果可能不一样

```java
public void test() {
    String s3 = new String("1") + new String("1");
    String s4 = "11";
    s3.intern();
    System.out.println(s3 == s4); // jdk6/7/8:false
}
```

此时交换了`String s4 = "11";`和`s3.intern();`，前一个语句在字符串常量池中生成了字符串"11"（和上一种情况不同，上一种情况中在intern()方法之前，常量池中没有"11"）



#### 总结

String的intern()的使用：

+ JDK1.6
  + 如果字符串常量池中有，不会将该字符串放入，而是返回已有的字符串对象的地址
  + 如果字符串常量池中没有，会将**对象**复制一份并放入常量池中，返回字符串对象的地址
+ JDK1.7起
  + 如果字符串常量池中有，不会将该字符串放入，而是返回已有的字符串对象的地址
  + 如果没有，会将**对象的引用地址**复制一份，放入字符串常量池中，并返回池中的**引用地址**





String的intern()效率：

+ 如果有大量重复的字符串，最好用intern来节省内存占用



### G1中的String去重操作	

+ 当垃圾回收器工作的时候，会访问堆上存活的对象。对每一个访问的对象都会检查是否是候选的要去重的String对象
  + 如果是，把这个对象的一个引用插入到队列中等待后续的处理，一个去重的线程在后台运行，处理这个队列，处理队列的一个元素意味着从队列删除这个元素，然后尝试去重它引用的String对象
+ 使用一个hashtable来记录所有的被String对象使用的不重复的char数组，当去重的时候，会检查这个哈hashtable，来看堆上是否存在一个一模一样的char数组
  + 如果存在，String对象会调整引用那个数组，释放对原来的数组的引用，最终会被垃圾回收器回收掉
  + 如果查找失败，char数组会被插入到hashtable，这样以后的时候就可以共享这个数组了