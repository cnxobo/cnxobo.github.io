---
title: String.replaceAll 反斜杠StringIndexOutOfBoundsException String index out of range
tags: []
id: '438'
categories:
  - - Java
  - - Something
date: 2015-05-12 23:18:13
---

尝试使用String.replaceAll把斜杠(/)替换成反斜杠()时，一直抛StringIndexOutOfBoundsException异常。 后来意识到写法错误，原代码如下："xobo/org".replaceAll("/", "\\") **正确**的写法是： "xobo/org".replaceAll("/", Matcher.quoteReplacement("\\")) 等价的写法是："xobo/org".replaceAll("/", "\\\\") Java字符串"\\\\"组成了一个正则表达式"\\",而正则表达式"\\"代表着纯文本""。 所以四个反斜杠才能真正表达一个反斜杆的含义。 java.util.regex.Matcher.quoteReplacement(String) 这个方法就是专门消除""及"$"特殊含义的方法。 在使用String.replace的时候就没这个烦恼，两个就好了"\\"，因为在replace方法内对replacement参数调用了Matcher.quoteReplacement方法。