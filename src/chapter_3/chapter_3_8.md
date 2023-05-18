# 3.8 复合数据类型

Rust中可以通过结构体或者枚举来构造复杂的数据类型，结构体使用struct 关键字，枚举使用enum关键字。通过结构体或者枚举将多个值组合在一起。

## 3.8.1 结构体

结构体（structure，缩写成 struct）有 3 种类型，使用 struct 关键字来创建：

- 元组结构体（tuple struct），事实上就是具名元组而已。
- 经典的 C 语言风格结构体（C struct）。
- 单元结构体（unit struct），不带字段，在泛型中很有用。

### 1. 定义和实例化

#### 1.1 元组结构体的定义和实例化

下面是定义一个元组结构体，用来表示颜色。

```rust
struct Color(i32, i32, i32);

// 实例化元组结构体
let color = Color(1, 1, 1);
```

使用元组结构体的特点是，给定元组具体的名字，可以和同类元组内的类型的元组做区分。

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

// 实例化元组结构体
let color = Color(1, 1, 1);
let point = Point(1, 1, 1);
```

可以看到Color 和Point虽然元组内的元素都是i32，但是给定了(i32, i32,i32)这个元组两个不同的结构体名称，所以Color和Point不是同一种类型。

```rust
#[derive(Debug, Eq, PartialEq)]
pub struct Color(i32, i32, i32);

#[derive(Debug, Eq, PartialEq)]
pub struct Point(i32, i32, i32);

fn main() {
    assert_eq!(Color(1, 1, 1), Point(1, 1, 1)); // error， 判断两个元组结构体是不是相等，rust将不同类型名称的结构体看作是不一样的结构体，虽然，结构内的元素都是一样的。
}
```

#### 1.2 经典的C结构体的定义和初始化

结构体内的每个字段都是命名字段，使得访问和修改更直观。

```rust
// 定义结构体
struct Person {
    name: String,
    age: u32,
    height: f32,
}

// 结构体实例化
let alice = Person {
    name: String::from("Alice"),
    age: 30,
    height: 1.65,
};
```

>思考：结构体内的数据可以用不同的类型，使用C结构和元组结构体的区别在于，要给结构内的每个字段给予名字来表明其意义。

#### 1.3 单元结构体的定义和初始化

这种结构体没有任何字段。它们通常用于实现特定的行为，而不是表示数据。

```rust
// 定义结构体
struct Dummy;

// 结构体实例化
let dummy = Dummy;
```

### 2. 方法

这三类结构体的方法的定义和使用方式相同，示例如下：

#### 2.1 元组结构体的方法

```rust
struct Color(u8, u8, u8);

impl Color {
    fn print(&self) {
        println!("Color: ({}, {}, {})", self.0, self.1, self.2);
    }
}

let red = Color(255, 0, 0);
red.print(); // 输出：Color: (255, 0, 0)
```

#### 2.2 类C结构体的方法

```rust
struct Person {
    name: String,
    age: u32,
}

impl Person {
    fn greet(&self) {
        println!("Hello, my name is {} and I'm {} years old.", self.name, self.age);
    }
}

let alice = Person {
    name: String::from("Alice"),
    age: 30,
};

alice.greet(); // 输出：Hello, my name is Alice and I'm 30 years old.
```

#### 2.3 单元结构体的方法

```rust
struct Dummy;

impl Dummy {
    fn do_something(&self) {
        println!("Doing something...");
    }
}

let dummy_instance = Dummy;
dummy_instance.do_something(); // 输出：Doing something...
```

## 3.7.2 枚举类型

### 1. 定义和实例化

#### 1.1 枚举的定义

枚举允许在一个数据类型中定义多个变量。这在表示多种可能情况时非常有用。每个枚举成员可以具有关联的数据。

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(u8, u8, u8),
}
```

#### 1.2 枚举的实例化

要创建枚举的实例，需要指定要使用的成员以及其关联的数据（如果有）。
```rust
let msg = Message::Write(String::from("Hello, Rust!"));
```

### 2. 方法

与结构体类似，也可以为枚举定义方法。

```rust
impl Message {
    fn process(&self) {
        match self {
            Message::Quit => println!("Quit"),
            Message::Move { x, y } => println!("Move to ({}, {})", x, y),
            Message::Write(text) => println!("Write: {}", text),
            Message::ChangeColor(r, g, b) => println!("Change color to ({}, {}, {})", r, g, b),
        }
    }
}

// 调用方法
msg.process(); // 输出：Write: Hello, Rust!
```

### 3. 控制流

#### 3.1 Match

