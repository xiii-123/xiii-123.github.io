---
title: Java数组排序
published: 2025-03-11
description: 'Java提供那些排序方法？'
image: ''
tags: [java, 排序]
category: 'java'
draft: false 
---

在Java中，标准库提供了多种方法用于数组排序，以下是一些常用的方法：
1. `Arrays.sort()`：这是最常用的数组排序方法，可以对基本数据类型数组（如int[], char[]等）和对象数组进行排序。对于对象数组，要求对象实现`Comparable`接口或者提供一个`Comparator`。
   ```java
   import java.util.Arrays;
   int[] arr = {5, 2, 9, 1};
   Arrays.sort(arr); // 对int数组进行排序
   String[] strArr = {"banana", "apple", "orange"};
   Arrays.sort(strArr); // 对String数组进行排序
   ```
2. `Arrays.parallelSort()`：这个方法与`Arrays.sort()`类似，但是它使用并行排序算法，可以更好地利用多核处理器，对于大型数组排序可能更快。
   ```java
   import java.util.Arrays;
   int[] arr = {5, 2, 9, 1};
   Arrays.parallelSort(arr); // 并行排序
   ```
3. `Collections.sort()`：这个方法用于对列表（List）进行排序，而不是数组。但是你可以将数组转换为列表，排序后再转换回数组。
   ```java
   import java.util.ArrayList;
   import java.util.Collections;
   import java.util.List;
   List<Integer> list = new ArrayList<>(Arrays.asList(5, 2, 9, 1));
   Collections.sort(list); // 对List进行排序
   Integer[] arr = list.toArray(new Integer[0]); // 转换回数组
   ```
4. `Arrays.sort(T[] a, Comparator<? super T> c)`：这个方法允许你提供一个自定义的`Comparator`来对对象数组进行排序。
   ```java
   import java.util.Arrays;
   import java.util.Comparator;
   Integer[] arr = {5, 2, 9, 1};
   Arrays.sort(arr, Comparator.reverseOrder()); // 使用自定义Comparator进行降序排序
   ```
   也可以使用`Lambda`表达式自定义`Comparator`。
   ```java
   import java.util.Arrays;
        public class Main {
            public static void main(String[] args) {
                int[][] arr = {
                    {3, 5},
                    {1, 2},
                    {2, 8},
                    {1, 3}
                };

                // 使用Lambda表达式直接在Arrays.sort中定义Comparator
                Arrays.sort(arr, (o1, o2) -> {
                    if (o1[0] != o2[0]) {
                        return Integer.compare(o1[0], o2[0]); // 按第一个元素升序
                    } else {
                        return Integer.compare(o2[1], o1[1]); // 按第二个元素降序
                    }
                });

                // 输出排序后的数组
                for (int[] pair : arr) {
                    System.out.println(Arrays.toString(pair));
                }
            }
        }
    ```

5. `Arrays.stream().sorted()`：使用Java 8的流（Stream）API进行排序，可以链式调用其他流操作，如`map`、`filter`等。
   ```java
   import java.util.stream.Stream;
        public class StreamSort {
            public static void main(String[] args) {
                // 创建一个流
                Stream<Integer> stream = Stream.of(5, 2, 9, 1, 5, 6);

                // 对流进行排序
                Stream<Integer> sortedStream = stream.sorted();

                // 输出排序后的流中的元素
                sortedStream.forEach(System.out::println);
            }
        }

   ```
6. `Arrays.stream().sorted(Comparator)`：与上一个方法类似，但是允许你提供一个自定义的`Comparator`。
   ```java
   import java.util.Arrays;
   import java.util.Comparator;
   Integer[] arr = {5, 2, 9, 1};
   Integer[] sortedArr = Arrays.stream(arr)
                               .sorted(Comparator.reverseOrder())
                               .toArray(Integer[]::new); // 使用自定义Comparator进行降序排序
   ```