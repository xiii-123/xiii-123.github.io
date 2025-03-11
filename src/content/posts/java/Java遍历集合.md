---
title: Java遍历集合
published: 2025-03-11
description: 'for-each的细节和lambda表达式遍历'
image: ''
tags: [java, 集合]
category: 'java'
draft: false 
---

# for-each
`for-each`是java中最常见的集合遍历方式，但是遍历时所使用的临时变量究竟是元素本身，还是一个第三方的值？让我们来看下面的代码。
```java
Collection<String> coll = new ArrayList<>();
coll.add("Zhangsan");
coll.add("Lisi");
for (String s : coll) {
    s = "123";
}
for (String s : coll) {
    System.out.println(s);
}
```
输出为：
```bash
Zhangsan
Lisi
```
因为`for-each`中的`s`只是一个临时的第三方变量用来存放元素的值，所以改变`s`并不会改变元素值。

# Lambda表达式遍历
Lambda表达式其实就是一个内部匿名类，让我们看看如何使用内部匿名类遍历集合。
```java
Collection<String> coll = new ArrayList<>();
coll.add("Zhangsan");
coll.add("Lisi");
for (String s : coll) {
    s = "123";
}
coll.forEach(new Consumer<String>() {
    @Override
    public void accept(String s) {
        System.out.println(s);
    }
});
```
上述代码也能遍历集合，为什么？
让我们看看`for-each`的底层原理。
```java
   @Override
    public void forEach(Consumer<? super E> action) {
        Objects.requireNonNull(action);
        final int expectedModCount = modCount;
        @SuppressWarnings("unchecked")
        final E[] elementData = (E[]) this.elementData;
        final int size = this.size;

        for (int i=0; modCount == expectedModCount && i < size; i++) {
            action.accept(elementData[i]);
        }

        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }
```
可以看到，`for-each`在底层就是使用了一个`for`循环，然后对每个元素使用`accept`方法。这就是为什么我们使用内部匿名类实现`Consumer`接口就可以遍历集合。

现在让我们回到Lambda表达式。Lambda表达式的标准形式为：
```java
(ParameterType ParameterName) -> {FunctionBody}
```
于是我们可以使用如下形式遍历集合。
```java
Collection<String> coll = new ArrayList<>();
coll.add("Zhangsan");
coll.add("Lisi");
for (String s : coll) {
    s = "123";
}
coll.forEach((String s) -> {
        System.out.println(s);
    }
);
```
对于比较简单的Lambda表达式，可以简化：
```java
Collection<String> coll = new ArrayList<>();
coll.add("Zhangsan");
coll.add("Lisi");
for (String s : coll) {
    s = "123";
}
coll.forEach(s -> System.out.println(s));
```