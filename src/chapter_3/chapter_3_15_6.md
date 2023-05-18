# 3.15.6 BTreeMap（B 树映射）

Rust 中的 BTreeMap（B 树映射）是一种自平衡的有序映射数据结构，它以 B 树的形式存储键值对。BTreeMap 具有对数级的时间复杂度，这使得它在需要维护键的顺序时非常有效。以下是有关 Rust BTreeMap 的一些详细信息：

## 1. 创建 BTreeMap：

要创建一个新的空 BTreeMap，可以使用 `BTreeMap::new()` 方法。需要导入 `std::collections::BTreeMap` 模块以使用 BTreeMap。

```rust
use std::collections::BTreeMap;

let mut map = BTreeMap::new();
```

## 2. 添加元素：

可以使用 `insert()` 方法向 BTreeMap 中添加键值对。如果键已存在，则此方法将返回 `Some(old_value)`，否则返回 `None`。

```rust
map.insert("one", 1);
map.insert("two", 2);
map.insert("three", 3);
```

## 3. 访问元素：

可以使用 `get()` 方法根据键查找对应的值。此方法返回一个 `Option<&V>` 类型，如果找到键，则返回 `Some(&value)`，否则返回 `None`。

```rust
let value = map.get("one"); // 返回 Option<&V>
```

还可以使用 `get_mut()` 方法获取可变引用。

## 4. 删除元素：

可以使用`remove()` 方法删除 BTreeMap 中的键值对。此方法返回一个 `Option<V>` 类型，如果找到并删除了键值对，则返回 `Some(value)`，否则返回 `None`。

```rust
map.remove("one"); // 删除键为 "one" 的键值对
```

## 5. 遍历元素：

可以使用 `iter()` 方法遍历 BTreeMap 中的所有键值对。遍历顺序按键的顺序进行。`iter_mut()` 方法可用于遍历可变引用。

```rust
for (key, value) in map.iter() {
    println!("{}: {}", key, value);
}
```

## 6. BTreeMap 的长度：

可以使用 `len()` 方法获取 BTreeMap 中的键值对数量。还可以使用 `is_empty()` 方法检查 BTreeMap 是否为空。

## 7. 最小和最大键：

可以使用 `first_key_value()` 和 `last_key_value()` 方法分别获取 BTreeMap 中具有最小和最大键的键值对。这些方法返回一个 `Option<(&K, &V)>` 类型，如果找到键值对，则返回 `Some((&key, &value))`，否则返回 None。

```rust
let min_key_value = map.first_key_value(); // 返回 Option<(&K, &V)>
let max_key_value = map.last_key_value(); // 返回 Option<(&K, &V)>
```

## 8. 范围查询：

可以使用 `range()` 方法查询 BTreeMap 中某个范围内的键值对。例如，可以查询所有键大于等于 "one" 且小于等于 "three" 的键值对：

```rust
for (key, value) in map.range("one".."three") {
    println!("{}: {}", key, value);
}
```
