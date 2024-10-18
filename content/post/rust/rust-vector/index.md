---
title: "Rust学习之路——动态数组Vector"
description: 
date: 2024-04-02T23:15:11+08:00
image: 
math: 
license: 
hidden: false
comments: true
draft: true
language: 汉语
categories:
    - Rust
tags: 
    - Rust
    - Rust语言圣经（Rust Course）
weight: 0
---

## 创建动态数组

### Vec::new()

使用 Vec::new 创建动态数组是最 rusty 的方式，它调用了 Vec 中的 new 关联函数：

```rust
let v: Vec<i32> = Vec::new();
```

这里，v 被显式地声明了类型 `Vec<i32>`，这是因为 Rust 编译器无法从 Vec::new() 中得到任何关于类型的暗示信息，因此也无法推导出 v 的具体类型，但是当你向里面增加一个元素后，一切又不同了：

```rust
let mut v = Vec::new();
v.push(1);
// Vec<i32>
```

如果预先知道要存储的元素个数，可以使用 Vec::with_capacity(capacity) 创建动态数组，这样可以避免因为插入大量新数据导致频繁的内存分配和拷贝，提升性能

### vec![]

还可以使用宏 vec! 来创建数组，与 Vec::new 有所不同，前者能在创建同时给予初始化值：

```rust
let v =vec![1, 2, 3];
```

## 更新Vector

```rust
let mut v = Vec::new();
v.push(1);
```

与其它类型一样，必须将 v 声明为 mut 后，才能进行修改。

## Vector与其元素共存亡

跟结构体一样，Vector 类型在超出作用域范围后，会被自动删除：

```rust
{
    let v = vec![1, 2, 3];

    // ...
} // <- v超出作用域并在此处被删除
```

当 Vector 被删除后，它内部存储的所有内容也会随之被删除。目前来看，这种解决方案简单直白，但是当 Vector 中的元素被引用后，事情可能会没那么简单。

## 从Vector中读取元素

读取指定位置的元素有两种方式可选：

- 通过下标索引访问。
- 使用 get 方法。

```rust
let v = vec![1, 2, 3];

let third: &i32 = &v[2];
println!("第三个元素是 {}", third);

match v.get(2) {
    Some(third) => println!("第三个元素是 {third}"),
    None() => println("没有第三元素"),
}
```

和其它语言一样，集合类型的索引下标都是从 0 开始，&v[2] 表示借用 v 中的第三个元素，最终会获得该元素的引用。而 v.get(2) 也是访问第三个元素，但是有所不同的是，它返回了 Option<&T>，因此还需要额外的 match 来匹配解构出具体的值。

### 下标索引与.get的区别

这两种方式都能成功的读取到指定的数组元素，既然如此为什么会存在两种方法？何况 .get 还会增加使用复杂度，这就涉及到数组越界的问题了，让我们通过示例说明：

```rust
let v = vec![1, 2, 3, 4, 5];

let does_not_exist = &v[100];
let does_not_exist = v.get(100);
```

运行以上代码，&v[100] 的访问方式会导致程序无情报错退出，因为发生了数组越界访问。 但是 v.get 就不会，它在内部做了处理，有值的时候返回 Some(T)，无值的时候返回 None，因此 v.get 的使用方式非常安全。

既然如此，为何不统一使用 v.get 的形式？因为实在是有些啰嗦，Rust 语言的设计者和使用者在审美这方面还是相当统一的：简洁即正义，何况性能上也会有轻微的损耗。

既然有两个选择，肯定就有如何选择的问题，答案很简单，当你确保索引不会越界的时候，就用索引访问，否则用 .get。例如，访问第几个数组元素并不取决于我们，而是取决于用户的输入时，用 .get 会非常适合，天知道那些可爱的用户会输入一个什么样的数字进来！

## 同时借用多个数组元素

```rust
let mut v = vec![1, 2, 3, 4, 5];

let first = &v[0];

v.push(6);

println!("The first element is: {first}");

$ cargo run
Compiling collections v0.1.0 (file:///projects/collections)
error[E0502]: cannot borrow `v` as mutable because it is also borrowed as immutable 无法对v进行可变借用，因此之前已经进行了不可变借用
--> src/main.rs:6:5
|
4 |     let first = &v[0];
|                  - immutable borrow occurs here // 不可变借用发生在此处
5 |
6 |     v.push(6);
|     ^^^^^^^^^ mutable borrow occurs here // 可变借用发生在此处
7 |
8 |     println!("The first element is: {}", first);
|                                          ----- immutable borrow later used here // 不可变借用在这里被使用

For more information about this error, try `rustc --explain E0502`.
error: could not compile `collections` due to previous error
```
原因在于：数组的大小是可变的，当旧数组的大小不够用时，Rust 会重新分配一块更大的内存空间，然后把旧数组拷贝过来。这种情况下，之前的引用显然会指向一块无效的内存，这非常 rusty —— 对用户进行严格的教育。

## 迭代遍历Vector中的元素

如果想要依次访问数组中的元素，可以使用迭代的方式去遍历数组，这种方式比用下标的方式去遍历数组更安全也更高效（每次下标访问都会触发数组边界检查）：

```rust
let v = vec![1, 2, 3];
for i in &v {
    println!("{i}");
}
```

也可以在迭代过程中，修改 Vector 中的元素：

```rust
let mut v = vec![1, 2, 3];
for i in &mut v{
    *i += 10
}
```

## 存储不同类型的元素

## Vector常用方法

## Vector的排序

### 整数数组的排序

### 浮点数数组的排序

### 对结构体数组进行排序