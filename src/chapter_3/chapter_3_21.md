# 3.21 unsafe编程

## 3.21.1 unsafe简介

到此节之前本书介绍的基本都是安全的Rust，即Rust编译时会强制执行内存安全保证的检查，只要编译通过，基本就能保证代码的安全性。既然有安全的Rust，那么必然也有不安全的Rust。

### 3.21.1.1 不安全的Rust存在的原因

- 代码静态分析相对保守，这就意味着某些代码可能是合法的，但是编译器检查也会拒绝通过。在此情况下，可以使用不安全的代码。
- 底层计算机硬件固有的不安全性。如果Rust不允许进行不安全的操作，有些任务根本就完成不了。

### 3.21.1.2. 不安全的Rust使用的场景

Rust通过unsafe关键字切换到不安全的Rust，不安全的Rust使用的场景如下：
- 解引用裸指针；
- 调用不安全的函数或者方法；
- 访问或修改可变静态变量；
- 实现不安全的trait。

注意：unsafe并不会关闭借用检查器或禁用任何其它的Rust安全检查规则，它只提供上述几个不被编译器检查内存安全的功能。unsafe也不意味着块中的代码一定是有问题的，它只是表示由程序员来确保安全。

## 3.21.2 使用unsafe编程

### 3.21.2.1. 解引用裸指针

裸指针又称为原生指针，在功能上和引用类似，但是引用的安全由Rust编译器保证，裸指针则不是。裸指针需要显式标明可变性，不可变的裸指针分别写作 ``*const T`` ，可变的裸指针写作 ``*mut T``。


裸指针与引用和只能指针的区别如下：
- 可以忽略借用规则，在代码中同时拥有不可变和可变的裸指针，或多个指向相同位置的可变裸指针；
- 不保证裸指针指向有效的内存；
- 允许裸指针为空；
- 裸指针不能实现任何自动清理功能。

可以在安全代码中创建可变或不可变的裸指针，但是只能在不安全块中解引用裸指针，示例如下：

```rust
fn main() {
    let mut num = 5;
    let r1 = &num as *const i32;
    let r2 = &mut num as *mut i32;
    let address = 0x012345usize;
    let _r = address as *const i32;
    unsafe {
        println!("r1 is: {}", *r1);
        println!("r2 is: {}", *r2);
    }
}
```

说明：创建一个裸指针不会造成任何危险，只有当访问其指向的值时（并且这个值已经无效）才可能造成危险，所以可以在安全的代码块中创建裸指针，但是使用则必须在unsafe的块中。

### 3.21.2.2. 调用不安全的函数或者方法

示例1，代码如下：

```rust
unsafe fn dangerous() {
    println!("Do some dangerous thing");
}
fn main() {
    unsafe {
        dangerous();
    }
    println!("Hello, world!");
}
```

示例2，代码如下：
```rust
fn foo() {
    let mut num = 5;
    let r1 = &num as *const i32;
    let r2 = &mut num as *mut i32;
    unsafe {
        println!("r1 is: {}", *r1);
        println!("r2 is: {}", *r2);
    }
}
fn main() {
    foo();
}
```

### 3.21.2.3. 访问或者修改可变静态变量

全局变量在Rust中被称为静态变量。静态变量的名称采用SCREAMING_SNAKE_CASE写法，它只能存储拥有 'static生命周期的引用。下面的示例使用了不可变的静态变量：

```rust
static HELLO_WORLD: &str = "Hello, world!"; // 不可变的静态变量
fn main() {
    println!("name is: {}", HELLO_WORLD);
}
```

这里对比一下常量和静态变量：

- 静态变量中的值有一个固定的内存地址（即使用这个值总会访问相同的地址），常量则允许在任何被用到的时候复制其数据。
- 静态变量可以是可变的，虽然这可能是不安全的。

下面的示例使用可变的静态变量：

```rust
static mut COUNTER: u32 = 0; // 可变的静态变量，同样需要mut关键字
fn add_to_count(inc: u32) {
    unsafe {
        COUNTER += inc;
    }
}
fn main() {
    add_to_count(3);
    unsafe {
        println!("COUNTER: {}", COUNTER);
    }
}
```

### 3.21.2.4. 实现不安全的trait

当至少有一个方法中包含编译器不能验证的invariant时，该trait就是不安全的。在trait之前增加unsafe关键字将trait声明为unsafe，同时trait的实现也必须标记为unsafe。下面为使用unsafe的trait的示例：

```rust
struct Bar;

unsafe trait Foo {
    fn foo(&self);
}

unsafe impl Foo for Bar {
    fn foo(&self) {
        println!("foo");
    }
}

fn main() {
    let a = Bar;
    a.foo();
}
```
