---
title: Java泛型
published: 2025-03-11
description: '以前还不知道泛型有这么多用法'
image: ''
tags: [java, 泛型]
category: 'java'
draft: false 
---
泛型允许在定义类、接口和方法时使用类型参数（type parameters）。泛型提供了编译时类型检查，可以确保类型安全，同时避免了强制类型转换。

**主要优点**：

1. 类型安全：在编译时检测到非法的类型，避免运行时错误。

2. 消除强制类型转换：使用泛型后，不再需要显式的类型转换。

3. 提高代码可读性：通过泛型，代码的意图更加清晰。

# java 中泛型标记符：

- E - Element (在集合中使用，因为集合中存放的是元素)
- T - Type（Java 类）
- K - Key（键）
- V - Value（值）
- N - Number（数值类型）
- ？ - 表示不确定的 java 类型

# 泛型方法
泛型方法允许一个方法接受不同类型的参数。根据传递给泛型方法的参数类型，编译器适当地处理每一个方法调用。
定义泛型方法的规则巴拉巴拉太多，就不介绍了，直接上代码。
```java
public class GenericMethodTest
{
   // 泛型方法 printArray                         
   public static < E > void printArray( E[] inputArray )
   {
      // 输出数组元素            
         for ( E element : inputArray ){        
            System.out.printf( "%s ", element );
         }
         System.out.println();
    }
 
    public static void main( String args[] )
    {
        // 创建不同类型数组： Integer, Double 和 Character
        Integer[] intArray = { 1, 2, 3, 4, 5 };
        Double[] doubleArray = { 1.1, 2.2, 3.3, 4.4 };
        Character[] charArray = { 'H', 'E', 'L', 'L', 'O' };
 
        System.out.println( "整型数组元素为:" );
        printArray( intArray  ); // 传递一个整型数组
 
        System.out.println( "\n双精度型数组元素为:" );
        printArray( doubleArray ); // 传递一个双精度型数组
 
        System.out.println( "\n字符型数组元素为:" );
        printArray( charArray ); // 传递一个字符型数组
    } 
}
```

## 有界类型参数
之前都不知道，Java中的泛型还可以定义类型的种类范围，例如我只想要接受`Numbers`类及其子类作为参数就可以使用有界类型参数。

要声明一个有界的类型参数，首先列出类型参数的名称，后跟extends关键字，最后紧跟它的上界。
```java
public class MaximumTest
{
   // 比较三个值并返回最大值
   public static <T extends Comparable<T>> T maximum(T x, T y, T z)
   {                     
      T max = x; // 假设x是初始最大值
      if ( y.compareTo( max ) > 0 ){
         max = y; //y 更大
      }
      if ( z.compareTo( max ) > 0 ){
         max = z; // 现在 z 更大           
      }
      return max; // 返回最大对象
   }
   public static void main( String args[] )
   {
      System.out.printf( "%d, %d 和 %d 中最大的数为 %d\n\n",
                   3, 4, 5, maximum( 3, 4, 5 ) );
 
      System.out.printf( "%.1f, %.1f 和 %.1f 中最大的数为 %.1f\n\n",
                   6.6, 8.8, 7.7, maximum( 6.6, 8.8, 7.7 ) );
 
      System.out.printf( "%s, %s 和 %s 中最大的数为 %s\n","pear",
         "apple", "orange", maximum( "pear", "apple", "orange" ) );
   }
}
```

# 泛型类
泛型类的声明和非泛型类的声明类似，除了在类名后面添加了类型参数声明部分，就不过多介绍，直接上代码。
```java
public class Box<T> {
   
  private T t;
 
  public void add(T t) {
    this.t = t;
  }
 
  public T get() {
    return t;
  }
 
  public static void main(String[] args) {
    Box<Integer> integerBox = new Box<Integer>();
    Box<String> stringBox = new Box<String>();
 
    integerBox.add(new Integer(10));
    stringBox.add(new String("菜鸟教程"));
 
    System.out.printf("整型值为 :%d\n\n", integerBox.get());
    System.out.printf("字符串为 :%s\n", stringBox.get());
  }
}
```

# 类型通配符
类型通配符一般是使用 `?` 代替具体的类型参数。例如 `List<?>` 在逻辑上是 `List<String>,List<Integer>` 等所有 `List<具体类型实参>` 的父类。
```java
import java.util.*;
 
public class GenericTest {
     
    public static void main(String[] args) {
        List<String> name = new ArrayList<String>();
        List<Integer> age = new ArrayList<Integer>();
        List<Number> number = new ArrayList<Number>();
        
        name.add("icon");
        age.add(18);
        number.add(314);
 
        getData(name);
        getData(age);
        getData(number);
       
   }
 
   public static void getData(List<?> data) {
      System.out.println("data :" + data.get(0));
   }
}
```
 因为 `getData()` 方法的参数是 `List<?>` 类型的，所以 `name`，`age`，`number` 都可以作为这个方法的实参。

 类型通配符上限通过形如`List`来定义，如此定义就是通配符泛型值接受`Number`及其下层子类类型。
 ```java
 import java.util.*;
 
public class GenericTest {
     
    public static void main(String[] args) {
        List<String> name = new ArrayList<String>();
        List<Integer> age = new ArrayList<Integer>();
        List<Number> number = new ArrayList<Number>();
        
        name.add("icon");
        age.add(18);
        number.add(314);
 
        //getUperNumber(name);//1
        getUperNumber(age);//2
        getUperNumber(number);//3
       
   }
 
   public static void getData(List<?> data) {
      System.out.println("data :" + data.get(0));
   }
   
   public static void getUperNumber(List<? extends Number> data) {
          System.out.println("data :" + data.get(0));
       }
}
```
在 `//1` 处会出现错误，因为 `getUperNumber()` 方法中的参数已经限定了参数泛型上限为 `Number`，所以泛型为 `String` 是不在这个范围之内，所以会报错。

类型通配符下限通过形如 `List<? super Number>` 来定义，表示类型只能接受 `Number` 及其上层父类类型，如 `Object` 类型的实例。