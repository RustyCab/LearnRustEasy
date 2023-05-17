Slice类型
Slice（切片）类型，表示引用集合中一段连续的元素序列。Slice是一类引用，没有所有权。Rust常见类型中，有三种支持Slice的操作，分别是String、数组、Vec类型。

1. Slice类型
假定s是可被切片的数据，则对应的操作有：
- s[n1..n2]：获取s中index=n1到index=n2(不包括n2)之间的所有元素；
- s[n1..]：获取s中index=n1到最后一个元素之间的所有元素；
- s[..n2]：获取s中第一个元素到index=n2(不包括n2)之间的所有元素；
- s[..]：获取s中所有元素；
- 其他表示包含范围的方式，如s[n1..=n2]表示取index=n1到index=n2(包括n2)之间的所有元素。

Rust中几乎总是使用切片数据的引用。切片数据的引用对应的数据类型描述为&[T]或&mut [T]，前者不可通过Slice引用来修改源数据，后者可修改源数据。示例如下：
fn main(){
  let mut arr = [11,22,33,44];

  let arr_slice1 = &arr[..=1];
  println!("{:?}", arr_slice1); // [11,22];

  let arr_slice2 = &mut arr[..=1];  
  arr_slice2[0] = 1111;
  println!("{:?}", arr_slice2);// [1111,22];
  println!("{:?}", arr);// [1111,22,33,44];
}
Slice类型是一个胖指针，包含两个字段：
- 第一个字段是指向源数据中切片起点元素的指针；
- 第二个字段是切片数据中包含的元素数量，即切片的长度。

2. String的切片类型
String的切片类型为&str而不是&String，其使用方式如下：
fn main() {
    let s = String::from("hello world!");
    let s1 = &s[6..];   // 切片类型&str
    let s2 = &s;        // 引用类型&String
    println!("{:?}", s1);
    println!("{:?}", s2);
}

&str和&String的内存布局如下：
![注释](../../assets/13.png)
