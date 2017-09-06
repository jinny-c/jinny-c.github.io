---
layout: category
title:  "String的分析"
description: 关于String类的分析。
categories: [java, string, base]
preview: /static/images/common/150/54.jpg
---


### java运行环境有一个字符串池（`string pool`）,由`String`类维护。

　　执行语句 `String str = "abc"` 时，首先查看字符串池中是否存在字符串`"abc"`，如果存在则直接将`"abc"`地址赋给`str`，如果不存在则先在字符串池中新建一个字符串`"abc"`，然后再将其赋给`str`。  
　　执行语句 `String str = new String("abc")` 时，不管字符串池中是否存在`"abc"`，直接新建一个字符串`"abc"`（注意：新建的字符串`"abc"`不是在字符串池中）,然后将其赋给`str`。  
　　前一语句的效率高，后一语句的效率低，因为新建字符串占用内存空间。`String str = new String()`;创建了一个空字符串，与`String str = new String("")`;相同。

### 引用变量与对象的区别：
　　字符串`"abc"` 是一个`String`对象，字符串池（`pool of literal strings`）和堆（`heap`）中存放着字符串对象。
1. 引用变量与对象
````java
    A a;
````
　　这个语句声明了一个类`A`的引用变量`a`，而对象一般通过`new`关键字创建，所以`a`仅仅是一个引用变量，而不是对象。
2. `java`中所有的字符串文字（字符串常量）都是一个`String`类的对象。  
　　有人（特别是c 程序员）在一些场合喜欢把字符串"当做/看成"字符数组，这也没有办法，因为字符串与字符数组存在一些内在的联系。事实上，它与字符数组是两种完全不同的对象。
3. 字符串对象的创建  
　　由于字符串对象的大量使用（它是一个对象，一般而言对象总是在堆（`heap`）中分配内存）,java中为了节省内存空间和运行时间（如比较字符串时，`==` 比 `equals()`方法快），在编译阶段就把所有的字符串文字放到一个字符串池（`pool of literal strings`）中，而运行时字符串池成为常量池的一部分。字符串池的好处，就是该池中所有相同的字符串常量被合并，只占用一个空间。

　　我们知道，对两个引用变量，使用 `==` 判断它们的值（引用）是否相等，即是否指向同一个对象：
````java
String s1 = "abc";
String s2 = "abc";
if(s1 == s2) System.out.println("s1,s2 refer to the same object");
else System.out.println("trouble");
````
　　这里的输出显示，两个字符串文字保存为一个对象，就是说，上面的代码只在字符串池（`string pool`）中创建了一个对象。
　　现在看看`String s = new String("abc");`语句，这里`"abc"`本身就是`pool`中的一个对象，而在运行时执行`new String()`时，将`pool`中的对象复制一份放到`heap`中，并且把`heap`中的这个对象的引用交给`s` 持有。ok，这条语句就创建了2个String对象。
````java
String s1 = new String("abc");
String s2 = new String("abc");
if(s1 == s2) { } // 不会执行的语句
````
　　这里用 `==` 判断就可知，虽然两个对象的"内容"相同（`equals()`判断）,但两个引用变量所持有的引用不同，上面的代码创建了几个String Object？ (三个，pool中一个，heap中两个）

**综上所述：**

　　创建字符串有两种方式：两种内存区域（`pool vs heap`）
1. `""` 引号创建的字符串在字符串池中
2. `new`， `new`关键字创建字符串时首先查看字符串池（`string pool`）中是否有相同值的字符串，如果有，则拷贝一份到堆（`heap`）中，然后返回堆中的地址；如果字符串池中没有，则在堆中创建一份，然后返回堆（`heap`）中的地址（注意，此时不需要从堆中复制到池中，否则，将使得堆中的字符串永远是池中的子集，导致浪费字符串池的内存空间）
3. 对字符串进行赋值时，如果右操作数含有一个或一个以上的字符串引用时，则在堆中再建立一个字符串对象，返回引用；如`String s = str1 + "blog"`；
比较两个已经存在于字符串池中字符串对象可以用 `==` 进行，拥有比`equals`操作更快的速度
