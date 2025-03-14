---
title: Java-Stream
published: 2025-03-14
description: 'Java中的Stream有那些细节'
image: ''
tags: [java, Stream]
category: 'java'
draft: false 
---
# 获取Stream流的四种用法
## 1. 单列集合
对于单列集合（如List、Set），可以直接通过调用`.stream()`方法来获取Stream流。
```java
List<String> list = Arrays.asList("a", "b", "c");
Stream<String> stream = list.stream();
```
## 2. 双列集合
对于双列集合（如Map），可以通过`.keySet().stream()`、`.values().stream()`或`.entrySet().stream()`来获取Stream流。
```java
Map<Integer, String> map = new HashMap<>();
map.put(1, "一");
map.put(2, "二");
Stream<Integer> keyStream = map.keySet().stream();
Stream<String> valueStream = map.values().stream();
Stream<Map.Entry<Integer, String>> entryStream = map.entrySet().stream();
```
## 3. 数组
对于数组，可以使用`Arrays.stream(Object[])`来获取Stream流。注意，对于基本类型数组，需要使用专门的stream方法，如`IntStream.of()`。
```java
String[] array = {"a", "b", "c"};
Stream<String> stream = Arrays.stream(array);
int[] intArray = {1, 2, 3};
IntStream intStream = Arrays.stream(intArray);
```
## 4. 零散数据
对于零散数据，可以使用`Stream.of(T... values)`来获取Stream流。

**注意：**`Stream.of`只能放入引用类型数组，基本类型数组需要单独处理。
```java
Stream<String> stream = Stream.of("a", "b", "c");
// 对于基本类型数组，可以这样处理
int[] intArray = {1, 2, 3};
IntStream intStream = Arrays.stream(intArray);
Stream<Integer> streamOfInts = intStream.boxed();
```
# Stream流的方法
## 中间方法
- `filter`：过滤Stream流中的元素。
  
```java
List<String> list = Arrays.asList("a", "b", "c");
list.stream()
    .filter(s -> s.startsWith("a"))
    .forEach(System.out::println); // 输出 "a"
```
- `limit`：限制Stream流中的元素数量。
  
```java
List<String> list = Arrays.asList("a", "b", "c", "d");
list.stream()
    .limit(2)
    .forEach(System.out::println); // 输出 "a" "b"
```
- `skip`：跳过Stream流中的前n个元素。
  
```java
List<String> list = Arrays.asList("a", "b", "c", "d");
list.stream()
    .skip(2)
    .forEach(System.out::println); // 输出 "c" "d"
```
- `distinct`：去除Stream流中的重复元素。
  
```java
List<String> list = Arrays.asList("a", "b", "c", "a");
list.stream()
    .distinct()
    .forEach(System.out::println); // 输出 "a" "b" "c"
```
- `concat`：合并两个Stream流。
  
```java
Stream<String> stream1 = Stream.of("a", "b");
Stream<String> stream2 = Stream.of("c", "d");
Stream.concat(stream1, stream2)
    .forEach(System.out::println); // 输出 "a" "b" "c" "d"
```
- `map`：将每个元素转换成另一个形式或提取信息。`map`有多种变体，如`mapToInt`、`mapToLong`、`mapToDouble`等。
  
```java
List<String> list = Arrays.asList("1", "2", "3");
list.stream()
    .map(Integer::parseInt)
    .forEach(System.out::println); // 输出 1 2 3
```
## 终结方法
- `forEach`：遍历Stream流中的每个元素。
  
```java
List<String> list = Arrays.asList("a", "b", "c");
list.stream()
    .forEach(System.out::println); // 输出 "a" "b" "c"
```
- `count`：计算Stream流中的元素数量。
  
```java
List<String> list = Arrays.asList("a", "b", "c");
long count = list.stream()
    .count(); // count为3
```
- `collect`：将Stream流中的元素收集到一个新的集合中。`collect`有多种变体，如`Collectors.toList()`、`Collectors.toSet()`、`Collectors.toMap()`等。
  
```java
List<String> list = Arrays.asList("a", "b", "c");
List<String> collectedList = list.stream()
    .collect(Collectors.toList());
```
`Collectors.toMap()`的使用要复杂一些，因为我们要实现`KeyMapper`和`ValueMapper`：
```java
public static <T, K, U>
Collector<T, ?, Map<K,U>> toMap(Function<? super T, ? extends K> keyMapper,
                                Function<? super T, ? extends U> valueMapper) {
    return toMap(keyMapper, valueMapper, throwingMerger(), HashMap::new);
}
```
我们可以使用Lambda表达式替代匿名内部类：
```java
public static void main(String[] args) {
    List<String> words = Arrays.asList("apple", "banana", "cherry", "date");

    // 使用Collectors.toMap()将字符串列表转换为Map，其中字符串为键，字符串长度为值
    Map<String, Integer> wordLengths = words.stream()
            .collect(Collectors.toMap(
                    // 键映射函数
                    word -> word,
                    // 值映射函数
                    word -> word.length()
            ));

    // 输出转换后的Map
    wordLengths.forEach((word, length) -> System.out.println(word + " -> " + length));
}
```