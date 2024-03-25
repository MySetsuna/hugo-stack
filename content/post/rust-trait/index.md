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
weight: 99
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

## 函数中返回 impl Trait

可以通过`impl Trait`来说明一个函数返回了实习了指定Trait的类型

```rust
fn returns_summarizable() -> impl Summary {
    Weibo {
        username: String::from("sunface"),
        content: String::from(
            "m1 max太厉害了，电脑再也不会卡",
        )
    }
}
```

注意，这里只能让函数调用者知道，函数的返回值实现了`Summary`这个Trait，却没有告知调用者，返回值的具体类型。
这种写法在返回类型特别复杂，无法明确定义的情况下特别有用，但是要注意的是：
***返回值只能是一种具体类型***，例如

```rust
fn returns_summarizable(switch: bool) -> impl Summary {
    if switch {
        Post {
            title: String::from (
                "Penguins win the Stanley Cup Championship!",
            ),
            author: String::from("Iceburgh"),
            content: String::from(
                "The Pittsburgh Penguins once again are then best
                hockey team in the NHL.",
            )
        }
    } else {
        Weibo {
            username: String::from("horse_ebooks"),
            content: String::from(
                "of course,as you probaly alread knoew,people",
            )
        }
    }
}
```

以上代码无法通过编译，因为返回了不同的类型。如果想实现返回不同的类型，需要使用下一章节中的<a href="/p/rust-trait-object/">**特征对象**</a>

## 修复上一节中largest函数

上一节例子中编译报错问题 // todo

```rust
fn largest<T>(list: &[T]) -> T {
    let mut largest = list[0];

    fro &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }
    largest
}

fn main() {
    let number =_list =vec![34, 58, ]
}


error[E0369]: binary operation `>` cannot be applied to type `T` // 无法在 `T` 类型上应用`>`运算符
 --> src/main.rs:5:17
  |
5 |         if item > largest {
  |            ---- ^ ------- T
  |            |
  |            T
  |
help: consider restricting type parameter `T` // 考虑使用以下的特征来约束 `T`
  |
1 | fn largest<T: std::cmp::PartialOrd>(list: &[T]) -> T {
  |             ^^^^^^^^^^^^^^^^^^^^^^

```

在 largest 函数体中我们想要使用大于运算符（>）比较两个 T 类型的值。这个运算符是标准库中特征 `std::cmp::PartialOrd` 的一个默认方法。

``` rust
fn largest<T: PartialOrd>(list: &[T]) -> T {}
```

会出现新的错误：

```rust
error[E0508]: cannot move out of type `[T]`, a non-copy slice
 --> src/main.rs:2:23
  |
2 |     let mut largest = list[0];
  |                       ^^^^^^^
  |                       |
  |                       cannot move out of here
  |                       help: consider using a reference instead: `&list[0]`

error[E0507]: cannot move out of borrowed content
 --> src/main.rs:4:9
  |
4 |     for &item in list.iter() {
  |         ^----
  |         ||
  |         |hint: to prevent move, use `ref item` or `ref mut item`
  |         cannot move out of borrowed content
```

错误的核心是 cannot move out of type [T], a non-copy slice，原因是 T 没有实现 Copy 特性，因此我们只能把所有权进行转移，毕竟只有 i32 等基础类型才实现了 Copy 特性，可以存储在栈上，而 T 可以指代任何类型（严格来说是实现了 PartialOrd 特征的所有类型）。

因此，为了让 T 拥有 Copy 特性，我们可以增加特征约束：

```rust
fn largest<T: PartialOrd + Copy>(list: &[T]) -> T {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest(&number_list);
    println!("The largest number is {}", result);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest(&char_list);
    println!("The largest char is {}", result);
}
```

## 通过derive派生特征

在本书中，形如#[derive(Debug)]的代码，这种是一种特征派生语法，被 derive 标记的对象会自动实现对应的默认特征代码，继承相应的功能。

Debug 特征，它有一套自动实现的默认代码，当你给一个结构体标记后，就可以使用 println!("{:?}", s) 的形式打印该结构体的对象。

Copy 特征，它也有一套自动实现的默认代码，当标记到一个类型上时，可以让这个类型自动实现 Copy 特征，进而可以调用 copy 方法，进行自我复制。

