---
title: Java反射
published: 2025-03-15
description: '记录一下反射的用法'
image: ''
tags: [java, 反射]
category: 'java'
draft: false 
---
Java中的反射是指在运行时动态地获取类的信息、创建对象、访问属性、调用方法的能力。反射是Java语言的一个强大特性，它允许程序在运行时进行自我检查，同时也提供了对底层操作的支持。
## 反射涉及的主要类：
1. `Class`类：反射的核心类，用于表示类的相关信息。
2. `Field`类：表示类的成员变量，可以用来获取和设置类之中的字段。
3. `Method`类：表示类的方法，可以用来获取类中的方法信息或者调用类和对象的方法。
4. `Constructor`类：表示类的构造方法，可以用来创建对象。
5. `Modifier`类：提供了关于访问修饰符的信息。
## 主要方法：
- Class类：
  - `forName(String className)`: 获取指定类的Class对象。
  - `newInstance()`: 创建此Class对象所表示的类的一个新实例。
- Field类：
  - `get(Object obj)`: 返回指定对象上此Field表示的字段的值。
  - `set(Object obj, Object value)`: 将指定对象变量上此Field对象表示的字段设置为指定的新值。
- Method类：
  - `invoke(Object obj, Object... args)`: 对带有指定参数的指定对象调用由此Method对象表示的底层方法。
- Constructor类：
  - `newInstance(Object... initargs)`: 使用此Constructor对象表示的构造函数来创建和初始化构造函数的声明类的新实例。
以下是一个使用反射操作BigStar类的成员变量、构造方法和成员方法的示例：
```java
public class BigStar{
    private String name;

    public BigStar(String name) {
        this.name = name;
    }

    public BigStar() {
    }

    @Override
    public String sing(String name) {
        System.out.println(this.name+"正在唱"+name);
        return "谢谢";
    }

    @Override
    public String dance(String name) {
        System.out.println(this.name+"正在跳"+name);
        return "爱你们";
    }

    public String getName() {
        return name;
    }
}
```
```java
public class ReflectionExample {
    public static void main(String[] args) throws Exception {
        // 获取BigStar类的Class对象
        Class<BigStar> bigStarClass = BigStar.class;
        // 获取并操作成员变量
        Field nameField = bigStarClass.getDeclaredField("name");
        nameField.setAccessible(true); // 设置访问权限
        BigStar bigStar = bigStarClass.newInstance();
        nameField.set(bigStar, "周杰伦"); // 设置成员变量name的值
        // 获取并操作构造方法
        Constructor<BigStar> constructor = bigStarClass.getConstructor(String.class);
        BigStar bigStarWithConstructor = constructor.newInstance("邓紫棋");
        // 获取并操作成员方法
        Method singMethod = bigStarClass.getMethod("sing", String.class);
        Method danceMethod = bigStarClass.getMethod("dance", String.class);
        
        // 调用成员方法
        String resultSing = (String) singMethod.invoke(bigStar, "稻香");
        String resultDance = (String) danceMethod.invoke(bigStarWithConstructor, "舞娘");
        
        System.out.println(resultSing); // 输出：谢谢
        System.out.println(resultDance); // 输出：爱你们
    }
}
```
输出：
```bash
周杰伦
周杰伦正在唱稻香
邓紫棋正在跳舞娘
谢谢
爱你们
```

