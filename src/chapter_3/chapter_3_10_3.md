# 3.10.3 常见的trait

Rust 常见的 trait 包括：
- `std::fmt::Display`: 格式化打印用户友好字符串。
- `std::fmt::Debug`: 格式化打印调试字符串。
- `std::cmp::PartialEq`: 比较值相等。
- `std::cmp::PartialOrd`: 比较值顺序。
- `std::cmp::Eq`: 类型完全相等关系。
- `std::cmp::Ord`: 类型完全顺序关系。
- `std::clone::Clone`: 创建类型副本。
- `std::ops::Add`: 定义加法操作。
- `std::ops::Mul`: 定义乘法操作。
- `std::iter::Iterator`: 实现迭代器。

下面分别介绍：

## 1. std::fmt::Display：

```rust
use std::fmt;

struct Person {
    name: String,
    age: u32,
}

impl fmt::Display for Person {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "{} ({} years)", self.name, self.age)
    }
}
```

## 2. std::fmt::Debug：

```rust
#[derive(Debug)]
struct Person {
    name: String,
    age: u32,
}
```

## 3. std::cmp::PartialEq 和 std::cmp::Eq：
```rust
#[derive(PartialEq, Eq)]
struct Point {
    x: i32,
    y: i32,
}
```

## 4. std::cmp::PartialOrd 和 std::cmp::Ord：

```rust
#[derive(PartialOrd, Ord)]
struct Point {
    x: i32,
    y: i32,
}
```

## 5. std::clone::Clone：

```rust
#[derive(Clone)]
struct Point {
    x: i32,
    y: i32,
}
```

## 6. std::ops::Add：

```rust
use std::ops::Add;

struct Point {
    x: i32,
    y: i32,
}

impl Add for Point {
    type Output = Point;

    fn add(self, other: Point) -> Point {
        Point {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
}
```

## 7. std::iter::Iterator：

```rust
struct Counter {
    count: u32,
}

impl Iterator for Counter {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {
        self.count += 1;
        if self.count < 6 {
            Some(self.count)
        } else {
            None
        }
    }
}
```
