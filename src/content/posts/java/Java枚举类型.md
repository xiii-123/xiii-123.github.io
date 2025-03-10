---
title: Java枚举类型
published: 2025-03-10
description: 'Java枚举类型有那些你不知道的用法'
image: ''
tags: [java, 枚举]
category: 'java'
draft: false 
---

# 枚举类的实现

每个枚举都是通过 Class 在内部实现的，且所有的枚举值都是 public static final 的。

以上的枚举类 Color 转化在内部类实现：
```java
class Color
{
     public static final Color RED = new Color();
     public static final Color BLUE = new Color();
     public static final Color GREEN = new Color();
}
```

# values(), ordinal() 和 valueOf() 方法
enum 定义的枚举类默认继承了 java.lang.Enum 类，并实现了 java.lang.Serializable 和 java.lang.Comparable 两个接口。

values(), ordinal() 和 valueOf() 方法位于 java.lang.Enum 类中：

values() 返回枚举类中所有的值。
ordinal()方法可以找到每个枚举常量的索引，就像数组索引一样。
valueOf()方法返回指定字符串值的枚举常量。
```java
enum Color 
{ 
    RED, GREEN, BLUE; 
} 
  
public class Test 
{ 
    public static void main(String[] args) 
    { 
        // 调用 values() 
        Color[] arr = Color.values(); 
  
        // 迭代枚举
        for (Color col : arr) 
        { 
            // 查看索引
            System.out.println(col + " at index " + col.ordinal()); 
        } 
  
        // 使用 valueOf() 返回枚举常量，不存在的会报错 IllegalArgumentException 
        System.out.println(Color.valueOf("RED")); 
        // System.out.println(Color.valueOf("WHITE")); 
    } 
}
```
输出结果：
```
RED at index 0
GREEN at index 1
BLUE at index 2
RED
```

# 枚举类成员
枚举跟普通类一样可以用自己的变量、方法和构造函数，构造函数只能使用 private 访问修饰符，所以外部无法调用。

枚举既可以包含具体方法，也可以包含抽象方法。 如果枚举类具有抽象方法，则枚举类的每个实例都必须实现它。
```java
enum Color 
{ 
    RED, GREEN, BLUE; 
  
    // 构造函数
    private Color() 
    { 
        System.out.println("Constructor called for : " + this.toString()); 
    } 
  
    public void colorInfo() 
    { 
        System.out.println("Universal Color"); 
    } 
} 
  
public class Test 
{     
    // 输出
    public static void main(String[] args) 
    { 
        Color c1 = Color.RED; 
        System.out.println(c1); 
        c1.colorInfo(); 
    } 
}
```
输出：
```
Constructor called for : RED
Constructor called for : GREEN
Constructor called for : BLUE
RED
Universal Color
```
当定义一个枚举类型时，每个枚举常量在枚举类型被加载时都会被实例化一次。这意味着枚举类型的构造函数会在类加载时为每个枚举常量调用一次。

在上述代码中，枚举Color有三个常量：RED、GREEN和BLUE。当运行程序时，JVM会加载Color枚举类型，并为每个枚举常量调用一次私有构造函数。这就是为什么你会看到每个颜色常量的构造函数都被调用了。

以下是代码执行时的输出顺序和原因：
1. `Constructor called for : RED`
   - 当`Color`枚举类型被加载时，`RED`是第一个枚举常量，因此它的构造函数首先被调用。
2. `Constructor called for : GREEN`
   - 接着，第二个枚举常量`GREEN`的构造函数被调用。
3. `Constructor called for : BLUE`
   - 最后，第三个枚举常量`BLUE`的构造函数被调用。

