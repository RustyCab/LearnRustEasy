# 3.13 闭包

## 3.13.1 闭包介绍

闭包是可以保存进变量或者作为参数传递给其它函数的匿名函数。闭包和函数不同的是，闭包允许捕获调用者作用域中的值。下面为使用闭包的简单示例：

```rust
fn main() {
    let use_closure = || {
        println!("This is a closure");
    };
    use_closure(); // 此行打印“This is a closure”
}
```

闭包有如下语法格式：

```rust
fn add_one_v1(x: u32) -> u32 { x + 1 }               //函数
let add_one_v2 = |x: u32| -> u32 { x + 1 };          //闭包
let add_one_v3 = |x| { x + 1 };                      //自动推导参数类型和返回值类型
let add_one_v4 = |x| x+1;                            //自动推导参数类型和返回值类型
```

闭包定义会为每个参数和返回类型推导一个具体类型，但是不能推导两次。下面是错误示例：
```rust
fn main() {
    let example_closure = |x| x;
    let s = example_closure(String::from("hello"));
    let n = example_closure(5); //报错，尝试推导两次，变成了不同的类型
}
```

## 3.13.2 闭包捕获环境

下面的示例展示了闭包捕获环境中的变量：

```rust
fn main() {
    let x = 4;
    let equal_to_x = |z| z == x; //捕获环境中的值
    let y = 4;
    assert!(equal_to_x(y));
}
```

闭包可以通过三种方式捕获其环境，对应函数的三种获取参数的方式，分别是获取所有权、可变借用和不可变借用。
这三种捕获值的方式被编码为如下三个trait：

- `FnOnce`：消费从周围作用域捕获变量（即获取捕获变量的所有权），闭包周围的作用域被称为其环境。为了消费捕获到的变量，闭包必须获取其所有权并将其移动进闭包。其名称的`Once`部分代表了闭包不能多次获取相同变量的所有权。
- `FnMut`：获取可变的借用值，所以可以改变其环境。
- `Fn`：从其环境获取不可变的借用值。

当创建一个闭包时，Rust会根据其如何使用环境中的变量来推断如何引用环境。由于所有闭包都可以被调用至少一次，因此所有闭包都实现了`FnOnce`。没有移动被捕获变量的所有权到闭包的闭包也实现了`FnMut`，而不需要对捕获的变量进行可变访问的闭包则实现了`Fn`。

下面示例分别给出了实现三种Trait的闭包：

```rust
fn call_once(c: impl FnOnce()) {
    c();
}
fn call_mut(c: &mut impl FnMut()) {
    c();
}
fn call_fn(c: impl Fn()) {
    c();
}
fn main() {
    // 1、闭包use_closure1只实现了FnOnce Trait，只能被调用一次
    let s = "Hello".to_string();
    let use_closure1 = move || {
        let s1 = s;
        println!("s1 = {:?}", s1);
    };
    use_closure1(); // 此行打印“s1 = "Hello"”
                    // println!("s = {:?}", s); // 编译错误：因为s所有权已经被移动闭包中use_closure1中
                    // use_closure1();  // 编译错误：多次调用use_closure1出错
    let s = "Hello".to_string();
    let use_closure11 = move || {
        let s1 = s;
        println!("s1 = {:?}", s1);
    };
    call_once(use_closure11);

    // 2、闭包use_closure2只实现了FnOnce Trait和FnMut Trait
    let mut s = "Hello".to_string();
    let mut use_closure2 = || {
        s.push_str(", world!");
        println!("s = {:?}", s);
    };
    use_closure2(); // 此行打印“s = "Hello, world!"”
    use_closure2(); // 可以多次调用，此行打印“s = "Hello, world!, world!"”
    call_mut(&mut use_closure2);
    call_once(use_closure2);

    // 3、闭包use_closure3实现了FnOnce Trait、FnMut Trait和Fn Trait
    let s = "Hello".to_string();
    let mut use_closure3 = || {
        println!("s = {:?}", s);
    };
    use_closure3(); // 此行打印“s = "Hello"”
    use_closure3(); // 可以多次调用，此行打印“s = "Hello!"”
    call_fn(use_closure3);
    call_mut(&mut use_closure3);
    call_once(use_closure3);
}
```

## 3.13.3 作为参数和返回值

### 1. 函数指针

函数指针的使用可以让函数作为另一个函数的参数。函数的类型是`fn`，`fn`被称为函数指针。指定参数为函数指针的语法类似于闭包。

