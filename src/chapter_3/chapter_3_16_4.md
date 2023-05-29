# 3.16.4 Drop trait

## 1. Drop trait

`Drop trait`类似于其它语言中的析构函数，当值离开作用域时执行此函数的代码。可以为任何类型提供Drop trait的实现。

为一个类型实现`Drop trait`的示例如下：

```rust
struct Dog(String);

//下面为Dog实现Drop trait
impl Drop for Dog {
    fn drop(&mut self) {
        println!("Dog leave");
    }
}

fn main() {
    let _a = Dog(String::from("wangcai"));
    let _b = Dog(String::from("dahuang"));
}
```

运行该代码，会有如下结果：

![注释](.././assets/24.png)

在上面示例代码中并没有打印语句，但是在drop方法中实现了打印，可以看到，当_a和_b离开作用域时，自动调用了drop方法。

## 2. 通过std::mem::drop提早丢弃值

当要显示的清理值时，不能直接调用Drop trait里面的drop方法，而要使用`std::mem::drop`方法，示例如下：

```rust
struct Dog(String);

//下面为Dog实现Drop trait
impl Drop for Dog {
    fn drop(&mut self) {
        println!("Dog leave");
    }
}

fn main() {
    let _a = Dog(String::from("wangcai"));
    let _b = Dog(String::from("dahuang"));
    //a.drop();//错误，不能直接调用drop
    drop(_a); //正确，通过std::mem::drop显示清理
    println!("At the end of main");
}
```

代码运行结果如下：

![注释](.././assets/25.png)

第一个“Dog leave”打印是第14行调用释放_a产生，最后一个“Dog leave”打印则是_b离开作用域时调用drop方法产生。
