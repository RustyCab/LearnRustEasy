# 3.16.3 Deref trait

## 1. 通过*使用引用背后的值

常规引用是一个指针类型，包含了目标数据存储的内存地址。对常规引用使用 `*` ，可以通过解引用的方式获取到内存地址对应的数据值，示例如下：

```rust
fn main() {
    let x = 5;
    let y = &x;

    assert_eq!(5, x);
    // assert_eq!(5, y);   //编译错误，必须通过*才能使用y引用的值
    assert_eq!(5, *y);
}
```

## 2. 通过*使用智能指针背后的值

对于智能指针，也可以通过`*`使用其背后的值，示例如下：

```rust
fn main() {
    let x = 5;
    let y = Box::new(x);
    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

## 3. 使用Deref

实现`Deref trait`允许我们重载解引用运算符`*`。通过为类型实现`Deref trait`，类型可以被当做常规引用来对待。简单来说，如果类型A实现了`Deref trait`，那么就可以写如下代码：

```rust
    let a: A = A::new();
    let b =  &a;
    let c = *b;   //对A实现了Deref trait，所以可以对A类型解引用
```

下面的代码定义一个`MyBox`类型，并为其实现`Deref trait`：

```rust
use std::ops::Deref;
struct MyBox<T>(T);
impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}
impl<T> Deref for MyBox<T> {
    //为MyBox实现Deref trait
    type Target = T;
    fn deref(&self) -> &T {
        //注意：此处返回值是引用，因为一般并不希望解引用获取MyBox<T>内部值的所有权
        &self.0
    }
}
fn main() {
    let x = 5;
    let y = MyBox::new(x);
    assert_eq!(5, x);
    assert_eq!(5, *y); //实现Deref trait后即可解引用，使用*y实际等价于 *(y.deref())
}
```

## 4. 函数和方法的隐式Deref强制转换

对于函数和方法的传参，Rust 提供了隐式转换：Deref 转换。若一个类型实现了 Deref 特征，那它的引用在传给函数或方法时，会根据参数签名来决定是否进行隐式的 Deref 转换，例如：

```rust
use std::ops::Deref;
struct MyString(String);
impl Deref for MyString { // MyString类型实现了Deref trait
    type Target = str;
    fn deref(&self) -> &str {
        self.0.as_str()
    }
}

fn print(s: &str) {
    println!("s: {:?}", s);
}
fn main() {
    let y = MyString("Hello".to_string());
    print(&y);  //此处发生隐式转换，&y从&MyString自动转换为&str类型，转换由&触发，过程如下：
                //  &y隐式转换过程: &MyString -> &str
}
```

上面代码的关键点为：

- `MyString`实现了Deref特征，可以在需要时自动被转换为`&str`类型；
- `&y`是一个`&MyString`类型，当它被传给`print`函数时，自动通过`Deref`转换成了`&str`；
- 必须使用`&y`的方式来触发Deref(仅引用类型的实参才会触发自动解引用)。

下面为调用方法时发生的隐式自动转换的例子：

```rust
use std::ops::Deref;
struct MyType(u32);
impl MyType {
    fn to_u32(&self) -> u32 {
        self.0
    }
}

struct MyBox(MyType);
impl Deref for MyBox {
    // MyString类型实现了Deref trait
    type Target = MyType;
    fn deref(&self) -> &MyType {
        &self.0
    }
}

fn main() {
    let mb = MyBox(MyType(5u32));
    let _a = mb.to_u32(); // 此行发生隐式转换，转换过程分析如下：
                          // let _a = mb.to_u32()  等价于 let _a = MyType::to_u32(&mb)
                          // MyType::to_u32(&mb)中，
                          //              1、由&触发隐式转换, &mb从&MyBox转换到&MyType
                          //              2、调用MyType的to_u32方法
}
```

Deref还支持连续的隐式转换，示例如下：

```rust
fn print(s: &str) {
    println!("{}", s);
}
fn main() {
    let s = Box::new(String::from("hello world"));
    print(&s) // &s隐式转换过程：&Box -> &String -> &str
}
```

上面的代码中，`Box`、`String`都实现了Deref，当把`&s`传入到`print`函数时，发送连续隐式转换（`&s`先从`&Box`转换到`&String`，再转换到`&str`）。

## 5. Deref强制转换与可变性交互

类似于如何使用`Deref trait` 重载不可变引用的*运算符，Rust提供了DerefMut trait用于重载可变引用的*运算符。

Rust在发现类型和 trait 实现满足三种情况时会进行 Deref 强制转换：

- 当`T: Deref<Target=U>`时从`&T`到`&U`。
- 当`T: DerefMut<Target=U>`时从`&mut T`到`&mut U`。
- 当 `T: Deref<Target=U>`时从`&mut T`到`&U`。
