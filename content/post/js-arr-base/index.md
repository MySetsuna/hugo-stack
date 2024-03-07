---
title: JavaScript数组的常用方法——slice（切片）
description: 数组slice的一些基础用法
date: 2024-03-08T01:10:57+08:00
image: 
math: 
license: 
hidden: false
comments: true
language: 汉语
categories:
    - JS基础
tags: 
    - js
    - js数组方法
---

## slice（切片）

### 基本使用

```ts

/**
 * @param startIndex 切片开始的索引，包括
 * @param endIndex 切片结束的索引，不包括
 *  
 */
Array.prototype.slice(startIndex?:number, endIndex?:number)

const arr = [1, 2 ,3 ,4];

// 1.不传参数，复制数组
const a = arr.slice()  // a = [1, 2, 3, 4]

// 2.传入startIndex，从startIndex开始切片数组
const b = arr.slice(2)  // a = [3, 4]

// 3.传入startIndex，endIndex
// 3.0正常传入
const d = arr.slice(2, 3)  // a = [3]
// 3.1 传入相同值
const c = arr.slice(2, 2)  // a = []
// 3.2 endIndex比startIndex小
const c = arr.slice(2, 1)  // a = []
// 3.4传入负数，从数组尾部往前推
const d = arr.slice(-2)  // a = [3, 4]
const d = arr.slice(-3, -1)  // a = [2， 3]

```

### 在类数组对象上调用 slice()

```ts
// 1. 类数组对象
const arrayLike = {
  length: 3,
  0: 2,
  1: 3,
  2: 4,
  3: 33, // ignored by slice() since length is 3
};
console.log(Array.prototype.slice.call(arrayLike, 1, 3));
// [ 3, 4 ]

// length: 3 length属性在这里被忽略, length之外的3:33也被忽略，因为超出了索引
console.log(Array.prototype.slice.call(arrayLike))
// [2, 3 ,4 ]

// 2.非类数组
const someLike = {
  len: 3,
  0: 2,
  1: 3,
  2: 4,
  3: 33, // ignored by slice() since length is 3
};
// 没有如期调用
console.log(Array.prototype.slice.call(someLike))
// []

const someLike1 = {
  //length: 3
  0: 2,
  1: 3,
  2: 4,
  3: 33, // ignored by slice() since length is 3
};
// 可见需要正确的length
console.log(Array.prototype.slice.call(someLike1))
// []
```

### 使用 slice() 将类数组对象转换为数组

```ts

// 调用 slice() 方法时，会将 this 对象作为第一个参数传入
const slice = Function.prototype.call.bind(Array.prototype.slice);

function list() {
  return slice(arguments);
}

const list1 = list(1, 2, 3); // [1, 2, 3]


```

### 在稀疏数组上使用 slice()

如果源数组是稀疏数组，slice() 方法返回的数组也会是稀疏数组。

```ts

console.log([1, 2, , 4, 5].slice(1, 4)); // [2, empty, 4]

```
