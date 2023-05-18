# 3.15.7 BTreeSet（B 树集合）

Rust 中的 BTreeSet（B 树集合）是一种自平衡的有序集合数据结构，它以 B 树的形式存储元素。BTreeSet 具有对数级的时间复杂度，这使得它在需要维护元素顺序时非常有效。以下是有关 Rust BTreeSet 的一些详细信息：

## 1. 创建 BTreeSet：

要创建一个新的空 BTreeSet，可以使用 `BTreeSet::new()` 方法。需要导入 `std::collections::BTreeSet` 模块以使用 BTreeSet。

```rust
use std::collections::BTreeSet;

let mut set = BTreeSet::new();
```

## 2. 添加元素：

可以使用 `insert()` 方法向 BTreeSet 中添加元素。如果元素已存在，则此方法将返回 `false`，否则返回 `true`。

```rust
set.insert(1);
set.insert(2);
set.insert(3);
```

## 3. 检查元素是否存在：

可以使用 `contains()` 方法检查 BTreeSet 中是否存在指定的元素。

```rust
let contains = set.contains(&1); // 返回布尔值
```

## 4. 删除元素：

可以使用 `remove()` 方法删除 BTreeSet 中的元素。此方法返回一个 bool 类型，如果找到并删除了元素，则返回 `true`，否则返回 `false`。

```rust
set.remove(&1); // 删除元素 1
```

## 5. 遍历元素：

可以使用 `for` 循环遍历 BTreeSet 中的所有元素。遍历顺序按元素的顺序进行。

```rust
for element in &set {
    println!("{}", element);
}
```

## 6. BTreeSet 的长度：

可以使用 `len()` 方法获取 BTreeSet 中的元素数量。还可以使用 `is_empty()` 方法检查 BTreeSet 是否为空。

## 7. 集合操作：

BTreeSet 支持一些基本的集合操作，如并集、交集、差集和对称差集。

- 并集（union）：返回一个新的 BTreeSet，包含两个集合中的所有元素。

    ```rust
    let set1: BTreeSet<_> = [1, 2, 3].iter().cloned().collect();
    let set2: BTreeSet<_> = [3, 4, 5].iter().cloned().collect();
    let union: BTreeSet<_> = set1.union(&set2).cloned().collect();
    ```

- 交集（intersection）：返回一个新的 BTreeSet，包含两个集合中共有的元素。

    ```rust
    let intersection: BTreeSet<_> = set1.intersection(&set2).cloned().collect();
    ```

- 差集（difference）：返回一个新的 BTreeSet，包含第一个集合中存在但第二个集合中不存在的元素。

    ```rust
    let difference: BTreeSet<_> = set1.difference(&set2).cloned().collect();
    ```

- 对称差集（symmetric_difference）：返回一个新的 BTreeSet，包含两个集合中唯一的元素（也就是只存在于一个集合中的元素）

    ```rust
    let symmetric_difference: BTreeSet<_> = set1.symmetric_difference(&set2).cloned().collect();
    ```

## 8. 最小和最大元素：

可以使用 `first()` 和 `last()` 方法分别获取 BTreeSet 中的最小和最大元素。这些方法返回一个 `Option<&T>` 类型，如果找到元素，则返回 `Some(&element)`，否则返回 `None`。

```rust
let min_element = set.first(); // 返回 Option<&T>
let max_element = set.last(); // 返回 Option<&T>
```

## 9. 范围查询：

可以使用 `range()` 方法查询 BTreeSet 中某个范围内的元素。例如，可以查询所有大于等于 1 且小于等于 3 的元素：

```rust
for element in set.range(1..=3) {
    println!("{}", element);
}
```

## 10. 清空 BTreeSet：

可以使用 `clear()` 方法删除 BTreeSet 中的所有元素。

```rust
set.clear(); // 清空 BTreeSet
```

总之，Rust 中的 BTreeSet 提供了一种有序集合数据结构，适用于需要维护元素顺序以及执行集合操作的场景。与 BTreeMap 类似，BTreeSet 具有对数级的时间复杂度，这使得它在需要维护元素顺序时非常有效。然而，在需要快速查找、添加和删除操作的场景中，使用基于哈希表的数据结构（如 HashSet）可能更适合。