derive 派生出来的是 Rust 默认给我们提供的特征，在开发过程中极大的简化了自己手动实现相应特征的需求。

### 用于开发者输出的 Debug

Debug 特征可以让指定对象输出调试格式的字符串，通过在 {} 占位符中增加 :? 表明，例如println!("show you some debug info: {:?}", MyObject);.

Debug 特征允许以调试为目的来打印一个类型的实例，所以程序员可以在执行过程中看到该实例的具体信息。

例如，在使用 assert_eq! 宏时， Debug 特征是必须的。如果断言失败，这个宏就把给定实例的值打印出来，这样程序员就能看到两个实例为什么不相等。

### 等值比较的 PartialEq 和 Eq

PartialEq 特征可以比较一个类型的实例以检查是否相等，并开启了 == 和 != 运算符的功能。

派生的 PartialEq 实现了 eq 方法。当 PartialEq 在结构体上派生时，只有所有 的字段都相等时两个实例才相等，同时只要有任何字段不相等则两个实例就不相等。当在枚举上派生时，每一个成员都和其自身相等，且和其他成员都不相等。

例如，当使用 assert_eq! 宏时，需要比较一个类型的两个实例是否相等，则 PartialEq 特征是必须的。

Eq 特征没有方法, 其作用是表明每一个被标记类型的值都等于其自身。 Eq 特征只能应用于那些实现了 PartialEq 的类型，但并非所有实现了 PartialEq 的类型都可以实现 Eq。浮点类型就是一个例子：浮点数的实现表明两个非数字（ NaN ，not-a-number）值是互不相等的。

例如，对于一个 HashMap<K, V> 中的 key 来说， Eq 是必须的，这样 HashMap<K, V> 就可以知道两个 key 是否一样。

### 次序比较的 PartialOrd 和 Ord

Clone 特征用于创建一个值的深拷贝（deep copy），复制过程可能包含代码的执行以及堆上数据的复制。查阅 通过 Clone 进行深拷贝获取有关 Clone 的更多信息。

派生 Clone 实现了 clone 方法，当为整个的类型实现 Clone 时，在该类型的每一部分上都会调用 clone 方法。这意味着类型中所有字段或值也必须实现了 Clone，这样才能够派生 Clone 。

例如，当在一个切片（slice）上调用 to_vec 方法时， Clone 是必须的。切片只是一个引用，并不拥有其所包含的实例数据，但是从 to_vec 中返回的 Vector 需要拥有实例数据，因此， to_vec 需要在每个元素上调用 clone 来逐个复制。因此，存储在切片中的类型必须实现 Clone。

Copy 特征允许你通过只拷贝存储在栈上的数据来复制值(浅拷贝),而无需复制存储在堆上的底层数据。查阅 通过 Copy 复制栈数据 的部分来获取有关 Copy 的更多信息。

实际上 Copy 特征并不阻止你在实现时使用了深拷贝，只是，我们不应该这么做，毕竟遵循一个语言的惯例是很重要的。当用户看到 Copy 时，潜意识就应该知道这是浅拷贝，复制一个值会非常快。

当一个类型的内部字段全部实现了 Copy 时，你就可以在该类型上派上 Copy 特征。 一个类型如果要实现 Copy 它必须先实现 Clone ，因为一个类型实现 Clone 后，就等于顺便实现了 Copy 。

总之， Copy 拥有更好的性能，当浅拷贝足够的时候，就不要使用 Clone ，不然会导致你的代码运行更慢，对于性能优化来说，一个很大的方面就是减少热点路径深拷贝的发生。

### 固定大小的值映射的 Hash

Hash 特征允许你使用 hash 函数把一个任意大小的实例映射到一个固定大小的值上。派生 Hash 实现了 hash 方法，对某个类型进行 hash 调用，其实就是对该类型下每个字段单独进行 hash 调用，然后把结果进行汇总，这意味着该类型下的所有的字段也必须实现了 Hash，这样才能够派生 Hash。

例如，在 HashMap<K, V> 上存储数据，存放 key 的时候， Hash 是必须的。

### 默认的Default

Default 特征会帮你创建一个类型的默认值。 派生 Default 意味着自动实现了 default 函数。 default 函数的派生实现调用了类型每部分的 default 函数，这意味着类型中所有的字段也必须实现了 Default，这样才能够派生 Default 。

