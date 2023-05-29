# 3.10.2 trait对象

在上一节中第4点返回trait对象时，提到了`produce_item_with_name`函数返回的是单一的类型，即在编译时就确定了具体的类型，因此`produce_item_with_name`无法正确编译。为解决这种问题，Rust中引入了trait对象。

## 1. 使用trait对象

在Rust中，trait自身不能当作数据类型来使用，但trait 对象可以当作数据类型使用。因此，可以将实现了`Trait A`的类型`B`、`C`、`D`当作`trait A`的trait对象来使用。使用trait对象时，基本都是以引用的方式使用，所以使用时通常是引用符号加`dyn`关键字（即`&dyn`）。

示例如下：

```rust
trait GetName {
    fn get_name(&self);
}

struct SchoolMember<'a>(&'a dyn GetName); // 学校成员是GetName trait对象

impl<'a> SchoolMember<'a> {
    fn print_name(&self) {
        self.0.get_name();
    }
}

// Student是实现了GetName trait的类型
struct Student {
    name: String,
}

impl GetName for Student {
    fn get_name(&self) {
        println!("name: {:?}", self.name);
    }
}

// Teacher是实现了GetName trait的类型
struct Teacher {
    name: String,
}

impl GetName for Teacher {
    fn get_name(&self) {
        println!("name: {:?}", self.name);
    }
}

fn main() {
    let alice = Student {
        name: "alice".to_string(),
    };
    let bob = Teacher {
        name: "bob".to_string(),
    };
    let sm1 = SchoolMember(&alice); // 把alice作为GetName trait对象传入
    sm1.print_name();
    let sm2 = SchoolMember(&bob); // 把bob作为GetName trait对象传入
    sm2.print_name();


    let chalie: &dyn GetName = &Student {
        name: "chalie".to_string(),
    };
    chalie.get_name();
}
```

上面代码中，`Student`和`Teacher`都实现了`GetName trait`，因此这两种类型可以当做`GetName`的trait对象使用。

## 2. trait对象动态分发的原理

对于trait对象，需要说明如下几点：

- trait 对象大小不固定：这是因为，对于`trait A`，类型`B`可以实现`trait A`，类型`C`也可以实现`trait A`，因此A trait对象的大小是无法确定的（因为可能是B类型也可能是C类型）。
- 使用trait对象时，总是使用它们的引用的方式：
  - 虽然trait对象没有固定大小，但它的引用类型的大小固定，它由两个指针组成，因此占两个指针大小。
  - 一个指针指向具体类型的实例。
  - 另一个指针指向一个虚表vtable，vtable中保存了实例对于可以调用的实现于trait的方法。当调用方法时，直接从vtable中找到方法并调用。
  - trait对象的引用方式有多种。例如对于`trait A`，其trait对象类型的引用可以是`&dyn A`、`Box<dyn A>`、`Rc<dyn A>`等。

下面通过一段代码来分析一下使用trait对象时内存的布局。代码如下：

```rust
  trait Vehicle {
    fn run(&self);
}

// Car是实现了Vehicle trait的类型
// 只有一个字段表示车牌号
struct Car(u32);

impl Vehicle for Car {
    fn run(&self) {
        println!("Car {:?} run ... ", self.0);
    }
}

// truck是实现了Vehicle trait的类型
// 只有一个字段表示车牌号
struct Truck(u32);

impl Vehicle for Truck {
    fn run(&self) {
        println!("Truck {:?} run ... ", self.0);
    }
}

fn main() {
    let car = Car(1001);
    let truck = Truck(1002);

    let vehicle1: &dyn Vehicle = &car;
    let vehicle2: &dyn Vehicle = &truck;

    vehicle1.run();
    vehicle2.run();
}
```

在上面的代码中，vehicle1和vehicle1都是Vehicle trait对象的引用；对于vehicle1来说，trait对象的具体类型是`Car`；对于vehicle2来说，trait对象的具体类型是`Truck`。上面代码对应的内存布局如下：

  ![注释](.././assets/14.png)

变量car和变量truck分别是Car类型和Truck类型，存储在栈上；vehicle1和vehicle2是Vehicle trait对象的引用，具有两个指针，其中指针ptr指向具体类型的实例，vptr指向虚函数表；虚函数表存储在只读数据区。

更进一步的理解，虚函数表存储在程序的可执行文件中的只读数据段（.rodata）中，这个只读数据段在程序运行时被加载到内存中，因此虚函数表也是只读的。实现trait对象的时候，编译器会在对象的内存布局中添加一个指向虚函数表的指针，这个指针被称为虚函数表指针。在程序运行到`vehicle1.run()`和`vehicle2.run()`时，程序通过虚函数表找到对应的函数指针，然后来执行。

## 3. trait对象要求对象安全

只有对象安全（object safe）的 trait 才可以组成 trait 对象。trait的方法满足以下两条要求才是对象安全的：
- 返回值类型不为 `Self`；
- 方法没有任何泛型类型参数。

> 分析：
>
>***不允许返回`Self`，是因为trait对象在产生时，原来的具体的类型会被抹去，Self具体是什么类型不知道，所以编译会报错；
>    不允许携带泛型参数，是因为Rust用带泛型的类型在编译时会做单态化，而trait对象是运行时才确定，即一个运行时才能确定的东西里又包含一个需要在编译时确定的东西，相互冲突，必然是不行的***。

如下代码编译会报错，因为`Clone`返回的是`Self`：

```rust
pub struct Screen {
   pub components: Vec<Box<dyn Clone>>,
}
```
