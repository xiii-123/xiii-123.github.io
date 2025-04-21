---
title: go拾遗
published: 2025-04-17
description: 'Go语言的使用'
image: ''
tags: [Go]
category: 'Go'
draft: false 
---
## rune和byte
Go 语言有两种类型声明方式：一种叫类型定义声明，另一种叫类型别名声明。其中，别名的使用在大型项目重构中作用最为明显，它能解决代码升级或迁移过程中可能存在的类型兼容性问题。而rune 跟 byte 是 Go 语言中仅有的两个类型别名，专门用来处理字符。
让我们来看看二者的源码：
```go
type byte = uint8
type rune = int32
```
rune 是 int32 的别名，而 int32 在 Go 语言中用来表示 Unicode 码点，也就是一个字符。而 byte 是 uint8 的别名，uint8 是 8 位无符号整数，也就是 0~255 之间的整数。在 Go 语言中，一个字符由一个或多个字节表示，而一个字节由 8 位组成。所以，一个字符最多由 4 个字节组成。而 rune 类型可以表示一个字符，所以它比 byte 类型更适合用来处理字符。

通常使用rune来表示UTF-8编码的字符，而byte用来表示二进制的字节。

## unsafe.Pointer和unintptr
在Go语言中，`unsafe.Pointer` 和 `uintptr` 是两种与底层内存操作相关的类型，但它们在引用计数管理方面的行为有所不同。以下通过具体解释和示例代码来详细说明：
---
### 1. **类型定义及用途**
- **`unsafe.Pointer`**：
  - 是一种通用指针类型，可以指向任何类型的值。
  - 主要用于不安全的类型转换，例如将一种类型的指针转换为另一种类型的指针。
  - **与垃圾回收的关系**：`unsafe.Pointer` 可以被垃圾回收器（GC）识别，因此它持有的对象不会被GC回收，直到所有引用都消失。
- **`uintptr`**：
  - 是一种整数类型，可以存储内存地址的数值表示。
  - 主要用于指针算术运算，但不被视为指针类型。
  - **与垃圾回收的关系**：`uintptr` 不被视为指针，因此它不会阻止垃圾回收器回收其指向的对象。
---
### 2. **引用计数的影响**
- **`unsafe.Pointer`**：
  - 因为它被垃圾回收器视为指针，所以它持有的对象会被计入引用计数，从而避免被GC回收。
  - 例如，如果将一个对象转换为 `unsafe.Pointer`，只要该 `unsafe.Pointer` 存在，对象就不会被回收。
- **`uintptr`**：
  - 由于它不是指针类型，垃圾回收器不会将其视为引用，因此不会影响对象的引用计数。
  - 当一个对象被转换为 `uintptr` 后，该对象可能被垃圾回收器回收，因为 `uintptr` 不持有对象的引用。
---
**示例代码**
以下代码通过示例说明两种类型在引用计数管理上的差异：
```go
package main
import (
    "fmt"
    "unsafe"
)
func main() {
    x := 42
    ptr := &x
    // 使用 unsafe.Pointer
    uptr := unsafe.Pointer(ptr)
    fmt.Printf("unsafe.Pointer: %v\n", uptr)
    // 此时 x 的地址被 uptr 引用，x 不会被 GC 回收
    // 使用 uintptr
    addr := uintptr(uptr)
    fmt.Printf("uintptr: %v\n", addr)
    // 此时 x 的地址被转换为整数值，x 可能会被 GC 回收
}
```
- **`unsafe.Pointer`**：
  - `uptr` 仍然指向 `x` 的地址，因此 `x` 不会被垃圾回收器回收。
  
- **`uintptr`**：
  - `addr` 仅存储 `x` 的地址的数值表示，GC 不认为 `addr` 是对 `x` 的引用，因此 `x` 可能被回收。
---

### 3. **使用**
我们先来看看官方对二者的定义：
```go
// ArbitraryType is here for the purposes of documentation only and is not actually
// part of the unsafe package. It represents the type of an arbitrary Go expression.
type ArbitraryType int

type Pointer *ArbitraryType
func Sizeof(x ArbitraryType) uintptr
func Offsetof(x ArbitraryType) uintptr
func Alignof(x ArbitraryType) uintptr

// uintptr is an integer type that is large enough to hold the bit pattern of
// any pointer.
type uintptr uintptr
```

可以看出，Pointer是一个可以指向任意类型的指针，类似于C语言中的void*。uintptr是一个整数类型，可以存储任何指针的位模式，类似于C语言中的uintptr_t。

官方提供了四种 Pointer 支持的场景：

- 任意类型的指针可以转换为一个 Pointer；
- 一个 Pointer 也可以被转为任意类型的指针；
- uintptr 可以被转换为 Pointer;
- Pointer 也可以被转换为 uintptr。

Pointer用以获取任意类型的指针，但是不能进行地址计算。

uintptr用以进行地址计算，但是不能进行指针解引用。

因此将二者结合就有了下面这种常用操作：
```go
f := unsafe.Pointer(&s.f) 
f := unsafe.Pointer(uintptr(unsafe.Pointer(&s)) + unsafe.Offsetof(s.f))

e := unsafe.Pointer(&x[i])
e := unsafe.Pointer(uintptr(unsafe.Pointer(&x[0])) + i*unsafe.Sizeof(x[0]))
```
我们可以通过unsafe.Pointer(&s.f)来获取结构体s中成员f的地址，或者通过unsafe.Pointer(uintptr(unsafe.Pointer(&s)) + unsafe.Offsetof(s.f))来获取结构体s中成员f的地址，再转回unsafe.Pointer以获取对应的元素。