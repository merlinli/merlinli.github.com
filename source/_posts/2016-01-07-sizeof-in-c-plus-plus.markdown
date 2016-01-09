---
layout: post
title: "C和C++里的sizeof"
date: 2016-01-07 20:03
comments: true
categories: 
---

首先要明白的是sizeof 是一个操作符，而不是一个函数。 

 ****
**用法**：  
**sizeof(** type **)**  (1)     
**sizeof** expression   (2)	

返回值都是**std::size_t**.     
第一种用法是返回类型所占的字节数，对于**参数是类型的一定要加括号**  
第二种是返回表达式所返回的对象的所占的字节数，可加括号，可不加

**NOTE**


1. 不同的电脑架构，一个字节可能是8个比特位或者更多，具体多少个bit，可以查看**CHAR_BIT**.  
2. sizeof(char), sizeof(signed char), 和 sizeof(unsigned char) 总是返回**1**.  
3. 对于结构体和类，sizeof 的结果是加了内存对齐之后的值。   
    strcut A{  
     int a;    
     char b;  
     }  
     则sizeof (A) 结果是 8       


4. **当数组做形参时候，sizeof 的结果是指针的大小**


**sizeof 和 strlen的区别**：  
1. sizeof是计算类型所占的内存大小，一般在编译时候就能确定大小，而strlen是在运行时候计算的。  
2. 因为sizeof返回的是内存占用大小，所以对字符串常量计算的时候多加一个占位符，而strlen是计  算字符串实际大小，不包括最后的占位符。  
下面的结果是在I4P8的机器上：  
> char a[100]="abcd"; //  the sizeof a is 100, the strlen(a) is 4  
>  
> char b[]="abcd";    // the sizeof b is 5, the strlen(b) is 4   
> 
> char a = 'a' ;  // the sizeof a  is 1
> 
> sizeof 'c'      //the result is 4，单引号表示字符的ASCII值，所以返回是是个int  
> 
> sizeof "c"     // 2 ,字符和占位符加起来是两个



