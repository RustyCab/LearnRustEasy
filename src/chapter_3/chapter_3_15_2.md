# 3.14.2 String（字符串）

Rust 中的 String（字符串）是一个可增长的、UTF-8 编码的字符串类型。它可以存储和处理文本数据。这里是关于 Rust 字符串的一些详细信息：

## 1. 创建 String：

可以使用 `String::new()` 创建一个新的空字符串，或者使用 `String::from()` 从字符串字面量创建字符串。

```rust
let mut s = String::new();
let s = String::from("hello");
```
还可以使用 `to_string()` 方法将基本类型转换为字符串。

```rust
let num = 42;
let s = num.to_string();
```

## 2. 字符串长度：

字符串的长度是其 UTF-8 编码的字节的数量，而不是 Unicode 字符的数量。可以使用 `len()` 方法获取字符串的长度。

```rust
let len = s.len();
```

## 3. 字符串拼接：

有多种方法可以将字符串拼接在一起。例如，可以使用 `+` 运算符或 `format!` 宏。

```rust
let s1 = String::from("hello");
let s2 = String::from("world");
let s3 = s1 + " " + &s2;
```

或者使用 `push_str()` 方法将字符串附加到现有字符串。

```rust
let mut s = String::from("hello");
s.push_str(" world");
```

## 4. 访问字符：

由于 Rust 字符串是 UTF-8 编码的，不能直接使用索引访问单个字符。但是，可以使用迭代器遍历字符串中的字符。

```rust
for c in s.chars() {
    println!("{}", c);
}
```

还可以使用 `bytes()` 方法遍历字节，或者使用 `char_indices()` 方法获取字符及其对应的字节索引。

## 5.字符串切片：

可以使用范围语法创建字符串的切片，它表示原始字符串中一部分的引用。需要确保范围边界位于有效的 UTF-8 字符边界上，否则将导致运行时错误。

```rust
let s = String::from("hello");
let slice = &s[0..4]; // 获取字符串前 4 个字节的切片
```

## 6. 修改字符串：

可以使用 `push()` 方法将一个字符添加到字符串的末尾，或者使用 `insert()`
方法将字符插入指定的字节位置。

```rust
let mut s = String::from("hello");
s.push('!');
s.insert(5, ',');
```

注意，字符串插入操作可能需要 `O(n)` 时间，其中 n 是插入位置之后的字节数。

## 7. 删除字符串中的字符：

可以使用 `pop()` 方法删除并返回字符串末尾的字符，或者使用 `remove()` 方法删除并返回指定字节位置处的字符。

```rust
let mut s = String::from("hello");
s.pop();
s.remove(0);
```

Rust 的字符串处理非常关注编码安全性和性能，因此，与其他编程语言相比，某些操作可能有所不同。然而，这使得 Rust 能够提供安全、高效的字符串操作。
