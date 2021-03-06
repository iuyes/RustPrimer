## 并行
理论上并行和语言并没有什么关系，所以在理论上的并行方式，都可以尝试用Rust来实现。本小节不会详细全面地介绍具体的并行理论知识，只介绍用Rust如何来实现相关的并行模式。

Rust的一大特点是，可以保证“线程安全”。而且，没有性能损失。更有意思的是，Rust编译器实际上只有`Send` `Sync`等基本抽象，而对“线程” “锁” “同步” 等基本的并行相关的概念一无所知，这些概念都是由库实现的。这意味着Rust实现并行编程可以有比较好的扩展性，可以很轻松地用库来支持那些常见的并行编程模式。
下面，我们以一个例子来演示一下，Rust如何将线程安全/执行高效/使用简单结合起来的。

在图形编程中，我们经常要处理归一化的问题： 即把一个范围内的值，转换到范围1内的值。比如把一个颜色值255归一后就是1。假设我们有一个表示颜色值的数组要进行归一，用非并行化的方式来处理非常简单，可以自行尝试。下面我们将采用并行化的方式来处理，把数组中的值同时分开给多个线程一起并行归一化处理。

```rust
extern crate rayon;

use rayon::prelude::*;

fn main() {
    let mut colors = [-20.0f32, 0.0, 20.0, 40.0,
        80.0, 100.0, 150.0, 180.0, 200.0, 250.0, 300.0];
    println!("original:    {:?}", &colors);

    colors.par_iter_mut().for_each(|color| {
        let c : f32 = if *color < 0.0 {
                0.0
            } else if *color > 255.0 {
                255.0
            } else {
                *color
            };
        *color = c / 255.0;
    });
    println!("transformed: {:?}", &colors);
}
```

运行结果：
```
original:    [-20, 0, 20, 40, 80, 100, 150, 180, 200, 250, 300]
transformed: [0, 0, 0.078431375, 0.15686275, 0.3137255, 0.39215687, 0.5882353, 0.7058824, 0.78431374, 0.98039216, 1]
```

以上代码是不是很简单。调用`par_iter_mut`获得一个并行执行的具有写权限的迭代器，`for_each`对每个元素执行一个操作。仅此而已。
我们能这么轻松地完成这个任务，原因是我们引入了 [rayon](https://github.com/nikomatsakis/rayon/) 这个库。它把所有的脏活累活都干完了，把清晰安全易用的接口暴露出来给了我们。Rust还可以完全以库的形式，实现异步IO、协程等更加高阶的并行程序开发模式。

为了更深入的加深对Rust并发编程的理解和实践，还安排了一个挑战任务：实现一个Rust版本的MapReduce模式。值得你挑战。
