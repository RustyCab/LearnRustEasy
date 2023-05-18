# 3.14 迭代器

通过迭代器模式可以对一个集合的项进行某些处理。迭代器（iterator）负责遍历集合中的每一项和决定何时结束处理的逻辑。当使用迭代器时，无需重新实现这些逻辑。迭代器是惰性的，即在调用方法使用迭代器之前，它不会有任何效果。

## 3.14.1 Iterator trait

迭代器都实现了`Iterator trait`，该trait定义在Rust标准库中，如下：

```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;
        ...
}
```

`type Item`和`Self::Item`这种用法叫做定义trait的关联类型。Item类型将是迭代器返回元素的类型，`next`方法是`Iterator`实现者被要求定义的唯一方法，`next`一次返回一个元素，当迭代器结束，则返回`None`。

示例如下：

```rust
struct Counter {
    count: u32,
}
impl Counter {
    fn new() -> Counter {
        Counter { count: 0 }
    }
}
impl Iterator for Counter { //Counter实现Iterator trait，是一个迭代器
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

fn main() {
    let mut counter = Counter::new();
    println!("{:?}", counter.next());
    println!("{:?}", counter.next());
    println!("{:?}", counter.next());
    println!("{:?}", counter.next());
    println!("{:?}", counter.next());

    let mut counter1 = Counter::new();
    for item in counter1 { // for循环是迭代器的语法糖，自动对迭代器进行迭代
        println!(" item = {}", item);
    }
}
```

## 3.14.2 IntoIterator trait

如果一个类型实现`IntoIterator trait`，就可以为该类型创建迭代器（换句话说就是可以把该类型转换为迭代器），进而能调用迭代器对应的方法。

通常有三种创建迭代器的方法，分别如下：
- `iter()`方法，创建一个在`&T`上进行迭代的迭代器，即在集合自身引用上迭代的迭代器；
- `iter_mut()`方法，创建一个在`&mut T`上进行迭代的迭代器，即集合自身可变引用上迭代的迭代器；
- `into_iter()`方法，创建一个在`T`上迭代的迭代器，即移动了自身所有权的迭代器。

Vec就实现了`IntoIterator trait`，所以它可以转换为迭代器，使用示例如下：

```rust
fn main() {
    // 1、----使用iter()示例------
    let v1 = vec![1, 2, 3];
    let v1_iter = v1.iter();   //得到迭代器
    //iter()产生的迭代器，使用迭代器不会改变每个元素，只是对每个元素的引用
    for val in v1_iter {
        println!("Got: {}", val);
    }
    println!("v1: {:?}", v1);

    // 2、----使用iter_mut()示例------
    let mut v2 = vec![1, 2, 3];
    let v2_iter = v2.iter_mut(); // 得到迭代器
    //iter_mut()产生的迭代器，使用迭代器可能会改变每个元素
    for val in v2_iter {
        if *val > 1 {
            *val = 1;
        }
        println!("Got: {}", val);
    }
    println!("v2: {:?}", v2);

    // 3、----使用into_iter()示例------
    let v3 = vec![1, 2, 3];
    let v3_iter = v3.into_iter(); // 得到迭代器
    //into_iter()产生的迭代器，使用迭代器后无法再使用原来的集合v3
    for val in v3_iter {
        println!("Got: {}", val);
    }
    // println!("v3: {:?}", v3); //此行将打开编译会出错
}
```

`IntoIterator trait`和`Iterator trait`的关系：
- `Iterator`就是迭代器的trait，实现了该trait就是迭代器；
- `IntoIterator trait`则是表示可以转换为迭代器，如果一个类型实现了`IntoIterator trait`，则它可以调用`iter()`、`iter_mut()`、`into_iter()`转换为迭代器。

## 3.14.3 迭代器消费器

迭代器通过`next`方法来消费一个项。下面为直接使用`next`方法的示例：

```rust
fn main() {
    let v1 = vec![1, 2, 3];
    let mut v1_iter = v1.iter();
    if let Some(v) = v1_iter.next() {
        println!("{}", v); //1
    }
}
```

`Iterator trait`有一系列由标准库提供的默认实现的方法，有些方法调用了`next`方法，这些调用`next`方法的方法被称为消费适配器。

下面是一个使用消费是适配器`sum`方法的例子：

```rust
fn main() {
    let v1 = vec![1, 2, 3];
    let v1_iter = v1.iter();
    let total: i32 = v1_iter.sum(); //调用消费适配器sum来求和
    println!("total = {:?}", total);
}
```

## 3.14.4 迭代器适配器

`Iterator trait`中定义了一类方法，可以把将当前迭代器变为不同类型的迭代器，这类方法就是迭代器适配器。

```rust
fn main() {
    let v1: Vec<i32> = vec![1, 2, 3];
    //下面的 v1.iter().map(|x| x + 1) 创建了一个新的迭代器
    let v2: Vec<_> = v1.iter().map(|x| x + 1).collect(); //v2 = vec![2, 3, 4]
    println!("total = {:?}", v1);
    println!("total = {:?}", v2);
}
```

下面的代码也是使用迭代器适配器：

```rust
fn main() {
    let v1: Vec<i32> = vec![1, 11, 5, 34, 2, 10];
    let v2: Vec<i32> = v1.into_iter().filter(|x| *x > 5).collect();
    // println!("v1 = {:?}", v1); // 此行打开将报错，因为上面创建的迭代器是将原来的所有权移到新的迭代器中了
    println!("v2 = {:?}", v2);
}
```

## 3.14.5 自定义迭代器

自定义迭代器示例如下：

```rust
struct Counter {
    count: u32,
}
impl Counter {
    fn new() -> Counter {
        Counter { count: 0 }
    }
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
fn main() {
    let counter = Counter::new();
    for item in counter { // 实现Iterator trait
        println!(" item = {}", item);
    }
}
```
