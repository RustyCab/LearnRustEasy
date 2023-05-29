# 3.23.1 声明宏
声明宏使用macro_rules!定义，是Rust中最常用的宏形式。下面代码中定义Vec时使用的vec!就是一个声明宏：

```rust
fn main() {
    let _v = vec![1, 2, 3];  // 使用声明宏vec！定义一个Vec
}
```

下面的例子演示定义一个声明宏并使用它：

```rust
// 定义一个声明宏my_vec
macro_rules! my_vec {
    ( $( $x:expr ),* ) => {
        {
            let mut temp_vec = Vec::new();
            $(
                temp_vec.push($x);
            )*
            temp_vec
        }
    };
}

fn main() {
    let v = my_vec![1, 2, 3];  // 使用声明宏 my_vec！
    println!("v: = {:?}", v);
}
```
