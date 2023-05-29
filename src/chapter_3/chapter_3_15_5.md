
# 3.15.5 LinkedList（链表）

Rust 中的 LinkedList（链表）是一种线性数据结构，它由一系列相互连接的节点组成。每个节点都包含一个元素和指向前一个节点和后一个节点的指针。这是有关 Rust LinkedList 的一些详细信息：

## 1. 创建 LinkedList：

要创建一个新的空 LinkedList，可以使用 `LinkedList::new()` 方法。需要导入 `std::collections::LinkedList` 模块以使用 LinkedList。

```rust
use std::collections::LinkedList;

let mut list = LinkedList::new();
```

## 2. 添加元素：

可以使用 `push_front()` 和 `push_back()` 方法将元素添加到链表的开头和结尾。

```rust
list.push_front(1);
list.push_back(2);
```

## 3. 访问元素：

可以使用 `front()` 和 `back()` 方法分别访问链表的第一个和最后一个元素。这些方法返回一个 `Option<&T>` 类型，如果链表不为空，则返回 `Some(&element)`，否则返回 `None`。

```rust
let first_element = list.front(); // 返回 Option<&T>
let last_element = list.back(); // 返回 Option<&T>
```

还可以使用 front_mut() 和 back_mut() 方法获取可变引用。

## 4. 删除元素：

可以使用 `pop_front()` 和 `pop_back()` 方法分别删除并返回链表的第一个和最后一个元素。这些方法返回一个 `Option<T>` 类型，如果链表不为空且成功删除元素，则返回 `Some(element)`，否则返回 `None`。

```rust
list.pop_front(); // 删除并返回第一个元素
list.pop_back(); // 删除并返回最后一个元素
```

## 5. 遍历元素：

可以使用 `iter()` 方法遍历链表中的所有元素。`iter_mut()` 方法可用于遍历可变引用。

```rust
for element in list.iter() {
    println!("{}", element);
}
```

## 6. 链表长度：

可以使用 `len()` 方法获取链表中的元素数量。还可以使用 `is_empty()` 方法检查链表是否为空。

## 7. 清空链表：

可以使用 `clear()` 方法删除链表中的所有元素。

```rust
list.clear(); // 清空链表
```

## 8. 分割链表：

可以使用 `split_off()` 方法在指定索引处分割链表。此操作会将链表分成两个链表，前一个链表包含指定索引之前的元素，后一个链表包含指定索引及之后的元素。

```rust
let mut list1 = LinkedList::new();
list1.push_back(1);
list1.push_back(2);
list1.push_back(3);

let list2 = list1.split_off(1);
```

这样，list1 将包含元素 1，而 list2 将包含元素 2 和 3。

总之，Rust 中的 LinkedList 提供了一种基于节点的线性数据结构，它适用于需要快速插入和删除操作的场景。然而，对于许多其他用途，如随机访问和查找操作，链表通常比数组或向量（Vector）等基于连续内存的数据结构性能差。这是因为链表的元素在内存中是分散存储的，这会导致较差的缓存局部性（cache locality）。而连续内存数据结构在许多情况下可以更有效地利用 CPU 缓存，从而提高性能。

当考虑使用链表时，务必评估其适用性，并与其他数据结构（如 Vector）进行比较。在某些情况下，链表可能是一个很好的选择，特别是当插入和删除操作的性能比访问和查找操作更重要时。然而，在许多场景中，使用基于连续内存的数据结构会带来更好的性能和更简单的代码。
