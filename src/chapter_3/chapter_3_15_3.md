
# 3.15.3 HashMap（哈希映射）

Rust 中的 HashMap（哈希映射）是一个基于键值对的无序集合，它提供了高效的查找、插入和删除操作。HashMap 使用哈希函数将键映射到相应的存储桶，这使得大部分操作具有 O(1) 的平均时间复杂度。以下是有关 Rust HashMap 的一些详细信息：

## 1. 创建 HashMap：

要创建一个新的空 HashMap，可以使用 `HashMap::new()` 方法。需要导入 `std::collections::HashMap` 模块以使用 HashMap。

```rust
use std::collections::HashMap;

let mut map = HashMap::new();
```

## 2. 插入键值对：

可以使用 `insert()` 方法向 HashMap 中添加键值对。如果使用相同的键插入新值，旧值将被替换。

```rust
map.insert("one", 1);
map.insert("two", 2);
```

## 3. 访问值：

可以使用 `get()` 方法根据键查找值。此方法返回一个 `Option<&V>` 类型，如果找到键，则返回 `Some(&value)`，否则返回 `None`。

```rust
let value = map.get("one"); // 返回 Option<&V>
```

还可以使用 `get_mut()` 方法获取可变引用。

## 4. 遍历键值对：

可以使用 `for` 循环遍历 HashMap 中的所有键值对。

```rust
for (key, value) in &map {
    println!("{}: {}", key, value);
}
```
## 5. 删除键值对：

可以使用 `remove()` 方法根据键删除键值对。此方法返回一个 `Option<V>` 类型，如果找到并删除了键值对，则返回 `Some(value)`，否则返回 `None`。
```rust
map.remove("one"); // 删除键为 "one" 的键值对
```

## 6. 检查键是否存在：

可以使用 `contains_key()` 方法检查 HashMap 中是否存在指定的键。

```rust
let has_key = map.contains_key("one"); // 返回布尔值
```

## 7. 更新值：

可以使用 `entry()` 方法与 `or_insert()` 方法结合，更新 HashMap 中的值或插入新值。

```rust
*map.entry("three").or_insert(3) += 1;
```

## 8. HashMap 的容量和长度：

可以使用 `len()` 方法获取 HashMap 中的键值对数量，使用 `capacity()` 方法获取容量。还可以使用 `shrink_to_fit()` 方法减小容量以匹配当前长度。

## 9. 默认哈希器：

Rust 的 HashMap 默认使用一个加密安全的哈希函数（SipHash），它在防止哈希碰撞攻击方面表现良好，但可能不如其他哈希函数快。可以通过为 HashMap 类型提供自定义的哈希器来改变默认行为。

```rust
use std::collections::HashMap;
use std::hash::BuildHasherDefault;
use twox_hash::XxHash64;

type FastHashMap<K, V> = HashMap<K, V, BuildHasherDefault<XxHash64>>;

let mut map: FastHashMap<&str, i32> = FastHashMap::default();
map.insert("one",  1);
map.insert("two", 2);
```

在这个例子中，我们使用了 `twox_hash` 库中的 `XxHash64` 哈希函数，它通常比默认的 `SipHash` 更快。请注意，使用自定义哈希函数可能会降低安全性，因此要确保在明确了解潜在风险的情况下进行更改。

## 10. 合并两个 HashMap：

可以使用 `extend()` 方法将另一个 HashMap 的键值对添加到当前 HashMap 中。如果存在重复的键，目标 HashMap 中的值将被源 HashMap 中的值覆盖。

```rust
let mut map1 = HashMap::new();
map1.insert("one", 1);
map1.insert("two", 2);

let mut map2 = HashMap::new();
map2.insert("two", 22);
map2.insert("three", 3);

map1.extend(map2); // 将 map2 中的键值对添加到 map1
```

这样，map1 将包含键值对 `"one" -> 1`, `"two" -> 22` 和 `"three" -> 3`。

总之，Rust 中的 HashMap 是一个功能丰富且性能优越的键值对集合，非常适合在需要快速查找和修改操作的场景中使用。
