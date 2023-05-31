# 3.3 函数

## 3.3.1. 函数定义

fn关键字、函数名、函数参数名及其类型（如果有的话）、返回值类型（如果有的话）组成函数签名， 加上由一对花括号包含的函数体组成函数。例子如下：

```rust
// 一个没有参数，也没有返回值的函数
fn print_line() {
    println!("++++++++++++++++");
}

// 一个有参数，没有返回值的函数
fn print(x: u32) {
    println!("result is {:?}", x);
}

// 一个有参数，也有返回值的函数
fn sum(a: u32, b: u32) -> u32 {
    a + b
}

// main函数是rust程序的入口函数
fn main() {
    print_line();
    let a = 1u32;
    let b = 2u32;
    let r = sum(a, b);
    print(r);
}
```

Rust中，函数也可以定义在函数内部，如下：

```rust
fn calculate(a: u32, b: u32) {
    println!("a is {:?}", a);
    println!("b is {:?}", b);

    // 在函数内部定义函数
    fn sum(a: u32, b: u32) -> u32 {
        a + b
    }

    let r = sum(a, b);
    println!("a + b is {:?}", r);
}

fn main() {
    let a = 1u32;
    let b = 2u32;
    calculate(a, b);
}
```

## 3.3.2. 语句和表达式

Rust中，语句是执行一个操作但不返回值的指令，表达式则计算并产生一个值。

```rust
fn main() {
    let a = 1u32; // "1u32"就是一个表达式， “let a = 1u32;”则是一个语句
    let b = a + 1;// “a + 1”就是一个表达式，“let b = a + 1;”则是一个语句
    // 下面的代码中，
    // “ b + d ”是一个表达式，
    let c = {
        let d = 2u32;
        b + d  //注意：这里是没有分号的
    }; // c = 4
}
```

## 3.3.3. 函数返回值

- 使用return指定返回值，如下：

    ```rust
    fn sum(a: u32, b: u32) -> u32 {
        let r = a + b;
        return r     //可以加分号，也可以不加分号, 所以这行等价于“return r;”
    }

    fn main() {
        let a = 1u32;
        let b = 2u32;
        let c = sum(a, b);
        println!("c = {:?}", c);
    }
    ```

    特别的return关键字不指定值时，表示返回的是()，如下：
    ```rust
    fn my_function() -> () {
        println!("some thing");
        return; //等价于 “return ()”
    }
    ```

- 不使用return关键字，将返回最后一条执行的表达式的计算结果，如下：

    ```rust
    fn sum(a: u32, b: u32) -> u32 {
        println!("a is {:?}", a);
        println!("b is {:?}", b);

        a + b //注意，是没有加分号的
    }

    fn main() {
        let a = 1u32;
        let b = 2u32;
        let c = sum(a, b);
        println!("c = {:?}", c);
    }
    ```
