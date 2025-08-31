---
# Required fields
title: "Java 反射"
date: 2025-08-30
description: "Java 中反射的使用"
draft: false
categories: ["Java"]
tags: ["Java基础"]
mermaid: false
comments: true
---

「反射」是 Java 中的一个重要机制，用于在运行时动态地控制类。

<!--more-->

## 反射的使用场景

通常我们在写代码的时候就知道需要实例化哪一个类并且运行什么方法，然后通过 `new` 方法创建类的对象，但是有些时候，我们需要在运行时根据不同情况动态实例化不同类。反射就是帮助我们**在运行时改变程序的调用行为**的方法。

## 类加载机制

Java 在编译阶段将 Java 文件编译成 .class 字节码文件，类加载器在类加载阶段将 .class 文件加载进内存，同时**实例化一个 java.lang.Class 对象**。

- 一个 Class 类（.class 文件）对应一个 Class 对象。
- Class 对象中保存了对应类的基础信息，比如类有几个字段（Field）？有几个构造方法（Constructor）？有几个方法（Method）？有哪些注解（Annotation）？等信息。
- 保存类信息对应的 java.lang.Class 对象永远只有一个。

## 操作反射的 Java 类

### 获取 Class 对象的方法

- `Class.forName(包名和类名)`

```java
Class clazz = Class.forName("com.shane.java.reflection.User");
```

这是反射最常用的方法，也是最能体现反射特性的使用方法。

- `类名.class`

```java
Class clazz = User.class;
```

- `类对象.getClass()`

```java
User user = new User();
Class clazz = user.getClass();
```

### 获取 Class 对象的成员变量

我们定义一个 Student 类：

```java
package com.shane.learn.spring.playground;

public class Student {
    static {
        System.out.println("Student static block");
    }

    public static void staticMethod() {
        System.out.println("This is a STATIC METHOD!!");
    }

    public String name;
    int age;
    int height;
    int weight;

    public Student(String name, int age, int height, int weight) {
        this.name = name;
        this.age = age;
        this.height = height;
        this.weight = weight;
    }
}
```

在 main 函数中通过反射获取 Class 对象：

```java
Class<?> cls = Class.forName("com.shane.learn.spring.playground.Student");
```

- 获取**非私有**的成员变量，包含从父类继承的成员变量

```java
Field[] filed = cls.getFields();
for (Field field : filed) {
    System.out.println(field.getName());
}
```

- 获取**所有**成员变量，但是不包含从父类继承的成员变量

```java
filed = cls.getDeclaredFields();
for (Field field : filed) {
    System.out.println(field.getName());
}
```

### 获取 Class 对象中的方法

- `getMethods()`：获取 Class 对象代表的类的所有的非私有方法，数组，**包含**从父类继承而来的方法
- `getDeclaredMethods()`：获取 Class 对象代表的类定义的所有的方法，数组，但是**不包含**从父类继承而来的方法
- `getMethod(methodName)`：获取 Class 对象代表的类的指定方法名的非私有方法
- `getDeclaredMethod(methodName)`：获取 Class 对象代表的类的指定方法名的方法

#### 调用方法

`invoke()` 方法需要传入对象，因此需要先实例化类。

```java
Method method = cls.getDeclaredMethod("message");
Student student = new Student("shane", 22, 169, 120);
method.invoke(student);
```

如果要调用类的静态方法，则直接传入 `null`。

```java
Method method = cls.getDeclaredMethod("staticMethod");
method.invoke(null);
```

## 反射的优缺点

### 优点

自由灵活，不受类的访问权限限制。

### 缺点

安全性低，性能较低，破坏 Java 类的封装性。