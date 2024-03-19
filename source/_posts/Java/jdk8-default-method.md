---
title: Java8 default method
tags:
  - java
categories:
  - - Java
date: 2024-02-14 10:30:00
---

Java8 引入"默认方法"的新特性，它允许开发者不破坏现有的接口实现的前提下给接口添加新的方法。它提供了灵活性，允许接口定义实现并在具体类未提供该方法的实现时使用默认实现。这个特性部分实现了多重继承的功能，在接口的演化和代码复用方面都非常实用。

<!--more-->

> Java8 加入了很多方法都是通过默认方法引入的，比如 forEach、stream、parallelStream。

### 默认方法的语法

在接口里实现一个用`default`修饰的方法。

```Java
interface IFoo {
    default void hello() {
        System.out.println("Hello IFoo!");
    }
}
```

#### 多重继承

当我们继承多个接口，而这些接口有相同的方法签名时，该接口需要显示的声明该方法。如果声明就会报错。

```Java
interface IBar {
    default void hello() {
        System.out.println("Hello IBar!");
    }
}
interface IFoobar extends IFoo, IBar {

}
```

这样会报错:

> IFoobar inherits unrelated defaults for hello() from types IFoo and IBar。

可以通过实现这个方法来修正这个错误。

```Java
interface IFoobar extends IFoo, IBar {
    default void hello() {}
}
```

如果想复用某个接口的实现，可以通过`接口名.super.方法名`来实现。

```Java
interface IFoobar extends IFoo, IBar {
    default void hello() {
        IFoo.super.hello();
    }
}
```

### static 方法

Java8 的接口也可以定义`static`方法。

```
interface IBar {
    default void hello() {
        System.out.println("Hello IBar!");
    }

    static void staticHello(){
        System.out.println("Hello static IBar!");
    }
}
```

可以通过`IBar.staticHello()`调用这个方法。要注意的是 static 方法不能继承，不能通过`IFoobar.staticHello()`的方式调用。否则会有如下报错

> Static method may be invoked on containing interface class only
