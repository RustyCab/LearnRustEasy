# 3.16.6 RefCell智能指针

## 1. 使用RefCell

**内部可变性（Interior mutability）** 是Rust中的一个设计模式，它允许在有不可变引用时改变数据，这通常是借用规则所不允许的。RefCell正是为Rust提供内部可变性的智能指针。
在Rust中，当使用mut或者&mut显示的声明一个变量或者引用时，才能修改它们的值。编译器会对此严格检查。

```rust
let mut a = 1u32;
a = 2u32;         // 可以修改a的值
let b = 3u32;
b = 4u32;         //  报错，不允许修改
```

但是当使用RefCell时，可以对其内部包含的内容进行修改，如下：

```rust
use std::cell::RefCell;
fn main() {
    let data = RefCell::new(1);              // data本身是不可变变量
    {
        let mut v = data.borrow_mut();
        *v = 2;                              // 但是却可以对RefCell内部的值进行修改
    }
    println!("data: {:?}", data.borrow());   // 将输出为2
}
```

## 2. 使用RefCell，编译器在运行时检查可变性

在上面的代码中，data本身是一个不可变变量，但是在代码中却可以改变它内部的值（第6行，将值从1改成了2），这就是内部可变性。**当使用RefCell定义变量后，编译器会认为：在编译时这个变量是不可变的，但是在运行时可以得到其可变借用，从而改变其内部的值**。换句话说，**使用RefCell是运行时检查借用规则**。

下面是另一个使用RefCell的例子：

```rust
#[derive(Debug)]
enum List {
    Cons(Rc<RefCell<i32>>, Rc<List>),
    Nil,
}

use crate::List::{Cons, Nil};
use std::cell::RefCell;
use std::rc::Rc;

fn main() {
    let value = Rc::new(RefCell::new(5));

    let a = Rc::new(Cons(Rc::clone(&value), Rc::new(Nil)));

    let b = Cons(Rc::new(RefCell::new(6)), Rc::clone(&a));  //不可变引用
    let c = Cons(Rc::new(RefCell::new(10)), Rc::clone(&a)); //不可变引用

    *value.borrow_mut() += 10; //得到可变借用，然后修改其内部的值

    println!("a after = {:?}", a);
    println!("b after = {:?}", b);
    println!("c after = {:?}", c);
}
```

下面是一个使用RefCell时容易犯错的例子：

```rust
use std::cell::RefCell;
fn main() {
    let data = RefCell::new(1); // data本身是不可变变量
    let mut v = data.borrow_mut(); // 使用可变引用
    *v = 2; // 通过可变引用对内部的值进行修改
    println!("data: {:?}", data.borrow()); // 编译报错：
    //   前面使用了可变引用，第7行这里又使用不可变引用，违背所有权规则
}
```

> ***分析：此处需要注意的是，对于RefCell的可变引用、不可变引用的作用域范围（Rust 1.68.2中），其定义方式还是从定义开始，到花括号前结束。这和普通引用是不一样的，因为在新版编译器中（1.31以后），普通引用的作用域范围变成了从定义开始，到不再使用结束。因此，下面的代码是可以正确的***：


```rust
fn main() {
    let mut a = 5u32;
    let b = &mut a; // b是可变引用
    *b = 6u32;
    // 新版编译器中，b的作用域到这里就结束了

    let c = &a; // c是可变引用，因为b的作用域已经结束，所以这里可以使用不可变引用
    println!("c === {:?}", c);
}
```

## 3. 关于Box、Rc或RefCell的选择总结

- Rc允许相同数据有多个所有者；`Box`和`RefCell`有单一所有者。
- Box允许在编译时执行不可变或可变借用检查；Rc仅允许在编译时执行不可变借用检查；RefCell 允许在运行时执行不可变或可变借用检查。
- 因为RefCell允许在运行时执行可变借用检查，所以可以在即便RefCell自身是不可变的情况下修改其内部的值。
