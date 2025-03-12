---
title: Java-Lambda表达式
published: 2025-03-12
description: 'Lambda表达式的原理及用法'
image: ''
tags: [java, Lambda]
category: 'java'
draft: false 
---
# Lambda表达式的注意点：
1. Lambda表达式可以用来简化匿名内部类的书写
2. Lambda表达式只能用来简化**函数式接口**的匿名内部类的写法
3. 函数式接口，指的是有且仅有一个抽象方法的接口，可以在接口上方加上`@FunctionalInterface`注解

```java
public class Lambda {
    public static void main(String[] args) {
        // 1. 匿名内部类
//        doSport(new Swim() {
//            @Override
//            public void Swimming() {
//                System.out.println("正在游泳");
//            }
//        });
        // 2. Lambda表达式
        doSport(() -> System.out.println("正在游泳"));
    }
    public static void doSport(Swim s) {
        s.Swimming();
    }
}
@FunctionalInterface
interface Swim{
    abstract void Swimming();
}
```

# Lambda表达式的简化写法
省略核心：可推导，可省略。
- 参数类型可以省略
- 如果只有一个参数，()也可以省略
- 如果方法体只有一行，大括号，分号，`return`可以省略，但需要同时省略
```java
List<String> list = new ArrayList<>();

list.add("Nihao");
list.add("HelloWorld");
list.add("Hi");

// 1. 不省略的形式
Collections.sort(list, (String s1,String s2) -> {
    return s2.length() - s1.length();
});

// 2. 省略的形式
Collections.sort(list, (s1,s2) -> s2.length() - s1.length());

System.out.println(list);
```