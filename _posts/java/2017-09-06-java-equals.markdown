---
layout: category
title:  "equals跟=="
description: 关于equals跟==的比较。
categories: java
#categories: [java, reference, base]
preview: /static/images/common/150/54.jpg
---



java中有两种类型  
基本类型：基本数据类类型存的是数值本身。  
引用类型：引用类型变量在内存放的是数据的引用。  
基本类型通过==比较的是他们的值大小，而引用类型比较的是他们的引用地址。

　　值类型（`int`,`char`,`long`,`boolean`等）都是用`==`判断相等性。对象引用的话，`==`判断引用所指的对象是否是同一个。  
　　`equals`是`Object`的成员函数，有些类会覆盖（`override`）这个方法，用于判断对象的等价性。例如`String`类，两个引用所指向的`String`都是`"abc"`，但可能出现他们实际对应的对象并不是同一个（和jvm实现方式有关），因此用`==`判断他们可能不相等，但用`equals`判断一定是相等的。

**总体来说要搞清楚，还是需要先理解清楚对象实体和引用之间的关系。**

*Object类中的equals实现*

````java
	public boolean equals(Object obj) {
	    return (this == obj);
	}
````

*Java部分类库重写了该方法*  
　　以`String`类为例：`String`类中的`equals`先判断两个类在堆内存中的地址是否一致，如果是则肯定为同一个对象；否则判断两者的内容是否一致。

### equals和==区别
1. 对于重写了`equals`方法的类库（`String`, `Integer`等），`==` 判断两个对象是否指向同一个引用，比较的是堆内存中的地址；而`equals`出了比较地址还比较“内容”是否一致。
2. 对于只是简单继承`Object`类`equals`方法的类库，`equals`和`==`的作用是相同的，因为在`Object`类中`equals`方法只是用`==`来进行判断，如果地址相同则返回`true`
3. `==`可以用来比较基本类型和引用类型，而`equals`只能用于引用类型的比较
4. `equals`具有五个特征：
  + 自省性（`reflexive`）：`x.equals(x)` 返回值为`true`
  + 对称性（`symmetric`）：`x.equals(y)`返回值为`true/false`，`y.equals(x)`返回值也为`true/false`
  + 传递性（`transitive`）：`x.equals(y)`返回值为`true，y.equals(z)`返回值为`true`，那么`x.equals(z)`返回值也为`true`
  + 连贯性（`consistent`）：在`x`、`y`没有修改的情况下，多次调用`x.equals(y)`的返回值应该始终一致
  + 对于`非null`的`x`，`x.equals(null)`返回值应该为`false` 
   
``值得一提的是：``*当重写 equals() 方法时，一般都要重写 hashCode() 方法，因为相同的对象必然有相同的 hash 码。*

给Interger 赋予的int数值在-128 - 127的时候，直接从cache中获取，这些cache引用对Integer对象地址是不变的，但是不在这个范围内的数字，则new Integer(i) 这个地址是新的地址，不可能一样的
````java
public static void main(String[] args) {
    Integer a = new Integer(3);
    Integer b = 3;                 
    int c = 3;
    System.out.println(a == b);    
    System.out.println(a == c);    
}
````
a == b分析 ：
Integer b = 3; 自动调用Integer.valueOf(3) 返回一个Integer的对象。 这个对象存放到cache中的（上面一段代码分析）。 而 Integer a = new Integer(3);
这里创建了一个新的对象Integer 所以 a == b 返回的是false。  
a == c 分析 ：
一个Integer 与 int比较，先将Integer转换成int类型，再做值比较，所以返回的是true。  

java中还有与Integer类似的是Long，它也有一个缓存，在区间[-128,127]范围内获取缓存的值，而Long与long比较的时候先转换成long类型再做值的比较。  
Double类型，它没有缓存，但是当Double与double比较的时候会先转换成double类型，再做值的比较。  

````java
Integer i01 = 59;
int i02 = 59;
Integer i03 = Integer.valueOf(59);
Integer i04 = new Integer(59);
````
>A: Integer 与 int 比较的时候将Integer转成int在比价两个值大小，所以i01 == i02  
B: Integer i01 = 59;默认处理Integer i01 =Integer.valueOf(59); i01与 i03数值在-128 - 127之间，所以在cache缓存中获取Integer对象，他们引用地址是一样的。所以i01 == i03   
C: i03获取的是cache中缓存好的Integer地址，而i04是重新在堆中创建一个地址，所以两个地址是不一样的 i03 != i04   
D:A一样的原理i02 == i04。  
