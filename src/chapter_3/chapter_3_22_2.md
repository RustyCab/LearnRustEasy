# 3.22.2 在C中调用Rust
在c中调用Rust，需要把Rust的实现进行封装，封装时主要做以下四点：

- 提供Rust方法、trait方法等公开接口的独立函数。对于泛型函数，需要提供具体某个类型的函数。
- 所有暴露给c的独立函数都要声明成#[no_mangle]。
- 数据结构要转换成和c兼容的结构。如果是自己定义的结构体，需要使用#[repr(c)]。
- 要是用catch_unwind把所有可能产生panic!的代码包裹起来。

下面示例展示在c中调用Rust，其目录结构如下：

![注释](.././assets/53.png)


目录中的foo是Rust代码，用cargo new foo --lib创建，其中src/lib.rs的代码如下：

```rust
#![crate_type = "staticlib"] // 生成静态库
#[no_mangle]  // 暴露给c的函数要声明#[no_mangle]
pub extern fn foo(){
    println!("use rust");
}
```

foo目录下的Cargo.toml内容如下：
```TOML
[package]
name = "foo"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]

[lib]
name = "foo"  # foo编译后对应的库名字
crate-type = ["staticlib"]  # 表示编译为静态库
```

在foo目录下运行命令cargo build编译，在foo/target/debug/目录下会生成静态库libfoo.a。

main.c中的c代码如下：
```C
#include <stdio.h>

extern void foo();  // foo函数为Rust提供的函数
int main()
{
    foo();
    return 0;
}
```

在main.c同级目录运行命令```gcc -o main main.c ./foo/target/debug/libfoo.a -lpthread -ldl```会生成执行文件main，如下图：

![注释](.././assets/54.png)
