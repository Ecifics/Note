# Linux

[TOC]

## 一、文件系统

### 1.1 文件分隔符

Linux下文件分隔符为`/`

Windows下为`\`



## 二、Vim编辑器

### 2.1 一般模式

`y$`复制从光标位置到行末的字符，按`p`进行粘贴

`y^`赋值从光标位置到行开始的字符，按`p`进行粘贴

### 2.2 命令模式

`Shitf + o`也就是`O`，可以在光标上面追加一列空行并将光标移动至改行开始处

`:wq!`强制修改没有写权限的文件9854



## 三、界面切换

`Ctrl + Alt + F(12345...)`切换图形界面和命令界面

也可以在图形界面输入命令`init 3`切换到命令界面（运行级别3）

在命令界面输入命令`init 5`切换到图形界面（运行级别5）



## 四、基本命令

### 4.1 删除

如果删除一个文件夹hello以及文件夹内的文件可以使用`rm -rf hello`

如果只想删掉文件夹内的文件，保留文件夹，可以使用`rm -rf hello/*`



## 五、Shell编程

### 5.1

通过bash sh 或者./shell脚本文件名 以及绝对路径 的方式去执行shell脚本时，会创建一个子shell去执行，如果脚本中的变量不是全局变量，那么在子shell中无法获取，也就无法操作

例如shell脚本文件hello.sh

```shell
#!/bin/bash
echo "hello, world"
echo $my_var
echo $new_var
```

其中"hello, world"和my_var都属于全局变量，而new_var属于局部变量

使用`./hello.sh`或者`sh ./hello.sh`不会输出new_var变量中的值，因为在子shell中没有定义这个变量

可以使用`source hello.sh`或者`. hello.sh`来执行，这样不会创建子shell，而在当前shell中执行