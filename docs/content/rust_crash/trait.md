# Rust 速成课/泛型类型

在开始这次的课程之前呢, 我们先来活动一下手指. 我遇到了一个问题, 我需要写一个函数, 这个函数接收两个数字, 然后返回较大的那个数字.

我不想说大话, 但这实在是太简单了, 我的功力很深厚, 所以, 很快啊, 我就写下了下面这个函数.

```rs
fn largest(a: u32, b: u32) -> u32 {
    if a > b {
        a
    } else {
        b
    }
}
```

这个函数能工作, 但我觉得不太好, 因为它只能比较两个 u32 类型数字的大小. 我现在除了想比较两个 u32 外, 还想比较两个 f32. 有一种可以行的办法, 我们可以定义多个 largest 函数, 让它们分别叫做 largest_u32, largest_f32, 这很简单, 只要变成一个无情的复制粘贴机器, 你还能写出 largest_u64 和 largest_f64. 这能工作, 但不美观. 有没有一种方法, 只使用 largest 这个函数名, 就能同时支持所有类型呢?

答案是有的, Rust 的编译器已经想到了这一点, 只要你使用泛型语法.

```rs
fn largest<T: std::cmp::PartialOrd>(a: T, b: T) -> T {
    if a > b {
        a
    } else {
        b
    }
}

fn main() {
    println!("{}", largest::<u32>(1, 2));
    println!("{}", largest::<f32>(1.0, 2.1));
}
```

除了在函数内使用, 还可以在结构体中使用, 例如

```rs
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let integer = Point { x: 5, y: 10 };
    let float = Point { x: 1.0, y: 4.0 };
}
```

当你发现你需要大量的复制粘贴代码的时候, 就应该考虑使用泛型.
