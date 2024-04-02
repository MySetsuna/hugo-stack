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
    - Rust基础
tags: 
    - Rust
    - Rust语言圣经（Rust Course）
weight: 96
---

## 创建动态数组

### Vec::new()

### vec![]

## 更新Vector

## Vector与其元素共存亡

## 从Vector中读取元素

### 下标索引与.get的区别

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

## 存储不同类型的元素

## Vector常用方法

## Vector的排序

### 整数数组的排序

### 浮点数数组的排序

### 对结构体数组进行排序