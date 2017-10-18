---
layout: category
title:  "equals跟=="
description: 关于equals跟==的比较。
categories: java
#categories: [java, reference, base]
preview: /static/images/common/150/54.jpg
---



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
