# 3.10.1 trait基础

trait定义了一组可以被共享的行为，只要实现了trait，就可以在代码中使用这组行为，它类似于其它语言中的接口（interface）：
- 可以通过trait以抽象的方式定义共享的行为；
- 可以使用trait bounds指定泛型是任何拥有特定行为的类型。

## 1. 定义trait

定义trait就是定义一组行为，如下：

```rust
pub trait GetInformation {
    fn get_name(&self) -> &String;
    fn get_age(&self) -> u32;
}
```

上述代码定义了一个叫做`GetInfomation`的trait，该trait提供了`get_name`和`get_age`方法。

## 2. 为类型实现trait

### 2.1 为类型实现trait

代码示例如下：

```rust
// 定义trait
pub trait GetInformation {
    fn get_name(&self) -> &String;
    fn get_age(&self) -> u32;
}

pub struct Student {
    pub name: String,
    pub age: u32,
}

// 为Student类型实现GetInformation trait
impl GetInformation for Student {
    fn get_name(&self) -> &String {
        &self.name
    }
    fn get_age(&self) -> u32 {
        self.age
    }
}

pub struct Teacher {
    pub name: String,
    pub age: u32,
}

// 为Teacher类型实现GetInformation trait
impl GetInformation for Teacher {
    fn get_name(&self) -> &String {
        &self.name
    }
    fn get_age(&self) -> u32 {
        self.age
    }
}

fn main() {
    let s = Student {
        name: "alice".to_string(),
        age: 18,
    };
    // 可以在类型Student上使用GetInfomation trait中定义的方法
    println!("s.name = {:?}, s.age = {:?}", s.get_name(), s.get_age());

    let t = Teacher {
        name: "bob".to_string(),
        age: 25,
    };
    // 可以类型Teacher使用GetInfomation trait中定义的方法
    println!("t.name = {:?}, t.age = {:?}", t.get_name(), t.get_age());
}
```

### 2.2 可以在trait定义时提供默认实现

可以在定义trait的时候提供默认的行为，trait的类型可以使用默认的行为，示例如下：

```rust
// 定义trait
pub trait GetInformation {
    fn get_name(&self) -> &String;
    fn get_age(&self) -> u32 {   // 在定义trait时就提供默认实现
        25u32
    }
}

pub struct Student {
    pub name: String,
    pub age: u32,
}

// 为Student类型实现GetInformation trait
impl GetInformation for Student {
    fn get_name(&self) -> &String {
        &self.name
    }
    // 实现get_age方法，Student不会使用trait定义时的默认实现
    fn get_age(&self) -> u32 {
        self.age
    }
}

pub struct Teacher {
    pub name: String,
    pub age: u32,
}

// 为Teacher类型实现GetInformation trait
impl GetInformation for Teacher {
    fn get_name(&self) -> &String {
        &self.name
    }
    // 不实现get_age方法，将使用trait的默认实现
}

fn main() {
    let s = Student {
        name: "alice".to_string(),
        age: 18,
    };
    // 可以在类型Student上使用GetInfomation trait中定义的方法
    // 输出t.name = "bob", t.age = 25
    println!("s.name = {:?}, s.age = {:?}", s.get_name(), s.get_age());

    let t = Teacher {
        name: "bob".to_string(),
        age: 25,
    };
    // 可以类型Teacher使用GetInfomation trait中定义的方法（t.get_age将使用定义trait时的默认方法）
    // 输出t.name = "bob", t.age = 25
    println!("t.name = {:?}, t.age = {:?}", t.get_name(), t.get_age());
}
```

如果定义trait时提供了某个方法的默认实现，则：
- 如果为类型实现该trait时，为该类型实现了此方法，则使用自己实现的方法（如上面示例中的Student类型）；
- 如果为类型实现该trait时，没有为该类型实现此方法，则该类型使用trait提供的默认实现（如上面的Teacher类型）。

## 3. trait作为参数

### 3.1 trait作为参数

trait可以用来参数，示例如下：

