---
title: Java方法引用
published: 2025-03-14
description: '啥是方法引用？有哪些细节？'
image: ''
tags: [java, 方法引用]
category: 'java'
draft: false 
---
1. 什么是方法引用？
    
    把已经存在的方法拿来用，当作函数式接口中抽象方法的方法体。
2. `::`是什么符号？

    方法引用符，类似`.`。
3. 有哪些注意点？
   - 需要有函数式接口
   - 被引用的方法必须已经存在
   - 被引用方法的形参和返回值必须和抽象方法保持一致
   - 被引用方法的功能满足需求

让我们来看一个例子：
```java
    public static void main(String[] args) {
        List<Integer> list = new ArrayList<>();
        list.add(1);
        list.add(3);
        list.add(2);
        Collections.sort(list,Main::subtraction);
        System.out.println(list);
    }

    public static int subtraction(int a, int b){
        return b-a;
    }
```
# 引用静态方法
调用格式：`ClassName::StaticFunction`
```java
public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        Collections.addAll(list,"1","2","3");
        list.stream().map(Integer::parseInt).forEach(s -> System.out.println(s));
    }
```

# 引用成员方法
调用格式：`Object::MemberMethod`
1. 其他类：`其他类对象::方法名`
2. 本类：`this::方法名`（引用处不能是静态方法）
3. 父类：`super::方法名`（引用处不能是静态方法）
```java
public class Main {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        Collections.addAll(list,"涨三","李四","王五","王致和");
        list.stream().filter(new NameFilter()::nameFilter).forEach(s -> System.out.println(s));
    }
}
class NameFilter{
    public boolean nameFilter(String s){
        return s.startsWith("王") && s.length()==3;
    }
}
```

# 引用构造方法 
调用格式：`ClassName::new`
```java
public class Main {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        Collections.addAll(list,"张三-17","李四-18","王五-19","王致和-20");
        list.stream().map(Student::new).forEach(s -> System.out.println(s));
    }
}
class Student{
    private String name;
    private int age;
    public Student(String str){
        String[] strings = str.split("-");
        this.name = strings[0];
        this.age = Integer.parseInt(strings[1]);
    }

    @Override
    public String toString() {
        return this.name+", this year "+this.age+" old";
    }
}
```
# 类名引用成员方法
让我们再来看一段代码：
```java
public class Main {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        Collections.addAll(list,"aaa","bbb","ccc");
        list.stream().map(String::toUpperCase).forEach(s -> System.out.println(s));
    }
}
```
这一段代码乍一看没有问题，但是我们再看一下方法引用的匿名内部类的实现方式，以及`toUpperCase`的具体代码：
```java
public class Main {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        Collections.addAll(list,"aaa","bbb","ccc");
//        list.stream().map(String::toUpperCase).forEach(s -> System.out.println(s));
        list.stream().map(new Function<String, String>() {
            @Override
            public String apply(String s) {
                return s.toUpperCase();
            }
        }).forEach(s -> System.out.println(s));
    }
}
```
```java
  public String toUpperCase() {
        return toUpperCase(Locale.getDefault());
    }
```
`toUpperCase`没有参数，而实现的匿名内部类中的方法有一个形参，这俩对不上啊？

这是因为类名引用成员方法这种方式有一种独有规则：
- 需要有函数式接口
- 被引用的方法必须已经存在
- 被引用方法的形参，**需要和抽象方法的第二个到最后一个参数保持一致**，返回值保持一致。
- 被引用方法的功能满足需求

抽象方法形参的详解:
- 第一个参数:表示被引用方法的调用者，决定了可以引用哪些类中的方法。在Stream流当中，第一个参数一般都表示流里面的每一个数据。假设流里面的数据是字符串，那么使用这种方式进行方法引用，只能引用string这个类中的方法。
- 第二个参数到最后一个参数:跟被引用方法的形参保持一致，如果没有第二个参数，说明被引用的方法需要是无参的成员方法。

**局限性：**该方法和使用`成员::成员方法`引用相比，具有局限性。例如上面的例子中，匿名内部类中的方法第一个参数是`String`，那么就只能引用`String`类的方法。

# 引用数组的构造方法
调用格式：`数组类型[]::new`
```java
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        Collections.addAll(list,"aaa","bbb","ccc");
        String[] array = list.stream().toArray(String[]::new);
        for (String s : array) {
            System.out.println(s);
        }
    }
```