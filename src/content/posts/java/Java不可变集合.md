---
title: Java不可变集合
published: 2025-03-13
description: 'Java还可以创建不允许修改的集合'
image: ''
tags: [java, 不可变集合]
category: 'java'
draft: false 
---
# Java8

在Java 8中，创建不可变的List、Set和Map可以通过以下`Collections`类中的相应方法实现：
## 不可变List
```java
import java.util.Collections;
import java.util.List;
import java.util.ArrayList;
public class ImmutableExample {
    public static void main(String[] args) {
        List<String> mutableList = new ArrayList<>();
        mutableList.add("Apple");
        mutableList.add("Banana");
        mutableList.add("Cherry");
        // 创建不可变的List
        List<String> immutableList = Collections.unmodifiableList(mutableList);
        // 尝试修改将抛出UnsupportedOperationException
        // immutableList.add("Date"); // 这行代码会抛出异常
    }
}
```
## 不可变Set
```java
import java.util.Collections;
import java.util.Set;
import java.util.HashSet;
public class ImmutableExample {
    public static void main(String[] args) {
        Set<String> mutableSet = new HashSet<>();
        mutableSet.add("Apple");
        mutableSet.add("Banana");
        mutableSet.add("Cherry");
        // 创建不可变的Set
        Set<String> immutableSet = Collections.unmodifiableSet(mutableSet);
        // 尝试修改将抛出UnsupportedOperationException
        // immutableSet.add("Date"); // 这行代码会抛出异常
    }
}
```
## 不可变HashMap
```java
import java.util.Collections;
import java.util.Map;
import java.util.HashMap;
public class ImmutableExample {
    public static void main(String[] args) {
        Map<String, String> mutableMap = new HashMap<>();
        mutableMap.put("Fruit", "Apple");
        mutableMap.put("Vegetable", "Carrot");
        mutableMap.put("Grain", "Wheat");
        // 创建不可变的Map
        Map<String, String> immutableMap = Collections.unmodifiableMap(mutableMap);
        // 尝试修改将抛出UnsupportedOperationException
        // immutableMap.put("Legume", "Pea"); // 这行代码会抛出异常
    }
}
```
通过`Collections.unmodifiableXXX`方法创建的不可变集合是视图，这意味着如果原始集合被修改，不可变集合也会反映出这些变化。如果需要确保原始集合也不可变，那么在创建不可变视图之后，你不应该再对原始集合进行任何修改。如果你想要创建一个完全独立的不可变集合，那么你需要确保在创建不可变视图之后不再使用原始集合。

# Java9

从Java 9开始，引入了一系列的集合工厂方法，这些方法允许你使用简洁的语法来创建不可变的集合。以下是使用`List.of`, `Set.of`, 和 `Map.of` 方法来创建不可变List、Set和Map的示例代码：
## 不可变List
```java
import java.util.List;
public class ImmutableExample {
    public static void main(String[] args) {
        List<String> immutableList = List.of("Apple", "Banana", "Cherry");
        // 尝试修改将抛出UnsupportedOperationException
        // immutableList.add("Date"); // 这行代码会抛出异常
    }
}
```
## 不可变Set
```java
import java.util.Set;
public class ImmutableExample {
    public static void main(String[] args) {
        Set<String> immutableSet = Set.of("Apple", "Banana", "Cherry");
        // 尝试修改将抛出UnsupportedOperationException
        // immutableSet.add("Date"); // 这行代码会抛出异常
    }
}
```
## 不可变Map
`Map.of`方法可以接受最多10个键值对参数。参数必须是成对的，第一个是键，第二个是值。以下是一个示例：
```java
import java.util.Map;
public class ImmutableMapExample {
    public static void main(String[] args) {
        Map<String, String> immutableMap = Map.of(
            "key1", "value1",
            "key2", "value2",
            "key3", "value3",
            "key4", "value4",
            "key5", "value5",
            "key6", "value6",
            "key7", "value7",
            "key8", "value8",
            "key9", "value9",
            "key10", "value10"
        );
        // 尝试修改将抛出UnsupportedOperationException
        // immutableMap.put("key11", "value11"); // 这行代码会抛出异常
    }
}
```

如果需要创建一个包含超过10个键值对的不可变`HashMap`，或者你想要更灵活地创建映射，你可以使用`Map.ofEntries`方法。以下是一个示例：
```java
import java.util.HashMap;
import java.util.Map;
import java.util.Map.Entry;

public class ImmutableMapExample {
    public static void main(String[] args) {
        // 创建一个可变的HashMap
        Map<String, String> hp = new HashMap<>();
        originalMap.put("key1", "value1");
        originalMap.put("key2", "value2");
        originalMap.put("key3", "value3");

        // 使用Map.ofEntries创建一个不可变的Map
        Map<String, String> immutableMap = Map.ofEntries(hp.entrySet().toArray(new Entry[0]));

        // 输出不可变Map的内容
        System.out.println(immutableMap);

        // 尝试修改将抛出UnsupportedOperationException
        // immutableMap.put("key4", "value4"); // 这行代码会抛出异常
    }
}

```

# Java10
Java 10 引入了 `Map.copyOf` 方法，它可以用来创建一个不可修改的地图，其内容是给定地图的副本。如果给定地图是空的，那么返回的地图将是空的。否则，返回的地图将包含与给定地图相同的键值映射关系，并且也是不可修改的。
以下是如何使用 `Map.copyOf` 方法来创建一个不可变 `HashMap` 的示例：
```java
import java.util.HashMap;
import java.util.Map;
public class ImmutableMapExample {
    public static void main(String[] args) {
        // 创建一个可变的HashMap
        Map<String, String> originalMap = new HashMap<>();
        originalMap.put("key1", "value1");
        originalMap.put("key2", "value2");
        originalMap.put("key3", "value3");
        // 使用Map.copyOf创建一个不可变的HashMap副本
        Map<String, String> immutableMap = Map.copyOf(originalMap);
        // 尝试修改将抛出UnsupportedOperationException
        // immutableMap.put("key4", "value4"); // 这行代码会抛出异常
        // 输出不可变Map的内容
        System.out.println(immutableMap);
    }
}
```

