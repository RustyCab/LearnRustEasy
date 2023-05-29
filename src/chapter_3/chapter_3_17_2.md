# 3.17.2 模块 (Module)
## 1. 使用模块对代码分组
使用模块方便对代码进行分组，以提高可读性和重用性，例如如下代码：
```rust
fn main() {
    println!("Produce something!");
}
```
将其中的打印代码放入到一个模块中，变成如下：
```rust
mod factory {  // 创建一个模块
    pub fn produce() { // 将打印函数放在模块中
        println!("Produce something!");
    }
}

fn main() {
    factory::produce(); // 调用模块的函数
}
```

## 2. 定义模块控制作用域和私有性
使用模块可以控制作用域和私有性，示例如下：
```rust
mod factory {
    pub struct PubStruct {
        // 该结构体被定义为公有，外部可以使用
        pub i: u32, // 该字段被定义为公有，外部可以使用
    }

    struct PrivateStruct {
        // 该结构体定义为私有，外部无法使用
        i: u32,
    }

    // 该函数被定义为公有，外部可以使用
    pub fn function1() {
        let p1 = PubStruct { i: 3u32 };
        println!("p1 = {:?}", p1.i);
        let p2 = PrivateStruct { i: 3u32 };
        println!("p2 = {:?}", p2.i);

        function2();
        println!("This is a public function!");
    }

    // 该函数被定义为私有，外部无法使用
    fn function2() {
        let p1 = PubStruct { i: 3u32 };
        println!("p1 = {:?}", p1.i);
        let p2 = PrivateStruct { i: 3u32 };
        println!("p2 = {:?}", p2.i);
        println!("This is a private function!");
    }
}

fn main() {
    let _p1 = factory::PubStruct { i: 3u32 };
    // let _p2 = factory::PrivateStruct { i: 3u32 }; // 打开将编译出错，只能访问公有的类型
    factory::function1();
    // factory::function2();   // 打开编译将出错，只能访问公有的函数
}
```

模块中的项默认是私有的，要在模块外部使用模块内的一个项，则必须将该项前面加上pub关键字。

## 3. 绝对路径和相对路径
使用模块中的项时，需要通过对应的路径才能使用这个项。路径有两种形式：

- 绝对路径（absolute path）：以 crate root开头的全路径；对于外部 crate 的代码，是以 crate 名开头的绝对路径，对于对于当前 crate 的代码，则以字面值 crate 开头。
- 相对路径（relative path）：从当前模块开始，以 self、super 或当前模块的标识符开头。

```rust
mod parent {
    pub struct A(pub u32);

    pub mod factory {
        pub fn produce() {
            let _a1 = super::A(1u32); // 使用相对路径
            let _a2 = crate::parent::A(2u32); // 使用绝对路径
            println!("Produce something!");
        }
    }
}

fn main() {
    crate::parent::factory::produce(); // 使用绝对路径调用模块factory的函数
    self::parent::factory::produce();  // 使用相对路径调用模块factory的函数
    parent::factory::produce();  // 使用相对路径调用模块factory的函数
}
```

## 4. 使用use关键字引入作用域
在外部使用模块中的每个项都带上路径会显得比较重复，可以使用use关键字引入路径，示例如下：
```rust
mod parent {
    pub struct A(pub u32);

    pub mod factory {
        pub fn produce() {
            let _a1 = super::A(1u32); // 使用相对路径
            let _a2 = crate::parent::A(2u32); // 使用绝对路径
            println!("Produce something!");
        }
    }
}

mod ss {
    pub struct B(pub u32);
    pub fn print() {
        println!("Hello, world!");
    }
}

fn main() {
    use parent::factory::produce; // 引入produce的路径，可以直接使用produce
    produce(); // 直接使用

    use ss::*; // 引入路径,可以使用ss中的所有公有的项
    let _b = B(8u32);
    print();
}
```

还可以使用as关键字为引入的项提供新的名字，示例如下：
```rust
pub mod factory {
    pub fn produce() {
        println!("Produce something!");
    }
}

fn main() {
    use factory::produce as new_produce; // 通过as将produce命名为新的名字
    new_produce(); // 用新名字使用factory::produce函数
}
```

## 5. 将模块拆成多个文件
我们将上面第4点中的第二个例子拆成多个文件，步骤如下：

- 创建一个factory.rs，其内容为mod factory中的内容：
```rust
// src/factory.rs
pub fn produce() {
    println!("Produce something!");
}
```

- 在main.rs中导出mod，如下：
```rust
// src/main.rs
mod factory; // 导出factory module，module名字和文件名字同名

fn main() {
    use factory::produce as new_produce; // 通过as将produce命名为新的名字
    new_produce(); // 用新名字使用factory::produce函数
}
```

整个工程的目录结构如下：

![注释](.././assets/38.png)