Default::default 函数通常结合结构体更新语法一起使用，这在第五章的 结构体更新语法 部分有讨论。可以自定义一个结构体的一小部分字段而剩余字段则使用 ..Default::default() 设置为默认值。

例如，当你在 Option<T> 实例上使用 unwrap_or_default 方法时， Default 特征是必须的。如果 Option<T> 是 None 的话, unwrap_or_default 方法将返回 T 类型的 Default::default 的结果。

## 调用方法需要引入特征

在一些场景中，使用 as 关键字做类型转换会有比较大的限制，因为你想要在类型转换上拥有完全的控制，例如处理转换错误，那么你将需要 TryInto：

```rust
use std::convert::TryInto;

fn main() {
  let a: i32 = 10;
  let b: u16 = 100;

  let b_ = b.try_into()
            .unwrap();

  if a < b_ {
    println!("Ten is less than one hundred.");
  }
}
```

上面代码中引入了 std::convert::TryInto 特征，但是却没有使用它，可能有些同学会为此困惑，主要原因在于如果你要使用一个特征的方法，那么你需要将该特征引入当前的作用域中，我们在上面用到了 try_into 方法，因此需要引入对应的特征。

但是 Rust 又提供了一个非常便利的办法，即把最常用的标准库中的特征通过 std::prelude 模块提前引入到当前作用域中，其中包括了 std::convert::TryInto，你可以尝试删除第一行的代码 use ...，看看是否会报错。

## 几个综合例子

### 为自定义类型实现 + 操作

在 Rust 中除了数值类型的加法，String 也可以做加法，因为 Rust 为该类型实现了 std::ops::Add 特征，同理，如果我们为自定义类型实现了该特征，那就可以自己实现 Point1 + Point2 的操作:

```rust
use std::ops::Add;

// 为Point结构体派生Debug特征，用于格式化输出
#[derive(Debug)]
struct Point<T: Add<T, Output = T>> { //限制类型T必须实现了Add特征，否则无法进行+操作。
    x: T,
    y: T,
}

impl<T: Add<T, Output = T>> Add for Point<T> {
    type Output = Point<T>;

    fn add(self, p: Point<T>) -> Point<T> {
        Point{
            x: self.x + p.x,
            y: self.y + p.y,
        }
    }
}

fn add<T: Add<T, Output=T>>(a:T, b:T) -> T {
    a + b
}

fn main() {
    let p1 = Point{x: 1.1f32, y: 1.1f32};
    let p2 = Point{x: 2.1f32, y: 2.1f32};
    println!("{:?}", add(p1, p2));

    let p3 = Point{x: 1i32, y: 1i32};
    let p4 = Point{x: 2i32, y: 2i32};
    println!("{:?}", add(p3, p4));
}
```

### 自定义类型的打印输出

在开发过程中，往往只要使用 #[derive(Debug)] 对我们的自定义类型进行标注，即可实现打印输出的功能：

```rust
#[derive(Debug)]
struct Point{
    x: i32,
    y: i32
}
fn main() {
    let p = Point{x:3,y:3};
    println!("{:?}",p);
}
```

但是在实际项目中，往往需要对我们的自定义类型进行自定义的格式化输出，以让用户更好的阅读理解我们的类型，此时就要为自定义类型实现 std::fmt::Display 特征：

```rust
#![allow(dead_code)]

use std::fmt;
use std::fmt::{Display};

#[derive(Debug,PartialEq)]
enum FileState {
  Open,
  Closed,
}

#[derive(Debug)]
struct File {
  name: String,
  data: Vec<u8>,
  state: FileState,
}

impl Display for FileState {
   fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
     match *self {
         FileState::Open => write!(f, "OPEN"),
         FileState::Closed => write!(f, "CLOSED"),
     }
   }
}

impl Display for File {
   fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
      write!(f, "<{} ({})>",
             self.name, self.state)
   }
}

impl File {
  fn new(name: &str) -> File {
    File {
        name: String::from(name),
        data: Vec::new(),
        state: FileState::Closed,
    }
  }
}

fn main() {
  let f6 = File::new("f6.txt");
  //...
  println!("{:?}", f6);
  println!("{}", f6);
}
```