```rust
// 定义trait
pub trait GetInformation {
    fn get_name(&self) -> &String;
    fn get_age(&self) -> u32 {
        25u32
    }
}

pub struct Student {
    pub name: String,
    pub age: u32,
}

// 为Student类型实现GetInformation trait
impl GetInformation for Student {
    fn get_name(&self) -> &String {
        &self.name
    }
    fn get_age(&self) -> u32 {
        self.age
    }
}

pub struct Teacher {
    pub name: String,
    pub age: u32,
}

// 为Teacher类型实现GetInformation trait
impl GetInformation for Teacher {
    fn get_name(&self) -> &String {
        &self.name
    }
}

// 参数类型必须是实现了GetInfomation trait的类型
pub fn print_information(item: impl GetInformation) {
    println!("name = {}", item.get_name());
    println!("age = {}", item.get_age());
}

fn main() {
    let s = Student {
        name: "alice".to_string(),
        age: 18,
    };
    print_information(s);

    let t = Teacher {
        name: "bob".to_string(),
        age: 25,
    };
    print_information(t);
}
```

在上面的例子中，函数`pub fn print_information(item: impl GetInformation)`的要求参数item必须实现`GetInformation trait`，否则无法调用该参数。

### 3.2 使用`trait bound`语法


上面中的print_information函数还可以写成如下：

```rust
// 使用trait bound的写法一
pub fn print_information<T: GetInformation>(item: T) {
    println!("name = {}", item.get_name());
    println!("age = {}", item.get_age());
}
```

这种写法叫做Trait bound语法，它是Rust中用于指定泛型类型参数所需的trait的一种方式，它还可以使用where关键字写成如下：

```rust
// 使用trait bound的写法二
pub fn print_information<T>(item: T)
where
    T: GetInformation,
{
    println!("name = {}", item.get_name());
    println!("age = {}", item.get_age());
}
```

### 3.3 通过“`+`”指定多个trait bound

可以要求类型实现多个trait，示例如下：
```rust
pub trait GetName {
    fn get_name(&self) -> &String;
}

pub trait GetAge {
    fn get_age(&self) -> u32;
}

//使用trait bound写法一，类型T必须实现GetName和GetAge trait
pub fn print_information1<T: GetName + GetAge>(item: T) {
    println!("name = {}", item.get_name());
    println!("age = {}", item.get_age());
}

//使用trait bound写法二，类型T必须实现GetName和GetAge trait
pub fn print_information2<T>(item: T)
where
    T: GetName + GetAge,
{
    println!("name = {}", item.get_name());
    println!("age = {}", item.get_age());
}

#[derive(Clone)]
struct Student {
    name: String,
    age: u32,
}

impl GetName for Student {
    fn get_name(&self) -> &String {
        &self.name
    }
}

impl GetAge for Student {
    fn get_age(&self) -> u32 {
        self.age
    }
}

fn main() {
    let s = Student {
        name: "alice".to_string(),
        age: 18u32,
    };
    print_information1(s.clone());
    print_information1(s);
}
```

在上面的代码中，`print_information1`和`print_information2`函数要求其参数类型T必须实现`GetName`和`GetAge`两个trait，通过`+`来进行多个约束的连接。

## 4. 返回trait的类型

trait类型可以作为函数的返回类型，示例如下：

```rust
pub trait GetName {
    fn get_name(&self) -> &String;
}

struct Student {
    name: String,
}

impl GetName for Student {
    fn get_name(&self) -> &String {
        &self.name
    }
}

// trait类型作为返回的参数
pub fn produce_item_with_name() -> impl GetName {
    // 返回一个实现了GetName trait的类型
    Student {
        name: "alice".to_string(),
    }
}

fn main() {
    let s = produce_item_with_name();
    println!("name: {:?}", s.get_name());
}
```

上面代码中，`produce_item_with_name`函数返回了一个实现了`GetName trait`的类型。不过需要注意的是，这种方式返回的是单一类型，例如如下的代码就是错误的，无法编译通过：

```rust
pub trait GetName {
    fn get_name(&self) -> &String;
}

struct Student {
    name: String,
}

impl GetName for Student {
    fn get_name(&self) -> &String {
        &self.name
    }
}

struct Teacher {
    name: String,
}

impl GetName for Teacher {
    fn get_name(&self) -> &String {
        &self.name
    }
}

// 下面的代码将是错误的，无法编译通过，因为在编译时，返回的类型就确定为某一个实现了GetName trait的具体类型了
pub fn produce_item_with_name(is_teacher: bool) -> impl GetName {
    let result = if is_teacher {
        Teacher {
            name: "alice".to_string(),
        }
    } else {
        Student {
            name: "alice".to_string(),
        }
    };

    result
}

fn main() {
    let s = produce_item_with_name(false);
    println!("name: {:?}", s.get_name());
}
```

