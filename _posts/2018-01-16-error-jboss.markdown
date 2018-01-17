---
layout: post
title:  "jboss问题"
preview: /static/images/common/150/54.jpg
categories: test
---

## eclipse中的jboss runtime server 问题



build时无maven的jar包时，配置参照  
**如下图所示：**  
![各种模式](/static/images/src/jboss-maven-bulid.bmp)


## maven依赖中的scope详解

默认就是compile，什么都不配置也就是意味着compile。compile表示被依赖项目需要参与当前项目的编译，当然后续的测试，运行周期也参与其中，是一个比较强的依赖。打包的时候通常需要包含进去。

scope为test表示依赖项目仅仅参与测试相关的工作，包括测试代码的编译，执行。比较典型的如junit。

runntime表示被依赖项目无需参与项目的编译，不过后期的测试和运行周期需要其参与。与compile相比，跳过编译而已，说实话在终端的项目（非开源，企业内部系统）中，和compile区别不是很大。
比较常见的如JSR×××的实现，对应的API jar是compile的，具体实现是runtime的，compile只需要知道接口就足够了。oracle jdbc驱动架包就是一个很好的例子，一般scope为runntime。
另外runntime的依赖通常和optional搭配使用，optional为true。我可以用A实现，也可以用B实现。

provided意味着打包的时候可以不用包进去，别的设施(Web Container)会提供。事实上该依赖理论上可以参与编译，测试，运行等周期。相当于compile，但是在打包阶段做了exclude的动作。

system从参与度来说，也provided相同，不过被依赖项不会从maven仓库抓，而是从本地文件系统拿，一定需要配合systemPath属性使用。

### scope的依赖传递

A–>B–>C。当前项目为A，A依赖于B，B依赖于C。知道B在A项目中的scope，那么怎么知道C在A中的scope呢？答案是：
>当C是test或者provided时，C直接被丢弃，A不依赖C；  
否则A依赖C，C的scope继承于B的scope。  

下面是一张nexus画的图   
![scope](/static/images/src/scope.jpg)