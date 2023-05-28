# 3.17.1 包、crate和模块介绍

关于三者的描述如下：

- 包（Package）是一个 Cargo 项目，它包含了一个 Cargo.toml 配置文件和一些 Rust 代码文件。一个包可以包含多个 crate，并且可以有多种构建方式和配置。
- crate（Crate）是一个可以编译成库或二进制文件的 Rust 代码单元。每个 crate 都有一个唯一的名称，它可以被其他 crate 引用，也可以被其它包中的代码引用。每个crate都有一个crate root ，它是一个源文件，Rust 编译器以它为起始点来构成crate根模块。对于crate来说，crate root要么是`src/main.rs`（对于二进制crate来说），要么是`src/lib.rs`（对于库crate来说）。
- 模块（Module）是一种组织 Rust 代码的方式，它可以将相关的代码放在一起，形成一个代码块。模块可以嵌套，可以有多个不同的访问级别（public、private），并且可以从其他模块中引用和访问。

三者的范围为：模块< crate < 包，即一个包包含一个或多个crate，一个crate可以包含多个模块。

## 1. 包含一个crate的包

下面的命令会创建一个Rust工程，这个工程同时是一个crate，也是一个包（只有一个crate的包）：

```bash
cargo new main
```

进入main文件夹，其目录层级如下：

![注释](.././assets/35.png)

其中Cargo.toml的内容如下：

![注释](.././assets/36.png)

可以看到这个crate的名字为main。

## 2. 包含多个crate的包

下面再创建一个稍微复杂一点的包。执行如下几条命令：

```bash
mkdir my-pack            # 创建包文件夹
cd my-pack
cargo new my-lib --lib   # 创建一个库crate
cargo new main           # 创建一个可执行的二进制crate
```

然后在my-pack文件夹中添加文件Cargo.toml，其内容如下：

```TOML
[workspace]

members = [
        "main",
        "my-lib",
]
```

最后的目录层级关系如下：

![注释](.././assets/37.png)

上面的步骤就创建了一个包my-pack，这个包包含两个crate，分别是my-lib和main。
