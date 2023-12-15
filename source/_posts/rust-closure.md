---
title: rust闭包的三种特征FnOnce、FnMut、Fn及其之间的关系
abbrlink: 18e952a5
date: 2023-12-15 22:09:29
tags:
---
>&emsp;&emsp;目前各个技术社区已经有大量的关于rust闭包介绍和使用方法，以及闭包的类型推导的描述，因此在本文中不再对这些部分进行赘述。如果想要了解这些部分，请参照[这篇文档](https://course.rs/advance/functional-programing/closure.html#%E9%97%AD%E5%8C%85%E4%BD%9C%E4%B8%BA%E5%87%BD%E6%95%B0%E8%BF%94%E5%9B%9E%E5%80%BC)。在学习rust闭包的时候，我经常在评论区看到一些关于闭包Fn特征的提问和回答，某些见解正确但不够详尽，甚至某些见解完全歪曲了闭包特征的事实，因此本文旨在尽可能详尽且准确地描述闭包的三种特征和它们之间的关系。

<!-- more -->

## 闭包特征——FnOnce、FnMut、Fn
闭包有三种特征，分别对应的是三种不同捕获变量的方式——`转移所有权`、`可变借用`、`不可变借用`。

### FnOnce
该类型闭包会拿走参数所有权，并且该类型闭包只能运行一次。
```rust
fn fn_once<F>(func: F)
where 
    F: FnOnce(i32) -> i32,
{
    println!("{}", func(1));
    println!("{}", func(2));
}

fn main() {
    let x = 1;
    fn_once(|num| x+num);
}
 ```
**仅**实现了FnOnce特征的闭包会在执行的时候拿走参数的所有权，因此上述代码会报错；在后续中我们将提到如何解决这个问题。
```bash
Compiling playground v0.0.1 (/playground)
error[E0382]: use of moved value: `func`
 --> src/main.rs:6:20
  |
1 | fn fn_once<F>(func: F)
  |               ---- move occurs because `func` has type `F`, which does not implement the `Copy` trait
...
5 |     println!("{}", func(1));
  |                    ------- `func` moved due to this call
6 |     println!("{}", func(2));
  |                    ^^^^ value used here after move
  |
note: this value implements `FnOnce`, which causes it to be moved when called
 --> src/main.rs:5:20
  |
5 |     println!("{}", func(1));
  |                    ^^^^
help: consider further restricting this bound
  |
3 |     F: FnOnce(i32) -> i32 + Copy,
  |                           ++++++

For more information about this error, try `rustc --explain E0382`.
error: could not compile `playground` (bin "playground") due to previous error
```