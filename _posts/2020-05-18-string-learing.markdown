---
layout:     post
title:      "String 的详细总结"
subtitle:   "Hello World, Hello Blog"
date:       2020-5-18
author:     "Caiiiiii"
header-img: "img/post-5-18.jpg"
tags:
    - Java
    - 基础
---

# String不可变
## 什么是不可变？
String不可变很简单，如下图，给一个已有字符串"abcd",第二次赋值成"abcedl"，不是在原内存地址上修改数据，而是重新指向一个新对象，新地址。
![46c03ae5abf6111879423f38375207cc_720w.jpg](http://ww1.sinaimg.cn/large/bfd348c6gy1gexu9t49yhj20k006hwek.jpg)

## String 为什么不可变？

String类是用final关键字修饰，这说明String类的主力成员字段 value是个char[]数组，而且是用final修饰的。final修饰的字段创建后就不可改变。

## 不可变的好处
安全。

![D}85Y0@_8NQT_CGACS~RGLW.png](http://ww1.sinaimg.cn/large/bfd348c6gy1gexuinc9i7j20gc0fdmxs.jpg)

# String的实例化方法

### 1.直接赋值，编译期就已经确定存储到Constant Pool中
`String str = "hello";`
Java 内存定义字符串常量池（JDK1.7后在堆中）,直接赋值时，会在字符串常量池中查找时候有'hello'的字段，如果没有则直接在字符串常量池中创建"hello",并且将该内存地址赋值给str,反之，直接将已创建的内存地址赋值给str。
```
String str1 = "hello";
String str2 = "hello";
System.out.println(str1 == str2) //true
```

### 2.通过构造方法实例化String类对象，创建的对象会存储到heap中,是运行期新创建的。
`String str = new String("hello");`
通过该方法创建String对象的时候，先在字符串常量池中查找时候存在"hello"字符串，如果没有则在字符串中创建"hello"，反之则不创建。第二步在堆中分配空间来创建"hello"对象并将内存地址赋值给str。
```
String str1 = "hello";
String str2 = new String("hello");
System.out.println("str1 == str2"); //false
```
ps: 有些笔试题中会问道，实例化时会创建几个对象：如果字符串常量池不存在创建的字符串，则会创建2个对象，分别在字符串常量池和堆中。

### 特殊的创建方法（笔试题中常出现类似的）
1.使用只包含常量的字符串连接符如”aa” + “aa”创建的也是常量,编译期就能确定,已经确定存储到String Pool中,String pool中存有“aaaa”；但不会存有“aa”。

2.使用包含变量的字符串连接符如”aa” + s1创建的对象是运行期才创建的,存储在heap中；只要s1是变量，不论s1指向池中的字符串对象还是堆中的字符串对象，运行期s1 + “aa”操作实际上是编译器创建了StringBuilder对象进行了append操作后通过toString()返回了一个字符串对象存在heap上。

3.String s2 = “aa” + s1; String s3 = “aa” + s1; 这种情况，虽然s2,s3都是指向了使用包含变量的字符串连接符如”aa” + s1创建的存在堆上的对象，并且都是s1 + “aa”。但是却指向两个不同的对象，两行代码实际上在堆上new出了两个StringBuilder对象来进行append操作。在Thinking in java一书中285页的例子也可以说明。

4.对于final String s2 = “111”。s2是一个用final修饰的变量，在编译期已知，在运行s2+”aa”时直接用常量“111”来代替s2。所以s2+”aa”等效于“111”+ “aa”。在编译期就已经生成的字符串对象“111aa”存放在常量池中。

### 延伸：比较字符串的‘==’和‘equals()’区别？

‘==’ 比较的是变量(栈)内存中存放的对象的(堆)内存地址，用来判断两个对象的地址是否相同，即是否是指相同一个对象。比较的是真正意义上的指针操作。注意：

1、比较的是操作符两端的操作数是否是同一个对象。

2、两边的操作数必须是同一类型的（可以是父子类之间）才能编译通过。

3、引用类型比较的是地址（即是否指向同一个对象），基本数据类型比较的是值，值相等则为true，如：int a=10 与 long b=10L 与 double c=10.0都是相同的（为true），因为他们都指向地址为10的堆。（String是字符串，不是基本数据类型）

‘equals()’用来比较的是两个对象是否相等，由于所有的类都是继承自java.lang.Object类的，在Object中的基类中定义了一个equals的方法，这个方法的初始行为是比较对象的内存地址，但String类中重写了equals方法， 比较的是字符串的内容 ，而不再是比较类在堆内存中的存放地址了。

总结：在没有重写equals方法的情况下，他们之间的比较还是基于他们在内存中的存放位置的地址值的，因为Object的equals方法也是用双等号（==）进行比较的，所以比较后的结果跟双等号（==）的结果相同。String类中重写了equals方法，变成了字符串内容的比较。


# StringBuilder 和 StringBuffer
```
StringBuffer 是线程安全的
StringBuilder 是线程不安全的，但速度上比StringBuffer快
```
String：String的值是不可变的，这就导致每次对String的操作都会生成新的String对象，这样不仅效率低下，而且大量浪费有限的内存空间。

当对字符串进行修改的时候，需要使用 StringBuffer 和 StringBuilder 类。

和 String 类不同的是，StringBuffer 和 StringBuilder 类的对象能够被多次的修改，并且不产生新的未使用对象。

StringBuilder 类在 Java 5 中被提出，它和 StringBuffer 之间的最大不同在于 StringBuilder 的方法不是线程安全的（不能同步访问）。

由于 StringBuilder 相较于 StringBuffer 有速度优势，所以多数情况下建议使用 StringBuilder 类。然而在应用程序要求线程安全的情况下，则必须使用 StringBuffer 类。
