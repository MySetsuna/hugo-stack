---
title: "数据结构第一讲（一）——什么是算法"
description: 
date: 2024-03-23T00:50:15+08:00
image: 
math: 
license: 
hidden: false
comments: true
draft: true
---

## 定义

- 算法 （Algorithm）
  - 一个有限指令集
  - 接受一些输入（有些情况下不需要输入）
  - 产生输出
  - 一定在有限步骤之后终止
  - 每一条指令必须
    - 有充分明确的目标，不可以有歧义
    - 计算机能处理的范围之内
    - 描述不应依赖于任何一种计算机语言以及具体的实现手段

## 例一：选择排序算法的伪码描述

```ts
const SelectionSort(List: number[], N: number) {
    for (let i = 0; i < N; i++) {
        //从List[i]到List[N-1]中找最小元，并将其位置赋给MinPosition：
        const MinPosition = ScanForMin(List, i ,N-1);
        //将未排序部分的最小元换到有序部分的最后位置
        Swap(List[i],List[MinPosition]);
    }
}
```

抽象 ——
    List到底是数组还是链表（虽然看上去很想数组）？
    Swap用函数实现还是用宏去实现？

## 什么是好的算法？

- `空间复杂度S(n)`——根据算法写成的程序在执行时占用存储单元的长度。这个长度往往预示输入数据的规模有段。空间复杂度过高的算法可能导致使用的内存超限，造成程序非正常中断。

- `时间复杂度T(n)`——根据算法写成的程序在执行时耗费时间的长度。这个长度往往也与输入数据的规模有关。时间复杂度过高的低效算法可能导致我们在有生之年都等不到运行结果。

### 例二

```ts
const PrintN = (N: number) => {
    if(N){
        PrintN(N - 1);
        console.log(N)
    }
}

PrintN(100000)
在内存中保存十万个函数的执行状态

S(N) = C·N

算法复杂度随N线性增长

for循环不会多次调用函数，因此内存值是固定的，只涉及变量和for循环的使用。
```

### 例三

题目：
$$
f(x) = a_0 +a_1x +...+ a_{n -1 }x^{n-1}+a_nx^n
$$

```ts
const f = (n: number, a:number[], x:number) => {
    let p = a[0];
    for(let i = 1;i<=n;i++){
        p+=(a[i] * Math.pow(x,i));
    }
    return p;
}
```

转化为结合律
$$
f(x)=a_0+x(a_1 + x(...(a_{n-1}+x(a_n))...))
$$

```ts
const f = (n: number, a:number[], x:number) => {
    let p = a[n];
    for(i = n;i>0li--){
        p = a[i-1] + x*p;
    }
    return p;
}

```

计算机中，加减法计算量很低，主要看乘除法的计算次数:

- 方法一，乘法次数为$(1+2+...+n)=(n^2+n)/2$次乘法 
  - 复杂度 $T(n)=C_{1}n^2+C_2n$

- 方法二，做了n次循环，每次循环中一次乘法 `n次乘法`！
  - 复杂度 $T(n)=C×n$

### 总结

- 在分析一般算法的效率时，我们经常关注下面两种复杂度
  - 最坏情况复杂度$T_{worst}(n)$
  - 平均复杂度T_{avg}$T_{avg}(n)$