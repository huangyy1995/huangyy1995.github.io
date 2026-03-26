---
layout: post
title: "python知识点复习"
date: 2018-01-13T16:39:59+08:00
tags:
  - Xingyao Huang
---

放假归来，这几天复习了一下好久不用的python，总结了一下知识点。

## 语法基础

  * tuple与list的异同
    * 都由多个元素组成
    * tuple由()组成，list由[]组成
    * tuple不可变，list可变
    * tuple表示的是一种结构，而list表示的是多个事物的集合
    * tuple操作比list快
  * 字符串用法要点

    * 转义符和其他语言类似
    * 在字符串前加r表示raw字符串: `print(r'C:\some\name')`
    * 使用’’’或者”””表示换行:
          
          print("""\  
          Usage: thingy [OPTIONS]  
               -h                        Display this usage message  
               -H hostname               Hostname to connect to  
          """)  
            
  
---  
  * if/elif/else嵌套  
通过缩进表示，标准建议使用4空格而非tab

  * while/else和for/else的用法  
else块会在循环按顺序执行完的情况下执行，即没有出发for/while里的break

  * value if cond else other的三段式表示

  * list comprehension
  * 常用操作符的运算优先级



## 函数

### 1、函数定义、参数传递、调用方式及返回值规则。

  * 没有显式返回值时的情况,返回的是None
  * list型及dict型的变长参数传递  
加`*`表示展开list，加`**`表示展开dict
  * 参数默认值定义规则  
设置了默认值的参数后不能跟未设置默认值的参数



### 2、理解嵌套函数及闭包的用法。

<http://yunjianfei.iteye.com/blog/2186092>

  * 闭包：在非全局（global）作用域中定义inner函数（即嵌套函数）时，会记录下它的嵌套函数namespaces（嵌套函数作用域的locals），可以称作：定义时状态，可以通过func_closure 这个属性来获得inner函数的外层嵌套函数的namespaces。



### 3、理解变量的作用域及生命周期（globals, locals）。

就作用域而言，Python与C有着很大的区别，在Python中并不是所有的语句块中都会产生作用域。只有当变量在Module(模块)、Class(类)、def(函数)中定义的时候，才会有作用域的概念。if condition并不会产生作用域。如下代码可以输出正确结果:  

    
    
    if True:  
        variable = 100  
        print (variable)  
    print (variable)  
      
  
---  
  
作用域级别: local > enclosing > global > built-in

一个non-L的变量相对于L而言，默认是只读而不能修改的。如果希望在L中修改定义在non-L的变量，为其绑定一个新的值，Python会认为是在当前的L中引入一个新的变量(即便内外两个变量重名，但却有着不同的意义)。即在当前的L中，如果直接使用non-L中的变量，那么这个变量是只读的，不能被修改，否则会在L中引入一个同名的新变量。这是对上述几个例子的另一种方式的理解。  
注意：而且在L中对新变量的修改不会影响到non-L的。当你希望在L中修改non-L中的变量时，可以使用global、nonlocal关键字。

### 4、yield语句及Generator的使用。

<https://www.jianshu.com/p/d09778f4e055>

### 5、熟悉Python官方的内置函数，如enumerate/eval/hasattr等

<https://docs.python.org/2/library/functions.html>

## 面向对象编程

### 1、类的定义及类的基本协议，如**init** /**str** 等。

### 2、类的继承、多继承及super的用法，特别是多继承下super调用可能引起的特殊情况。

### 3、理解属性（property）的应用场合及本质

### 4、理解类命名空间及其访问规则。

### 5、理解类的动态性。

### 6、理解方法和函数的异同

<http://blog.csdn.net/chili_min/article/details/10447923>

### 7、类的静态方法、类方法及其与一般方法、函数的异同

### 8、理解命名空间及对象空间，及此模型与C/C++的模型差异。

### 9、循环引用产生，解环及无法解环时Python的处理方式。

## 模块
    
    
    1、理解模块的本质是什么。
    2、分清内置模块、标准模块及扩展模块的区别。
    3、理解Python模块的加载机制。
    

<https://www.jianshu.com/p/c04dc172335e>  
4、熟悉Python官方提供的常用模块，如copy/types/os/random/re/gc等

## 异常

### 1、理解异常的使用方法及适用范围。

### 2、异常的栈展开原理

### 3、常见的异常类\

## Other links

[Python2, 3区别](<http://blog.51cto.com/9478652/2062653>)
