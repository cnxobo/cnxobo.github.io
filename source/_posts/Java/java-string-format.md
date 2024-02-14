---
title: Java字符串格式化String.Format
tags:
  - format
  - java
  - string
id: '381'
categories:
  - - Java
date: 2014-09-14 22:55:54
---

每当拼字符串的时候，就会思念C的printf,不过后来才发现Java也实现了类似功能。

### 示例

代码：`String.format("Hello %s, you're the %03d visitor.", "xobo", 7);`

输出：`Hello xobo, you're the 007 visitor.`

默认情况下，参数会依次替换格式说明符。当我们需要重复引用时，就需要对格式说明符进行编号。

代码： `String.format("%1$s %1$s %1$s, you are over %2$.2fM tall .", "xobo", 1.0);`

输出：`xobo xobo xobo, you are over 1.00M tall .`

### 解释

#### 格式说明符

*   %s %03d %1$s %2$.2f等以%开始的字符串被称为格式说明符（format specifiers）；
*   格式说明符的定义为：
    
    > %\[argument_index$\]\[flags\]\[width\]\[.precision\]conversion_
    
*   \[\]表示非必填项，也就是说最简单的格式说明符定义为%conversion。

%为格式说明符的起始标志，如果想在字符串中输出%，就需要写成%%；

*   argument\_index 可选 是一个整数，它表示参数在参数列表的位置。**注意它是从1开始的**，第一个参数是1$，第二个参数是2$；
*   flags 可选 是可以改变输出格式的字符。​
    
    *   如果不提供flag默认格式为：
        
        *   输出按照width做右对齐
        *   负数以’-‘开始
        *   正数和零不带符号或者前置空格
        *   没有组分隔符​​​
        
        常用的flag:
        
        *   '-' 左对齐 （通用）
        *   '+' 数字一直带有符号 （数字有效）
        *   '0' 用0补空白处 （数字有效）
        *   ','
*   width 可选 是一个非负整数​，表示输出的最小字符数。
*   precision 可选 是一个非负整数​，限制输出字符长度。
*   conversion **必填** 一个字符，表明参数将如何被格式化。