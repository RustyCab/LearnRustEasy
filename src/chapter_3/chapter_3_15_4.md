# 3.15.4 HashSet（哈希集合）

Rust 中的 HashSet（哈希集合）是一种无序的、不含重复元素的集合。它使用哈希函数将元素映射到相应的存储桶，这使得大部分操作具有 O(1) 的平均时间复杂度。以下是有关 Rust HashSet 的一些详细信息：

## 1. 创建 HashSet：

要创建一个新的空 HashSet，可以使用 `HashSet::new()` 方法。需要导入 `std::collections::HashSet` 模块以使用 HashSet。

```rust
use std::collections::HashSet;

let mut set = HashSet::new();
```

## 2. 添加元素：

可以使用 `insert()` 方法向 HashSet 中添加元素。如果元素已存在，则此方法将返回 `false`，否则返回 `true`。

```rust
set.insert(1);
set.insert(2);
set.insert(3);
```

## 3. 检查元素是否存在：

可以使用 `contains()` 方法检查 HashSet 中是否存在指定的元素。

```rust
let contains = set.contains(&1); // 返回布尔值
```

## 4. 删除元素：

可以使用 `remove()` 方法删除 HashSet 中的元素。此方法返回一个 bool 类型，如果找到并删除了元素，则返回 `true`，否则返回 `false`。

```rust
set.remove(&1); // 删除元素 1
```

## 5. 遍历元素：

可以使用 for 循环遍历 HashSet 中的所有元素。
```rust
for element in &set {
    println!("{}", element);
}
```

## 6. HashSet 的长度：

可以使用 `len()` 方法获取 HashSet 中的元素数量。还可以使用 `is_empty()` 方法检查 HashSet 是否为空。

## 7. 集合操作：

HashSet 支持一些基本的集合操作，如并集、交集、差集和对称差集。

- 并集（union）：返回一个新的 HashSet，包含两个集合中的所有元素。

    ```rust
    let set1: HashSet<_> = [1, 2, 3].iter().cloned().collect();
    let set2: HashSet<_> = [3, 4, 5].iter().cloned().collect();
    let union: HashSet<_> = set1.union(&set2).cloned().collect();
    ```

- 交集（intersection）：返回一个新的 HashSet，包含两个集合中共有的元素。

    ```rust
    let intersection: HashSet<_> = set1.intersection(&set2).cloned().collect();
    ```

- 差集（difference）：返回一个新的 HashSet，包含第一个集合中存在但第二个集合中不存在的元素。

    ```rust
    let difference: HashSet<_> = set1.difference(&set2).cloned().collect();
    ```

- 对称差集（symmetric_difference）：返回一个新的 HashSet，包含两个集合中唯一的元素（也就是只存在于一个集合中的元素）。

    ```rust
    let symmetric_difference: HashSet<_> = set1.symmetric_difference(&set2).cloned().collect();
    ```

## 8. 清空 HashSet：

可以使用 `clear()` 方法删除 HashSet 中的所有元素。

```rust
set.clear(); // 清空 HashSet
```

总的来说，Rust 中的 HashSet提供了一种高效且易于使用的无序集合实现，适用于需要快速查找、添加和删除操作的场景。由于 HashSet 的底层实现基于哈希表，它能够在大部分情况下为这些操作提供 O(1) 的平均时间复杂度。

与 HashMap 类似，Rust 的 HashSet 默认使用一个加密安全的哈希函数（SipHash），以防止哈希碰撞攻击。如果需要更高的性能，可以考虑使用自定义哈希器，但要确保在明确了解潜在风险的情况下进行更改。这是一个使用自定义哈希器的例子
：

```rust
use std::collections::HashSet;
use std::hash::BuildHasherDefault;
use twox_hash::XxHash64;

type FastHashSet<T> = HashSet<T, BuildHasherDefault<XxHash64>>;

let mut set: FastHashSet<i32> = FastHashSet::default();
set.insert(1);
set.insert(2);
```

在这个例子中，我们使用了 `twox_hash` 库中的 `XxHash64` 哈希函数，它通常比默认的 SipHash 更快。
