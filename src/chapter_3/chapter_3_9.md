# 3.9 泛型

泛型是具体类型或者其它属性的抽象代替，用于减少代码的重复。在编写Rust代码时，可以用泛型来表示各种各样的数据类型，等到编译阶段，泛型则被替换成它所代表的的具体的数据类型。

## 3.9.1 函数定义中的泛型

如果没有泛型，当为不同的类型定义逻辑相同的函数时，可能如下：

```rust
fn return_i8(v: i8) -> i8 { v }
fn return_i16(v: i16) -> i16 { v }
fn return_i32(v: i32) -> i32 { v }
fn return_i64(v: i64) -> i64 { v }
fn return_u8(v: u8) -> u8 { v }
fn return_u16(v: u16) -> u16 { v }
fn return_u32(v: u32) -> u32 { v }
fn return_u64(v: u64) -> u64 { v }

fn main() {
    let _a = return_i8(2i8);
    let _b = return_i16(2i16);
    let _c = return_i32(2i32);
    let _d = return_i64(2i64);

    let _e = return_u8(2u8);
    let _f = return_u16(2u16);
    let _g = return_u32(2u32);
    let _h = return_u64(2u64);
}
```

使用泛型后，可以在函数定义时使用泛型，在调用函数的地方指定具体的类型，如下：

```rust
fn return_value<T>(v: T) -> T{ v }

fn main() {
    let _a = return_value(2i8);
    let _b = return_value(2i16);
    let _c = return_value(2i32);
    let _d = return_value(2i64);

    let _e = return_value(2u8);
    let _f = return_value(2u16);
    let _g = return_value(2u32);
    let _h = return_value(2u64);
}
```

## 3.9.2 结构体定义中的泛型

在结构体中使用泛型的示例如下：

```rust
#[derive(Debug)]
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let integer = Point { x: 1, y: 2 };      // Point的两个字段都是整型
    println!("{:#?}", integer);

    let float = Point { x: 0.99, y: 1.99 };   // Point的两个字段都是浮点型
    println!("{:#?}", float);
}
```

也可以像如下方式使用：

```rust
#[derive(Debug)]
struct Point<T, U> {  // Point的两个字段可以指定为不同的类型
    x: T,
    y: U,
}

fn main() {
    let a = Point { x: 1, y: 2.0 };
    println!("{:#?}", a);

    let b = Point { x: 1, y: 1.99 };
    println!("{:#?}", b);
}
```

## 3.9.3 枚举定义中的泛型

标准库的`Option`类型就是使用泛型的枚举类型，其定义如下：

```rust
enum Option<T> {
    Some(T),
    None,
}
```
同样的还有`Result`类型，其定义如下：

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

下面再举一个枚举类型中使用泛型的例子：

```rust
enum Message<T, U> {
    Msg1(u32),
    Msg2(T),
    Msg3(U),
}



fn main() {
    let _msg1: Message<u8, String> = Message::Msg1(1u32);
    let _msg2:Message<u8, String> = Message::Msg2(2u8);
    let _msg3:Message<u8, String> = Message::Msg3("hello".to_string());
}
```

## 3.9.4 方法定义中的泛型

还可以在方法中使用泛型，例子1：
`
```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn get_x(&self) -> &T {
        &self.x
    }
    fn get_y(&self) -> &T {
        &self.y
    }
}

fn main() {
    let p = Point { x: 1, y: 2 };
    println!("p.x = {}", p.get_x());
    println!("p.y = {}", p.get_y());
}
```

方法中的泛型不一定和结构体中的一样，示例如下：

```rust
struct Point<T, U> {
    x: T,
    y: U,
}
impl<T, U> Point<T, U> {
    fn mixup<V, W>(self, other: Point<V, W>) -> Point<T, W> {
        Point {
            x: self.x,
            y: other.y,
        }
    }
}
fn main() {
    let p1 = Point { x: 5, y: 10.4 };
    let p2 = Point { x: "Hello", y: 'c'};
    let p3 = p1.mixup(p2); // 对应的T、U分别是整型和浮点型，V、W则分别是字面字符串和字符类型
    println!("p3.x = {}, p3.y = {}", p3.x, p3.y);
}

```

## 3.9.5 泛型代码的性能

在Rust中，使用泛型并不会造成程序性能上的损失。因为Rust通过在编译时进行泛型代码的单态化来保证效率（就是在编译时，把泛型换成了具体的类型）。单态化是通过填充编译时使用的具体类型，将通用代码转换为特定代码的过程。
