---
title: 由SQL where 1=1想到的字符串处理方法
tags: []
id: '246'
categories:
  - - Java
date: 2013-03-25 23:24:17
---

字符串拆成集合简单,集合再合成字符串就有点麻烦. 对于Javascript,一个join方法就可以解决问题,可惜Java的集合没有提供join的方法. 不过强大的Java的三方库提供了类似的功能,例如

//Commons Lang:
org.apache.commons.lang.StringUtils.join(list, conjunction);

//Spring:
org.springframework.util.StringUtils.
    collectionToDelimitedString(list, conjunction);

当然也可以自己写一个实现,常见的写法为

public static String join(Collection c, String delim) {
if (c == null  c.size()==0) {
return "";
}
StringBuilder sb = new StringBuilder();
Iterator it = c.iterator();
int i = 0;
while (it.hasNext()) {
if (i++ > 0) {
sb.append(delim);
}
sb.append(it.next());
}
return sb.toString();
}

总觉得循环中的if很别扭，很想把它干掉,于是有了稍稍优雅的写法:

public static String join(String delimiter, Iterable<? extends Object> objs) {
if (objs == null  !objs.hasNext()) {
return "";
}
Iterator<? extends Object> iter = objs.iterator();
StringBuilder buffer = new StringBuilder();
buffer.append(iter.next());
while (iter.hasNext()) {
buffer.append(delimiter).append(iter.next());
}
return buffer.toString();
}