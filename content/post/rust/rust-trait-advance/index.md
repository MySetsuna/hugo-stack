---
title: "Rust学习之路——深入了解特征"
description: 
date: 2024-04-01T22:33:18+08:00
image: 
math: 
license: 
hidden: false
comments: true
language: 汉语
categories:
    - Rust
tags: 
    - Rust
    - Rust语言圣经（Rust Course）
weight: 97
draft: false
---

## 关联类型

关联类型时在特征定义的语句块中，申明一个自定义类型，这样就可以在特征的方法签名中使用该类型。

```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;
}
```

*`Self`用来指代当前调用者的具体类型，那么`Self::Item`就用来指代该类型实现中定义的`Item`类型*：

```rust
impl Iterator for Counter {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {
        //...
    }
}

fn main() {
    let c = Counter{..}
    c.next()
}
```

上述代码中，`Counter`类型实现了`Iterator`特征，变量`c`是特征`Iterator`的实例，也是next方法的调用者。

为什么不使用泛型:

```rust
pub trait Iterator<Item>{
    fn next(&mut self) -> Option<Item>;
}
```

因为代码的可读性

```rust
// 泛型写法
trait Container<A,B> { 
    fn contains(&self,a: A,b: B) ->bool;
}

fn difference<A,B,C(container:&C) -> i32
    where
        c: Container<A,B> {...}

// 关联类型实现
trait Container{
    type A;
    type B;
    fn copntains(&self, a:Self::A,b:&Self::B) -> bool;
}

fn differnce<C: Container>(container:&:C) {}
```

## 默认泛型类型参数

## 在外部类型上实现外部特征（newtype）

```rust
use std::fmt;

struct Wrapper(Vec<String>);

impl fmt::Display for Wrapper {
    fn fmt(&self, f:&mut fmt::Formatter) -> fmt::Result {
        write!(f, "[{}]", self.0.join(", "))
    }
}

fn main() {
    let w = Wrapper(vec![String::from("hello"), String::from("world")]);
    println!("w = {}", w);
}
```
