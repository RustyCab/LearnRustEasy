# 3.20 Rust并发编程

## 3.20.1 使用线程

### 3.20.1.1 相关概念

- 进程是资源分配的最小单位，线程是CPU调度的最小单位。
- 在使用多线程时，经常会遇到如下一些问题：
    - 竞争状态：多个线程以不一致的顺序访问数据或资源；
    - 死锁：两个线程相互等待对方停止使用其所拥有的资源，造成两者都永久等待；
    - 只会发生在特定情况下且难以稳定重现和修复的bug。
- 编程语言提供的线程叫做绿色线程，如go语言，在底层实现了M:N的模型，即M个绿色线程对应N个OS线程。但是，Rust标准库只提供1：1的线程模型的实现，即一个Rust线程对应一个Os线程。

### 3.20.1.2. 创建线程

创建一个新线程需要调用thread::spawn函数并传递一个闭包，示例如下：

```rust
use std::thread;
use std::time::Duration;

fn main() {
    thread::spawn(|| {   // 创建一个线程
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }
}
```

### 3.20.1.3. 等待线程结束

前面的例子并不能保证子线程执行完所有的打印，因为主线程有可能在子线程执行完成前结束从而退出程序，所以需要在主线程中等待子线程结束。等待子线程结束需要调用join方法，示例如下：

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }

    handle.join().unwrap(); // 等待子线程结束
}
```

### 3.20.1.4. 线程与move闭包

move关键字可用于传递给thread::spawn的闭包，获取环境中的值的所有权，从而达到将值的所有权从一个线程传送到另一个线程的目的。示例如下：

```rust
use std::thread;
fn main() {
    let v = vec![1, 2, 3];
    let handle = thread::spawn(move || {  // 将v移动进了闭包，完成了值从主线程到子线程的过程
        println!("Here's a vector: {:?}", v);
    });
    handle.join().unwrap();
}
```

## 3.20.2 传递消息

### 3.20.2.1. 通道简单介绍

Rust中实现消息传递并发的主要工具是通道。通道由发送者和接受者两部分组成：
- 发送者用来发送消息；
- 接收者用来接收消息；
- 发送者或者接收者任一被丢弃时就认为通道被关闭了。
Rust标准库中提供的通道叫做mpsc，是多个生产者，单个消费者的通道，其使用示例如下：

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel(); //创建channel，返回发送者、接收者

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap(); //使用发送者通过channel发送
    });

    let received = rx.recv().unwrap(); //使用接收者通过channel接收
    println!("Got: {}", received);
}
```

关于mpsc通道的使用，有以下几点说明：
- 发送者的send方法返回一个Result类型，如果接收端已经被丢弃了，将没有发送值的目标，所以发送操作将返回错误；
- 接收者的recv方法也返回Result类型，当通道发送端关闭时，将返回一个错误值表明不会再由新的值到来了；
- 接收还可以使用try_recv方法，recv方法会阻塞到一直等待到消息到来，而try_recv不会阻塞，它会立即返回，Ok值标识包含可用信息，而Err则代表此时没有任何信息。

### 3.20.2.2. 通道和所有权

在使用通道时，send 函数会获取参数的所有权并移动这个值归接收者所有。例如下面的代码将会编译错误：

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
        println!("val is {}", val); //错误，此处不能使用val，因为val的所有权已经move到通道里面去了
    });

    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}