```rust
fn add_one(x: i32) -> i32 {
    x + 1
}
fn do_twice(f: fn(i32) -> i32, arg: i32) -> i32 {  //第一个参数为函数指针
    f(arg) + f(arg)
}
fn main() {
    let answer = do_twice(add_one, 5);
    println!("The answer is: {}", answer);
}
```

函数指针实现了闭包的三个trait（`Fn`、`FnMut` 和 `FnOnce`），函数指针作为参数的地方也可以传入闭包。

### 2. 闭包作为参数和返回值

基于上面的1的知识可知，闭包可以作为参数，同样也可以作为返回值，闭包作为参数的示例如下：

```rust
fn wrapper_func<T>(t: T, v: i32) -> i32
where
    T: Fn(i32) -> i32,
{
    t(v)
}

fn func(v: i32) -> i32 {
    v + 1
}

fn main() {
    let a = wrapper_func(|x| x + 1, 1); // 闭包作为参数
    println!("a = {}", a);

    let b = wrapper_func(func, 1);  // 函数作为参数
    println!("b = {}", b);
}
```

闭包作为返回值的示例如下：

```rust
fn returns_closure() -> Box<dyn Fn(i32) -> i32> {  // 返回的是trait对象
    Box::new(|x| x + 1)
}

fn main() {
    let c = returns_closure();
    println!("r = {}", c(1)); //等价于println!("r = {}", (*c)(1));
}
```
需要注意的是，函数定义时返回的是Box包含的trait对象，因为编译器在编译时需要知道返回值的大小。所以对于下面两种`returns_closure`函数定义，编译器将报错：

```rust
// 错误方式一
fn returns_closure() -> dyn Fn(i32) -> i32 {
    |x| x + 1
}
// 错误方式二
fn returns_closure() -> Fn(i32) -> i32 {
    |x| x + 1
}
```

### 3. 闭包和泛型

闭包还可以和泛型结合在一起使用，示例如下：

```rust
// T 要求实现Fn
fn returns_closure1<T>(f: T) -> T
where
    T: Fn(i32) -> i32,
{
    f
}
// T 要求实现FnMut
fn returns_closure2<T>(f: T) -> T
where
    T: FnMut(),
{
    f
}
// T 要求实现FnOnce
fn returns_closure3<T>(f: T) -> T
where
    T: FnOnce(),
{
    f
}

fn main() {
    let closure1 = |x| x + 1;
    let c = returns_closure1(closure1);
    println!("r = {}", c(1));

    // T 实现了FnMut、FnOnce
    let mut s = "Hello".to_string();
    let closure2 = || {
        s.push_str(", world!");
    };
    let mut c = returns_closure2(closure2);
    c();
    println!("s: {:?}", s);

    let s = "Hello".to_string();
    let closure3 = move || {
        let s1 = s;
        println!("s = {:?}", s1);
    };
    let c = returns_closure3(closure3);
    c();
}
```

## 3.13.4 闭包背后的原理

Rust中的闭包是通过一个特殊的结构体实现的。具体来说，每个闭包都是一个结构体对象，其中包含了闭包的代码和环境中捕获的变量。这个结构体对象实现了一个或多个trait，以便可以像函数一样使用它。
当定义一个闭包时，Rust编译器会根据闭包的代码和捕获的变量生成一个结构体类型，这个结构体类型实现了对应的trait。例如，以下代码定义了一个闭包`add_x`并调用它：

```rust
fn main() {
    let x = 10;
    let add_x = |y| x + y;     // 闭包
    println!("{}", add_x(5));  // 调用闭包
}
```

在编译时，Rust编译器会将这个闭包`add_x`转换为如下的结构体类型：

```rust
struct Closure<'a> {
    x: i32,
}

impl<'a> FnOnce<(i32,)> for Closure<'a> {
    type Output = i32;
    fn call_once(self, args: (i32,)) -> i32 {
        self.x + args.0
    }
}

impl<'a> FnMut<(i32,)> for Closure<'a> {
    fn call_mut(&mut self, args: (i32,)) -> i32 {
        self.x + args.0
    }
}

impl<'a> Fn<(i32,)> for Closure<'a> {
    extern "rust-call" fn call(&self, args: (i32,)) -> i32 {
        self.x + args.0
    }
}
```

当闭包被调用时，它实际上是通过调用结构体的方法来执行的。所以调用闭包的代码就变成了如下：
```rust
fn main() {
    let x = 10;
    let mut add_x = Closure { x, y: 0 };
    println!("{}", add_x(5));
}
```
