---
title: jdk8-method-reference
tags:
  - Java
categories: 
date: 2024-02-18
---


**方法引用**

**简介**

方法引用是一种在 Java 中使用现有方法来创建简洁的 lambda 表达式的方式。它可以让你直接指向现有方法，而无需重新定义方法体。方法引用可以使代码更加简洁、易读，并减少冗余。

**种类**

方法引用有四种类型：

1. **静态方法引用：** 引用静态方法，例如 `ClassName::staticMethodName`。
2. **特定对象的实例方法引用：** 引用特定对象的实例方法，例如 `containingObject::instanceMethodName`。
3. **任意对象的实例方法引用：** 引用特定类型的任意对象的实例方法，例如 `ContainingType::methodName`。
4. **构造函数引用：** 引用构造函数，例如 `ClassName::new`。

**示例**

1. **静态方法引用：**
    
    Java
    
    ```
    Arrays.sort(rosterAsArray, Person::compareByAge);
    ```

    
    等价于：
    
    Java
    
    ```
    Arrays.sort(rosterAsArray, (a, b) -> Person.compareByAge(a, b));
    ```
    
    Use code with caution. [Learn more](https://bard.google.com/faq#coding)
    
    content_copy
    
2. **特定对象的实例方法引用：**
    
    Java
    
    ```
    ComparisonProvider myComparisonProvider = new ComparisonProvider();
    Arrays.sort(rosterAsArray, myComparisonProvider::compareByName);
    ```
    
    Use code with caution. [Learn more](https://bard.google.com/faq#coding)
    
    content_copy
    
3. **任意对象的实例方法引用：**
    
    Java
    
    ```
    String[] stringArray = { "Barbara", "James", ... };
    Arrays.sort(stringArray, String::compareToIgnoreCase);
    ```
    
    Use code with caution. [Learn more](https://bard.google.com/faq#coding)
    
    content_copy
    
4. **构造函数引用：**
    
    Java
    
    ```
    Set<Person> rosterSet = transferElements(roster, HashSet::new);
    ```
    
    Use code with caution. [Learn more](https://bard.google.com/faq#coding)
    
    content_copy
    

**总结**

方法引用是一种简洁、高效的代码编写方式，可以有效地减少代码量，提高代码的可读性。熟练掌握方法引用可以使你编写出更加简洁、优雅的 Java 代码。