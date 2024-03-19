---
title: JavaScript Number 四舍五入并保留两位小数
tags:
  - IEEE-754
  - 浮点
id: "962"
categories:
  - - JavaScript
comments: false
date: 2022-07-12 21:52:32
---

JavaScript 浮点怎么算都算不对。

<!--more-->

## Math.round()

JavaScript 提供了一个函数`Math.round()` 来做整数的四舍五入。

```JavaScript
console.log(Math.round(0.4), Math.round(-0.4),Math.round(0.5), Math.round(-0.5), Math.round(0.6), Math.round(-0.6));
// 预期输出: 0 -0 1 -0 1 -1
```

`Math.round()` 准确的定义是把参数转化为最接近的整数，如果与相邻两个整数距离相等那么返回较大的整数。在正数情况下符合日常理解的四舍五入的行为，在负数情况下更像是五舍六入。 例如, -0.5，距离相邻两个整数 0, -1 距离一样，返回较大的整数 0。-0.6 距离-1 更近，返回-1。

## 使用 Math.round()先乘 100 再除 100 保留两位小数 ❌

Math.round() 只能处理整数，可以通过乘以 100 把小数点左移两位，然后再通过 Math.round()完成整数的四舍五入,然后再把小数点右移两位(x1/100)。

```JavaScript
function round2(num) {
    return Math.round(num * 100)/100;
}
console.log(round2(3.1415926),  round2(3.145926))
// 预期输出: 3.14 3.15
```

看起来是好的，再多一点测试。

```JavaScript
console.log(round2(1.005))
// 预期输出: 1.01 实际输出: 1
```

### Number.EPSILON ❌

浮点计算会丢精度可以考虑加一点点数值来修正。加一个`Number.EPSILON`。 `Number.EPSILON`是一个常量值，最小的大于 1 的浮点数和 1 的差。

> Chrome 下 Number.EPSILON =2.220446049250313e-16

```JavaScript
function round2(num) {
    return Math.round((num+Number.EPSILON) * 100)/100;
}
console.log(round2(3.1415926),  round2(3.145926), round2(1.005))
// 预期输出: 3.14 3.15 1.01
```

再多一点测试

```JavaScript
console.log(round2(5.015))
// 预期输出: 5.02 实际输出: 5.01
```

没的救了，换个其他方法

## toFixed ❌

toFixed 参数是小数点后保留几位小数，返回的是 string。参数默认为 0。

```JavaScript
console.log(12.345.toFixed(), 12.345.toFixed(1), 12.345.toFixed(2))
// 预期输出: 12 12.3 12.35
```

```JavaScript
function round2(num) {
    return parseFloat(num.toFixed(2));
}
console.log(round2(5.015))
// 预期输出: 5.02 实际输出: 5.01
```

一模一样的失败，还有个 toPrecision 跟 toFixed 一样的失败。

### toPrecision ❌

toPrecision 参数一共有几位有效数字，包括整数和小数位。如果没填参数，就等同于`Number.prototype.toString()`。

```JavaScript
console.log(12.345.toPrecision(), 12.345.toPrecision(1), 12.345.toPrecision(2), 12.345.toPrecision(3), 12.345.toPrecision(4))
// 预期输出: 12.345 1e+1 12 12.3 12.35
```

### toPrecision & round

浮点数在计算时会丢失精度，可以通过 JavaScript 有效数字位数是 17, 可以通过降低精度 toPrecision(15)完成精度的修正。差不多算是错错之后看起来为正。

```JavaScript
function round2(num) {
    return Math.round((num* 100).toPrecision(15) )/100;
}
console.log(round2(5.015), round2(1.005), round2(39.425))
// 预期输出: 5.02 1.01 39.43
```

## Exponential notation

任何一个实数都可以写成 $x \* 10^y$ 的形式，用指数表示法就是 `xEy`, `e`或`E` 在浮点数字中代表指数。

```JavaScript
console.log(1e2==100, 1e-2=0.01)
// 预期输出: true true
```

通过指数表示法，避免对浮点数做计算的方式来调整小数点位置，防止精度损失放大，然后配合 round 完成四舍五入。

```JavaScript
function roundNumber(num, scale) {
  if(!("" + num).includes("e")) {
    return +(Math.round(num + "e+" + scale)  + "e-" + scale);
  } else {
    var arr = ("" + num).split("e");
    var sig = ""
    if(+arr[1] + scale > 0) {
      sig = "+";
    }
    return +(Math.round(+arr[0] + "e" + sig + (+arr[1] + scale)) + "e-" + scale);
  }
}
function round2(num) {
    return roundNumber(num, 2);
}
console.log(round2(5.015), round2(1.005), round2(39.425))
// 预期输出: 5.02 1.01 39.43
```

### lodash

Lodash 的 round 实现原理类似。 \_.round(number, \[precision = 0\])

```JavaScript
console.log(_.round(1.005, 2))
// 预期输出: 1.01
```

## 为什么一个简单的四舍五入并保留两位小数这么麻烦

### 0.1 在浮点数里并不存在

当我们声明一个小数时，比如`0.1`其实它并不是`0.1`它其实是`0.10000000000000000555111512312578270211815834045410`。编程语言在默认打印的时候会降低精度再打印，让他看起来是`0.1`。

```JavaScript
console.log(0.1.toFixed(50))
// 输出: 0.10000000000000000555111512312578270211815834045410

console.log(0.2.toFixed(50))
// 输出: 0.20000000000000001110223024625156540423631668090820

console.log(0.5.toFixed(50))
// 输出: 0.50000000000000000000000000000000000000000000000000

console.log(5.015.toFixed(50))
// 输出: 5.01499999999999968025576890795491635799407958984375
```

这样就能看出为什么保留两位小数有时候好使，有时候就不好使。

### 浮点的标识方式

浮点数是采用科学计数法来表示一个数字的，常见的编程语言的浮点数都是基于 IEEE754 标准。 IEEE754 浮点数基数固定是 2，所以还需要关注符号、尾数和指数三部分。 sign 符号, 表示正负号 exponent 指数，表示次方数 mantissa 尾数，表示精确度 $(-1)^s_2^e_1.mantissa$ 由于基数是 2，也就无法表达出我们常见的小数，例如: 0.1, 0.2, 0.05。对于无法表达的数只能使用近似值，也就导致 0.1 + 0.2 = 0.30000000000000004。 只要是浮点表示的小数，早晚会遇到精度问题。 对于需要精准小数计算的还是要用[Math.js](https://mathjs.org/)、[decimal.js](https://mikemcl.github.io/decimal.js/)这种专门的库来计算。

## 更多浮点相关的知识

[15 张图带你深入理解浮点数](https://polarisxu.studygolang.com/posts/basic/diagram-float-point/) [IEEE-754 浮点数对应二进制转化](https://baseconvert.com/ieee-754-floating-point) [IEEE-754 浮点数可视化](https://bartaz.github.io/ieee754-visualization/) [IEEE-754 浮点数可视化 float-toy](https://evanw.github.io/float-toy/)