错误原因分析（非常重要）：
上面的代码中的`produce_item_with_name`函数的定义实际上等价于如下：

```Rust
pub fn produce_item_with_name<T: GetName>(is_teacher: bool) -> T {
    ...

    result
}
```

返回的值相当于是一个泛型，这个泛型要求要实现`GetName`这个trait。回顾泛型的知识，Rust实际上是在编译的时候把泛型换成了具体的类型，所以上面的定义中，T在编译时会变成确定的某个类型(按照上下文，即Student类型或Teacher类型)。所以在编译时，上面的代码可能被翻译成如下两种情况：

```rust
// 编译时代码将被翻译成如下：
pub fn produce_item_with_name(is_teacher: bool) -> Teacher {
    let result = if is_teacher {
        Teacher { name: "alice".to_string() }
    } else {
        Student { name: "alice".to_string() }
    };

    result
}

// 也可能翻译成如下：
pub fn produce_item_with_name(is_teacher: bool) -> Student {
    let result = if is_teacher {
        Teacher { name: "alice".to_string() }
    } else {
        Student { name: "alice".to_string() }
    };

    result
}
```

无论是哪种情况，都是错误的。

那如果需要返回多种实现了trait的类型，则需要使用后续讲解的内容trait对象（3.10.2节）来满足需求。

## 5. 使用`trait bound`有条件的实现方法

通过使用带有 `trait bound `的泛型参数的`impl` 块，可以有条件地只为那些实现了特定 trait 的类型实现方法，示例如下：

```rust
pub trait GetName {
    fn get_name(&self) -> &String;
}

pub trait GetAge {
    fn get_age(&self) -> u32;
}

struct PeopleMatchInformation<T, U> {
    master: T,
    employee: U,
}

// 11-15行也可以写成： impl<T: GetName + GetAge, U: GetName + GetAge> PeopleMatchInformation<T, U>
impl<T, U> PeopleMatchInformation<T, U>
where
    T: GetName + GetAge,   // T和U都必须实现GetName和GetAge trait
    U: GetName + GetAge,
{
    fn print_all_information(&self) {
        println!("teacher name = {}", self.master.get_name());
        println!("teacher age = {}", self.master.get_age());
        println!("student name = {}", self.employee.get_name());
        println!("student age = {}", self.employee.get_age());
    }
}

//使用
pub struct Teacher {
    pub name: String,
    pub age: u32,
}

impl GetName for Teacher {
    fn get_name(&self) -> &String {
        &(self.name)
    }
}

impl GetAge for Teacher {
    fn get_age(&self) -> u32 {
        self.age
    }
}

pub struct Student {
    pub name: String,
    pub age: u32,
}

impl GetName for Student {
    fn get_name(&self) -> &String {
        &(self.name)
    }
}

impl GetAge for Student {
    fn get_age(&self) -> u32 {
        self.age
    }
}

fn main() {
    let t = Teacher {
        name: String::from("andy"),
        age: 32,
    };
    let s = Student {
        name: String::from("harden"),
        age: 47,
    };
    let m = PeopleMatchInformation {
        master: t,
        employee: s,
    };
    m.print_all_information();
}
```

上面的代码中，就是为`PeopleMatchInformation`有条件的实现`print_all_information`方法。

## 6. 对任何实现了特定trait的类型有条件的实现trait

在Rust中，另外一种比较常见的trait的用法就是对实现了特定trait的类型有条件的实现trait，示例如下：

```rust
// trait定义
pub trait GetName {
    fn get_name(&self) -> &String;
}

pub trait PrintName {
    fn print_name(&self);
}

// 为实现了GetName trait的类型实现PrintName trait
impl<T: GetName> PrintName for T {
    fn print_name(&self) {
        println!("name = {}", self.get_name());
    }
}

// 将为Student实现对应的trait
pub struct Student {
    pub name: String,
}

impl GetName for Student {
    fn get_name(&self) -> &String {
        &(self.name)
    }
}

fn main() {
    let s = Student {
        name: String::from("Andy"),
    };
    s.print_name(); //student实现了GetName trait，因此可是直接使用PrintName trait中的函数print_name
}
```

上面的例子中，就是为实现了`GetName trait`的类型实现`PrintName trait`。
