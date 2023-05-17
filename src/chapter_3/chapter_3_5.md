## 3.5 控制流

Rust中的控制流结构主要包括：

- if条件判断；
- loop循环；
- while循环；
- for .. in 循环。

### 3.5.1. if条件判断

- if执行条件判断，示例如下：
```Rust
fn main() {
    let a = 2u32;

    if a > 5u32 {
        println!("a > 5");
    } else {
        println!("a <= 5");
    }
}
```

- if - else if处理多重条件，示例如下：
```Rust
fn main() {
    let a = 3u32;

    if a > 5u32 {
        println!("a > 5");
    } else if a > 4u32 {
        println!("a > 4");
    } else if a > 3u32 {
        println!("a > 3");
    } else if a > 2u32 {
        println!("a > 2");
    } else if a > 1u32 {
        println!("a > 1");
    } else {
        println!("a = 1");
    }
}
```

- 在let语句中使用if

if是一个表达式，所以可以在let右侧使用，如下：
```Rust
fn main() {
    let a = 3u32;

    let a_bigger_than_two: bool = if a > 2u32 {
        true
    } else {
        false
    };

    if a_bigger_than_two {
        println!("a > 2");
    } else {
        println!("a <= 2");
    }
}
```

### 3.5.2. loop循环

- loop重复执行代码
```Rust
fn main() {
    // 一直循环打印 again
    loop {
        println!("again!");
    }
}
```

- 使用break终止循环
```Rust
fn main() {
    let mut counter = 0;

    loop {
        println!("counter = {:?}", counter);
        counter += 1;

        if counter == 10 {
            break;  // 将终止循环
        }
    }
}
```
上面的代码将打印10次，遇到break后终止循环。另外，break也可以返回值，如下：
```Rust
fn main() {
    let mut counter = 0;

    let result = loop {
        counter += 1;

        if counter == 10 {
            break counter * 2;
        }
    };

    println!("The result is {result}");
}
```

- 使用continue可以直接跳到下一轮循环
```Rust
fn main() {
    let mut x = 0;
    // 此循环将只打印10以内的奇数
    loop {
        x += 1;
        if x == 10 {
            break;
        }
        if x % 2 == 0 {
            continue;      //将直接跳到下一轮循环
        }
        println!("{}", x);
    }
}
```

### 3.5.3. while条件循环
- while条件循环执行代码，当条件不满足后结束循环，如下：
```Rust
fn main() {
    let mut cnt = 0u32;
    while cnt < 10 {
        println!("cnt = {:?}", cnt);
        cnt += 1;
    }
}
```

- 在while循环中也可以使用break和continue，如下：
```Rust
fn main() {
    let mut x = 0;

    while x < 10 {
        x += 1;
        println!("{}", x);
        if x % 2 == 0 {
            continue; // 跳到下一轮循环
        }
        if x > 8 {
            break; // 提前结束
        }
    }
}
```

### 3.5.4. for .. in 循环

for循环用来对一个集合的每个元素执行一些代码，使用方式如下：
```Rust
fn main() {
    let a = [10, 20, 30, 40, 50];

    for item in a {
        println!("the value is: {:?}", item);
    }
}
```
