---
title: "Rust学习之路——特征对象"
description: 
date: 2024-03-25T22:42:09+08:00
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
weight: 98
draft: true
---

在上一节中有一段代码无法通过边缘：

```rust
fn returns_summarizable(switch: bool) -> impl Summary {
    if switch {
        Post {
            // ...
        }
    } else {
        Weibo {
            // ...
        }
    }
}
```

其中Post和Weibo都实现了`Summary` 特征，因此上面的函数试图通过返回 impl Summary 来返回这两个类型，但是编译器却无情地报错了，原因是 `impl Trait` 的返回值类型并不支持多种不同的类型返回，那如果我们想返回多种类型，该怎么办？

再来考虑一个问题：现在在做一款游戏，需要将多个对象渲染在屏幕上，这些对象属于不同的类型，存储在列表中，渲染的时候，需要循环该列表并顺序渲染每个对象，在 Rust 中该怎么实现？

使用枚举

```rust
#[derive(Debug)]
enum UiObject {
    Button,
    SelectBox,
}

fn main() {
    let object = [
        UiObject::Button,
        UiObject::SelectBox
    ];

    for o in object {
        draw(o)
    }
}

fn draw(o: UiObject) {
    println!("{:?}", o);
}
```

问题，如果对象集合并不能事先明确的知道？缺失或想要新增枚举类型？

我们无法知道所有的UI对象类型，只是知道：

- UI对象的类型不同
- 需要一个统一的类型来处理这些对象，无论是作为函数参数还是作为列表中的一员
- 需要对每个对象调用draw方法

在拥有继承的语言中，可定义一个名为Component的类，该类上有一个draw方法。其他的类比如Button、Image和SelectBox会从Component派生并因此继承draw方法。它们各自都可以覆盖draw方法来定义自己的行为，但是框架会把所有这些类型当作是Component的实例，并在其上调用draw。不过Rust并没有继承。

## 特征对象定义

为了解决上面的所有问题，Rust引入了一个概念--特征对象。
先定义一个特征

```rust
pub trait Draw {
    fn draw(&self);
}
```

只要实现了Draw特征，就可以调用draw方法来进行渲染。假设有个Button和SelectBox组件实现了Draw特征：

```rust
pub struct Button {
    pub width: u32,
    pub height: u32,
    pub label: String,
}

impl Draw for Button {
    fn draw(&self) {
        // 绘制按钮代码
    }
}

struct SelectBox {
    width: u32,
    height: u32,
    options: Vec<String>,
}

impl Draw for SelectBox {
    fn draw(&self) {
        // 绘制SelectBox的代码
    }
}

//需要一个动态数组来存储这些UI对象
pub struct Screen {
    pub components: Vec<?>
}
```

代码中的`?`: 可以替换为Draw的特征对象，通过`&`引用或者`Box<T>`智能指针的方式来创建特征对象。

tips: Box<T> 包裹的值会被强制分配在堆上

