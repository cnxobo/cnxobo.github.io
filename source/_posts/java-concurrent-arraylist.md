---
title: Java多线程并发操作ArrayList
tags: []
id: '826'
categories:
  - - Java
  - - logs
date: 2020-12-27 16:56:20
---

给公司一个业务系统做性能优化时，有个地方需要在循环内实现对外交互。有网络IO的地方很容易出现性能瓶颈，就打算通过parallelStream实现并发操作，

```Java
List resultList = new ArrayList();
xxxList.forEach(item -> {
    result = doSomethingWithRemoteServer()
    resultList.add(result)
});
return resultList;
```

如果直接把`forEach`改为`parallelStream().forEach`，就会引发新的问题，因为业务代码里使用了 arraylist.add 方法收集计算结果，ArrayList 是非线程安全的使用一个线程安全的容器。 Java里线程安全的集合容器，可以通过如下方法: `Vector` \* 古老且线程安全的List, 每次扩容一倍空间，而ArrayList扩容50%。 \* 在这个场景下因为集合需要返回上层做额外操作，如果使用`Vector`会有不必要的锁开销，当然这点儿性能影响可以忽略不计，如果不想有额外的锁开销就需要在返回时多了一层转换，把Vector转化为ArrayList。 `CopyOnWriteArrayList` \* 每次添加新元素时创建一个新的List，适合读多写少的场景。该场景基本没并发读的场景，完全没必要使用。 `Collections.synchronizedList` \* 返回一个包装类`SynchronizedList`,对被包装的真实List的所有场景加锁。 其他 \* 因为我这里这个场景比较简单也可以使用`ConcurrentHashMap`等集合容器来实现线程安全。 我现在的这个场景挺适合`Collections.synchronizedList`

```Java
List resultList = new ArrayList();
List syncResultList = Collections.synchronizedList(resultList)

List resultList = new ArrayList();
xxxList.parallelStream().forEach(item -> {
    result = doSomethingWithRemoteServer()
    syncResultList.add(result)
});
return resultList;
```

全部改完之后，又想到我只用到了 arraylist.add 其实只需要同步这一方法就行了。

```Java
List resultList = new ArrayList();
xxxList.parallelStream().forEach(item -> {
    result = doSomethingWithRemoteServer()
    synchronized(resultList){
        resultList.add(result)
    }
});
return resultList;
```

这样没有一句废话的实现了多线程环境下，给ArrayList下添加元素。如果还在学生时代，估计我会直接写出最后这种代码，但是随着工作时间久了，习惯于使用各种工具包来实现各种功能，渐渐的忘记最初的样子, 习惯于把简单的问题复杂化。 Keep it simple and stupid.