---
title: JavaBitset
published: 2025-03-10
description: '啥是Bitset，以前没见过啊'
image: ''
tags: [java, Bitset]
category: 'java'
draft: false 
---
`BitSet`类在Java的`java.util`包中，它提供了多种方法来操作位序列，例如设置位、清除位、翻转位、以及进行位运算（如与、或、异或等）。`BitSet`内部通常使用长整型数组来存储位序列，这使得它在空间和时间效率上都表现良好。

`BitSet`的一些常见用途包括：

标志位集合：当需要表示一系列布尔状态时，可以使用`BitSet`来存储这些状态，每个位表示一个状态。
位运算：在进行位级别的运算时，`BitSet`提供了方便的方法来进行与、或、异或等操作。
数据压缩：在某些情况下，可以使用`BitSet`来压缩数据，因为它们只占用实际设置位的存储空间。
索引和过滤：在数据库和搜索引擎中，`BitSet`常用于实现高效的索引和过滤机制。
算法实现：在某些算法中，如布隆过滤器（`Bloom Filter`），`BitSet`是核心数据结构。
使用`BitSet`时需要注意，它不是线程安全的，如果在多线程环境中使用，需要外部同步。

BitSet定义了两个构造方法。

第一个构造方法创建一个默认的对象：
```java
BitSet()
```
第二个方法允许用户指定初始大小。所有位初始化为0。
```java
BitSet(int size)
```
下面的程序说明这个数据结构支持的几个方法：
```java
import java.util.BitSet;
 
public class BitSetDemo {
 
  public static void main(String args[]) {
     BitSet bits1 = new BitSet(16);
     BitSet bits2 = new BitSet(16);
      
     // set some bits
     for(int i=0; i<16; i++) {
        if((i%2) == 0) bits1.set(i);
        if((i%5) != 0) bits2.set(i);
     }
     System.out.println("Initial pattern in bits1: ");
     System.out.println(bits1);
     System.out.println("\nInitial pattern in bits2: ");
     System.out.println(bits2);
 
     // AND bits
     bits2.and(bits1);
     System.out.println("\nbits2 AND bits1: ");
     System.out.println(bits2);
 
     // OR bits
     bits2.or(bits1);
     System.out.println("\nbits2 OR bits1: ");
     System.out.println(bits2);
 
     // XOR bits
     bits2.xor(bits1);
     System.out.println("\nbits2 XOR bits1: ");
     System.out.println(bits2);
  }
}
```
以上实例编译运行结果如下：
```bash
Initial pattern in bits1:
{0, 2, 4, 6, 8, 10, 12, 14}

Initial pattern in bits2:
{1, 2, 3, 4, 6, 7, 8, 9, 11, 12, 13, 14}

bits2 AND bits1:
{2, 4, 6, 8, 12, 14}

bits2 OR bits1:
{0, 2, 4, 6, 8, 10, 12, 14}

bits2 XOR bits1:
{}
```
下面是`Bitmap`的接口：
当然，以下是您提供的BitSet方法描述的Markdown表格格式：
| 序号 | 方法描述                                 |
|------|------------------------------------------|
| 1    | `void and(BitSet set)`                   |
|      | 对此目标位 set 和参数位 set 执行逻辑与操作。     |
| 2    | `void andNot(BitSet set)`                |
|      | 清除此 BitSet 中所有的位，其相应的位在指定的 BitSet 中已设置。 |
| 3    | `int cardinality()`                      |
|      | 返回此 BitSet 中设置为 true 的位数。             |
| 4    | `void clear()`                           |
|      | 将此 BitSet 中的所有位设置为 false。             |
| 5    | `void clear(int index)`                  |
|      | 将索引指定处的位设置为 false。                   |
| 6    | `void clear(int startIndex, int endIndex)`|
|      | 将指定的 startIndex（包括）到指定的 toIndex（不包括）范围内的位设置为 false。 |
| 7    | `Object clone()`                         |
|      | 复制此 BitSet，生成一个与之相等的新 BitSet。       |
| 8    | `boolean equals(Object bitSet)`          |
|      | 将此对象与指定的对象进行比较。                   |
| 9    | `void flip(int index)`                   |
|      | 将指定索引处的位设置为其当前值的补码。             |
| 10   | `void flip(int startIndex, int endIndex)` |
|      | 将指定的 fromIndex（包括）到指定的 toIndex（不包括）范围内的每个位设置为其当前值的补码。 |
| 11   | `boolean get(int index)`                 |
|      | 返回指定索引处的位值。                           |
| 12   | `BitSet get(int startIndex, int endIndex)`|
|      | 返回一个新的 BitSet，它由此 BitSet 中从 fromIndex（包括）到 toIndex（不包括）范围内的位组成。 |
| 13   | `int hashCode()`                         |
|      | 返回此位 set 的哈希码值。                       |
| 14   | `boolean intersects(BitSet bitSet)`      |
|      | 如果指定的 BitSet 中有设置为 true 的位，并且在此 BitSet 中也将其设置为 true，则返回 true。 |
| 15   | `boolean isEmpty()`                      |
|      | 如果此 BitSet 中没有包含任何设置为 true 的位，则返回 true。 |
| 16   | `int length()`                           |
|      | 返回此 BitSet 的"逻辑大小"：BitSet 中最高设置位的索引加 1。 |
| 17   | `int nextClearBit(int startIndex)`       |
|      | 返回第一个设置为 false 的位的索引，这发生在指定的起始索引或之后的索引上。 |
| 18   | `int nextSetBit(int startIndex)`         |
|      | 返回第一个设置为 true 的位的索引，这发生在指定的起始索引或之后的索引上。 |
| 19   | `void or(BitSet bitSet)`                 |
|      | 对此位 set 和位 set 参数执行逻辑或操作。         |
| 20   | `void set(int index)`                    |
|      | 将指定索引处的位设置为 true。                     |
| 21   | `void set(int index, boolean v)`         |
|      | 将指定索引处的位设置为指定的值。                   |
| 22   | `void set(int startIndex, int endIndex)`  |
|      | 将指定的 fromIndex（包括）到指定的 toIndex（不包括）范围内的位设置为 true。 |
| 23   | `void set(int startIndex, int endIndex, boolean v)` |
|      | 将指定的 fromIndex（包括）到指定的 toIndex（不包括）范围内的位设置为指定的值。 |
| 24   | `int size()`                             |
|      | 返回此 BitSet 表示位值时实际使用空间的位数。       |
| 25   | `String toString()`                      |
|      | 返回此位 set 的字符串表示形式。                   |
| 26   | `void xor(BitSet bitSet)`                |
|      | 对此位 set 和位 set 参数执行逻辑异或操作。         |