Rust 中有一个特殊的控制流结构，叫做 match。它用于匹配枚举成员并针对每个成员执行相应的代码。

```rust
match msg {
    Message::Quit => println!("Quit"),
    Message::Move { x, y } => println!("Move to ({}, {})", x, y),
    Message::Write(text) => println!("Write: {}", text),
    Message::ChangeColor(r, g, b) => println!("Change color to ({}, {}, {})", r, g, b),
}
```

match 表达式需要穷举所有可能的枚举成员，这有助于确保代码的完整性和安全性。在某些情况下，如果不需要处理所有枚举成员，可以使用 `_` 通配符来匹配任何未明确指定的成员。

```rust
match msg {
    Message::Write(text) => println!("Write: {}", text),
    _ => println!("Other message"),
}
```
#### 3.2 `if let`

除了 match 语句之外，Rust 还提供了 `if let` 语法，用于简化某些模式匹配的情况。`if let` 对于只关心单个枚举变体的情况特别有用，这样可以避免编写繁琐的 `match` 语句。`if let` 可以将值解构为变量，并在匹配成功时执行代码块。

下面是一个使用 `Option` 枚举的示例：

```rust
fn main() {
    let some_number = Some(42);

    // 使用 match 语句
    match some_number {
        Some(x) => println!("The number is {}", x),
        _ => (),
    }

    // 使用 if let 语句
    if let Some(x) = some_number {
        println!("The number is {}", x);
    }
}
```

在这个示例中，`if let` 语法让代码更简洁，因为只关心 `Some` 变体。这里还可以使用 `else` 子句处理未匹配的情况。

```rust
fn main() {
    let some_number: Option<i32> = None;

    if let Some(x) = some_number {
        println!("The number is {}", x);
    } else {
        println!("No number found");
    }
}
```

在此示例中，由于 `some_number` 是 `None`，`if let` 语句不匹配，因此将执行 `else` 子句，输出 `"No number found"`。

`if let` 可以与 `Result` 枚举一起使用，以便更简洁地处理错误。当只关心 `Ok` 或 `Err` 变体之一时，这特别有用。以下是一个处理 `Result` 枚举的示例。
定义一个可能返回错误的函数：
```rust
fn divide(numerator: f64, denominator: f64) -> Result<f64, String> {
    if denominator == 0.0 {
        Err(String::from("Cannot divide by zero"))
    } else {
        Ok(numerator / denominator)
    }
}
```

接下来，我们使用 `if let` 处理成功的情况：

```rust
let result = divide(4.0, 2.0);

if let Ok(value) = result {
    println!("The result is {}", value);
}
```

在这种情况下，由于除法操作成功，`if let` 语句将匹配 `Ok` 变体，并输出结果。如果只关心错误情况，可以使用 `if let` 匹配 `Err` 变体：

```rust
let result = divide(4.0, 0.0);

if let Err(error) = result {
    println!("Error: {}", error);
}
```

在这种情况下，由于除法操作失败，`if let` 语句将匹配 `Err` 变体，并输出错误消息。
使用 `if let` 处理 `Result` 可以简化错误处理，特别是当只关心 `Ok` 或 `Err` 变体之一时。然而，请注意，对于更复杂的错误处理逻辑，`match` 语句或 `?` 运算符可能更适合。

### 4. 常用的枚举类型

Rust 标准库中有一些常用的枚举类型，例如 `Option` 和 `Result`。

#### 4.1 Option：表示一个值可能存在或不存在。其成员为 `Some(T)`（其中 `T` 是某种类型）和 `None`。

```rust
fn divide(numerator: f64, denominator: f64) -> Option<f64> {
    if denominator == 0.0f64 {
        None
    } else {
        Some(numerator / denominator)
    }
}

let result = divide(4.0, 2.0);
match result {
    Some(value) => println!("The result is {}", value),
    None => println!("Cannot divide by zero"),
}
```

#### 4.2 `Result`：表示一个操作可能成功或失败。其成员为 `Ok(T)`（其中 `T` 是某种类型）和 `Err(E)`（其中 `E` 是错误类型）。



```rust
fn divide_result(numerator: f64, denominator: f64) -> Result<f64, String> {
    if denominator == 0.0f64 {
        Err(String::from("Cannot divide by zero"))
    } else {
        Ok(numerator / denominator)
    }
}

let result = divide_result(4.0, 0.0);
match result {
    Ok(value) => println!("The result is {}", value),
    Err(error) => println!("Error: {}", error),
}
```


这些枚举类型有助于更安全地处理可能出现的错误情况，避免在代码中使用不安全的值（如空指针）。
