---
title: "Rust学习之路——特征（Trait）"
description: 
date: 2024-03-11T22:36:29+08:00
image: 
math: 
license: 
hidden: false
comments: true
language: 汉语
categories:
    - Rust基础
tags: 
    - Rust
    - Rust语言圣经（Rust Course）
---

—— 本文内容从<a href="https://course.rs/basic/trait/trait.html">《Rust语言圣经（Rust Course）》</a>中归纳

## 特征 Trait

```rust

fn add<T: std::ops::Add<Output = T>>(a:T, b:T) -> T {
    a + b
}

```

Trait 定义了一组可以被共享的行为，只要实现了特征，你就能实现这组行为。

## 定义 Trait

定义特征是把一些方法组合在一起，目的是定义一个实现某些目标所必需的行为的合集。

```rust

pub trait Summary {
    fn summarize(&self) -> String;
}

```

1. 这里使用trait关键字来声明一个特征，Summary是特证名。在大括号中定义了该特征的所有方法。

2. 特征只定义行为看起来是什么样的，而不定义行为具体是什么样的。

3. 实现这个特征的类型都需要具体实现该特征的相应方法。

## 为类型实现特征

特征只定义行为，需要为类型实现具体的特征，定义具体行为。

```rust

pub trait Summary {
    fn summarize(&self) -> String;
}
pub struct Post {
    pub title: String, // 标题
    pub author: String, // 作者
    pub content: String, // 内容
}

impl Summary for Post {
    fn summarize(&self) -> String {
        format!("文章{}, 作者是{}", self.title, self.author)
    }
}

pub struct Weibo {
    pub username: String,
    pub content: String
}

impl Summary for Weibo {
    fn summarize(&self) -> String {
        format!("{}发表了微博{}", self.username, self.content)
    }
}

fn main() {
    let post = Post{title: "Rust语言简介".to_string(),author: "Sunface".to_string(), content: "Rust棒极了!".to_string()};
    let weibo = Weibo{username: "sunface".to_string(),content: "好像微博没Tweet好用".to_string()};

    println!("{}",post.summarize());
    println!("{}",weibo.summarize());
}

```

### 特征定义与实现的位置（孤儿规则）

！！重要原则（孤儿原则）
**若为类型A实现特征T，则A或T至少有一个需要时在当前作用域中定义的**

这可以确保他人编写的代码不会破坏你的代码

### 默认实现

默认实现不要求重载Trait内的方法

```rust
pub trait Summary {
    fn summarize(&self) -> String {
        String::from("(Read more...)")
    }
}

impl Summary for Post {}

impl Summary for Weibo {
    fn summarize(&self) -> String {
        format!("{}发表了微博{}", self.username, self.content)
    }
}

// Summary for Post: (Read more...)
// Summary for Weibo :sunface发表了微博好像微博没Tweet好用


```

默认实现可以调用相同特征中的其他方法，哪怕这些方法没有默认实现

```rust
// 定义
pub trait Summary {
    fn summarize_author(&self) -> String;

    fn summarize(&self) -> String {
        format!("(Read more from {}...)", self.summarize_author())
    }
}
//调用
impl Summary for Weibo {
    fn summarize_author(&self) -> String {
        format!("@{}", self.username)
    }
}
println!("1 new weibo: {}", weibo.summarize());

```

## 使用特征作为函数参数

特征作为函数参数

```rust
pb fun notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
}
```

```impl Summary```,顾名思义，指实现了Summary特征的item参数。

## 特征约束（trait bound）

```impl [Trait Name]``` 实际上是一个语法糖，完整形式是：

```rust
pub fn notify<T: Summary>(item: &T){
    println!("Breaking news!", item.summarize());
}
```

完整的书写形式：```T: [Trait Name]```，被称为**特征约束**。

```impl [Trait Name]``` 适用于简单场景，

```rust
pub fn notify(item1: &impl Summary, item2: &impl Summary) {}
```

```T: [Trait Name]```可以进一步约束参数必须类型相同

```rust
pub fn notify<T:Summary>(item: &T, item2: &T){}
```

### 多重约束

```rust
// 语法糖
pub fn notify(item: &(impl Summary + Display)) {}

// 特征约束
pub fn notify<T: Summary + Display>(item: &T) {}
```

### Where 约束

```rust
// 特征约束变多，签名变得复杂
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {}

// 通过where 简化
fn some_fucntion<T, U>(t: &T, u: &U) -> i32
    where T: Display + Clone,
          U: Clone + Debug
{}
```

### 使用特征约束有条件地实现方法或特征

指定类型 + 指定特征的条件下去实现方法

```rust
use std::fmt::Display;

struct Pair<T>{
    x:T,
    y:T,
}

impl<T> Pair<T>{
    fn new(x:T, y:Y) -> Self {
        Self {
            x,
            y,
        }
    }
}

// 只有T同时实现了Display + PartialOrd得Pair<T>才有cmp_display方法
impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        }else{
            println!("The largest member is y = {}", self.y);
        }
    }
}
```

### 有条件地实现特征

标准库为任何实现Display特征的类型实现了ToString特征：

```rust
impl<T: Display> ToString for T {
    //...
}

//任何实现了Display的特征的类型调用由ToString定义的to_string方法。

let s = 3.to_string();
```

未完...