```

### 3.20.2.3. 发送多个值示例

利用通道发送多个值的示例如下：
```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("thread"),
        ];

        for val in vals {
            tx.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    for received in rx {
        println!("Got: {}", received);
    }
}
```

### 3.20.2.4. 多个生产者示例

mpsc是multiple producer, single consumer的缩写，此通道可以有多个生产者。相对应的，spmc通道则可以有单个生产者，多个消费者。下面的示例演示多个生产者、单个消费者一起工作：

```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel();
    let tx1 = mpsc::Sender::clone(&tx); //通过clone来使用
    thread::spawn(move || {
        //第一个发送者线程
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("thread"),
        ];

        for val in vals {
            tx1.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    thread::spawn(move || {
        //第二个发送者线程
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("thread"),
        ];

        for val in vals {
            tx.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    for received in rx {
        println!("Got: {}", received);
    }
}
```

上面代码中，有两个发送者发送数据，一个接收者结束数据。

## 3.20.3 共享内存

某些情况下，多个线程之间需要共享内存，即多个线程都访问某个变量。如何在多个线程间安全的共享内存，就需要用到锁、原子操作等机制，下面主要介绍Rust中锁的使用。

### 3.20.3.1. 互斥锁（Mutex）

在任意时刻，互斥锁只允许一个线程访问数据；在使用数据前需获取锁，使用完成后需要释放锁。关于Mutex的用法示例如下：
```rust
use std::sync::Mutex;
fn main() {
    let m = Mutex::new(5);
    {
        let mut num = m.lock().unwrap();
        *num = 6;  // 可以对Mutex内部的值进行修改，可见Mutex和RefCell类似，提供内部可变性
    } // 离开作用域，Mutex<T>的锁会自动释放
    println!("m = {:?}", m);
}
```
Mutex本质上是一个智能指针，lock()返回一个叫做MutexGuard的智能指针，其内部提供了drop方法，所以当MutexGuard离开作用域时会自动释放锁。Mutex获取到锁后，可以对其内部的值进行修改，可见它和RefCell类似，也提供了内部可变性。

### 3.20.3.2. 多线程与多所有权

有了Mutex之后，能对共享数据进行保护；但是由于Rust的所有权机制，每个值都有且仅有一个所有者。下面的代码将无法通过编译：

```rust
use std::sync::Mutex;
use std::thread;
fn main() {
    let counter = Mutex::new(0);
    let mut handles = vec![];
    for _ in 0..10 {
        let handle = thread::spawn(move || {  // 不能将counter移到多个线程中
            let mut num = counter.lock().unwrap();
            *num += 1;
        });
        handles.push(handle);
    }
    for handle in handles {
        handle.join().unwrap();
    }
    println!("Result: {}", *counter.lock().unwrap());
}
```

因此要在多个线程间共享内存，还需要有一种机制能让多个线程都能访问Mutex保护的数据。根据本书目前学到的知识，可以想到Rc智能指针。上面的代码使用Rc智能指针后的示例如下：

```rust
use std::rc::Rc;
use std::sync::Mutex;
use std::thread;
fn main() {
    let counter = Rc::new(Mutex::new(0));
    let mut handles = vec![];
    for _ in 0..10 {
        let counter = Rc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();
            *num += 1;
        });
        handles.push(handle);
    }
    for handle in handles {
        handle.join().unwrap();
    }
    println!("Result: {}", *counter.lock().unwrap());
}
```

上面的代码仍将编译错误，因为Rc是非线程安全的，此时需要使用Arc类型。Arc是一个类似于Rc并可以安全的用于并发环境的类型，使用Arc的示例如下：

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0));  // 使用Arc共享
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

### 3.20.3.3. 读写锁

读变量并不会改变变量的状态，互斥锁Mutex每次读写都会加锁，当在有大量读、少量写的场景时使用Mutex就会效率不高，此时可以使用读写锁RwLock，示例如下：

```rust
use std::sync::{Arc, RwLock};
use std::thread;

fn main() {
    let shared_data = Arc::new(RwLock::new(vec![1, 2, 3])); // 创建一个共享的 vector 数据

    let mut threads = vec![];

    // 创建 5 个读线程
    for i in 0..5 {
        let shared_data = shared_data.clone();
        threads.push(thread::spawn(move || {
            // 获取读锁
            let shared_data = shared_data.read().unwrap();
            println!("Thread {} read data: {:?}", i, *shared_data);
        }));
    }

    // 创建 2 个写线程
    for i in 0..2 {
        let shared_data = shared_data.clone();
        threads.push(thread::spawn(move || {
            // 获取写锁
            let mut shared_data = shared_data.write().unwrap();
            println!("Thread {} write data", i);
            shared_data.push(i); // 向 vector 中添加数据
        }));
    }

    // 等待所有线程完成
    for thread in threads {
        thread.join().unwrap();
    }
}
```

### 3.20.3.4. 小结

关于Mutex和RwLock的选择：

- 追求高并发读取时，使用RwLock，因为Mutex一次只允许一个线程读取；
- 如果要保证写操作的成功性，使用Mutex；
- 不知道哪个合适，统一使用Mutex。

关于Mutex和Arc类型：

- Mutex和RefCell一样具有内部可变性，不过Mutex是线程安全的，而RefCell不是线程安全的；
- Arc和Rc功能类似，但是Arc是线程安全的，而Rc不是线程安全的。

## 3.20.4 Send trait和Sync trait

### 3.20.4.1. Send和Sync

Send trait和Sync trait是Rust语言中的两个标记trait（即未定义任何行为，但是可以标记一个类型），其作用如下：
- 实现了Send的类型可以在线程间安全的传递其所有权；
- 实现了Sync的类型可以在线程间通过引用安全的共享。

实现Sync的类型是通过引用在线程间共享的，因此一个类型要在线程间安全的共享，那么它的应用必须能安全的在线程间进行传递。所以可以有结论：若&T满足Send，那么T满足Sync。

### 3.20.4.2. 实现了Send和Sync的类型

Rust中几乎所有类型都默认实现了Send和Sync。Send和Sync也是可自动派生的trait，因此一个复合类型，如果其内部成员都实现了Send或Sync，那么它就自动实现了Send或Sync。

Rust中绝大多数类型都实现了Send和Sync，但是下面几个是没有实现Send或者Sync的:

- 裸指针既没有实现Send也没有实现Sync，因为它本身就没有任何安全保证的；
- UnsafeCell没有实现Sync，Cell和RefCell也没有实现Sync（但是它们实现了Send）；
- Rc既没有实现Send也没有实现Sync。

通常情况下不需要为某个类型手动实现Send和Sync trait，手动实现这些标记trait 涉及到编写不安全的Rust代码，本书在后面unsafe编程部分介绍。
