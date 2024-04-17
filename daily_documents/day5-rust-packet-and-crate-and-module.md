# Rust 项目组织管理

Rust 提供了以下概念用于项目代码的组织管理。

## 模块

> 在 Rust 中，**模块（Module）** 是一个用于将代码按逻辑单元划分的工具，它允许将相关联的项（常量、函数、结构、枚举、特性、其它模块等）放置在一个命名空间中。

定义模块的语法如下：

```rust
// 定义一个名为 module_name 的模块
mod module_name {
    // 代码
}
```

## 包

> 在 Rust 中，**包（Package）** 是一个或多个 crate 的集合，提供一组功能。

创建一个名为 `my_package` 的包：

```bash
$ cargo new my_package
$ tree my_package/
my_package/
├── Cargo.toml
└── src
    └── main.rs

2 directories, 2 files
```
- `Cargo.toml` 文件包含包的元数据和依赖信息。
- `src` 目录包含 Rust 代码。
- `main.rs` 是默认的二进制可执行文件入口。

## 箱

> 在 Rust 中，**箱（Crate）** 是编译单位。

当你运行 `rustc my_crate.rs` 时，`my_crate.rs` 被视为箱文件。如果  `my_crate.rs` 中有 `mod` 声明，那么模块文件的内容将被插入到箱文件中的 `mod` 声明处，然后再运行编译器。模块不会单独编译，只有箱才会被编译。

Rust 有两种类型的箱，分别是 **库类型（lib）** 和 **二进制类型（bin）** 。

### 二进制箱

创建一个名为 `my_bin` 的包：

```bash
$ cargo new my_bin --bin
$ tree my_bin/
my_bin/
├── Cargo.toml
└── src
    └── main.rs

2 directories, 2 files
```

在当前目录下创建新的目录 `my_bin` ，其中包含一个 `Cargo.toml` 文件和一个 `src` 目录，`src` 目录下的 `main.rs` 文件就是这个二进制类型箱的根，该二进制箱的箱（Crate）名和所属包（Package）名相同，在这里都是 `my_bin` 。

### 库类型箱

创建一个名为 `my_lib` 的包：

```rust
$ cargo new my_lib --lib
$ tree my_lib
my_lib
├── Cargo.toml
└── src
    └── lib.rs

2 directories, 2 files
```

在当前目录下创建新的目录 `my_lib` ，其中包含一个 `Cargo.toml` 文件和一个 `src` 目录，`src` 目录下的 `lib.rs` 文件就是这个库类型箱的根，该库类型箱的箱（Crate）名和所属包（Package）名相同，在这里都是 `my_lib` 。

> 一个 Package 只能包含 **一个库类型** 的箱，但是可以包含 **多个二进制类型** 的箱。

## 工作空间

> 在 Rust 中，**工作空间（Workspace）** 是由多个包（Package）组成的集合，它们共享一个 Cargo.lock文件、输出目录和一些设置。

工作空间的主要目的是在大型项目中组织和管理多个相关的包，当项目增长到包含多个库（Crate），你可能希望将它们进一步拆分为多个库（Crate），工作空间就是用来管理这种情况的。

下面是一个工作空间的例子：

```bash
rust-analyzer/
 Cargo.toml
 Cargo.lock
 crates/
   rust-analyzer/
   hir/
   hir_def/
   hir_ty/
   ...
```
在这个例子中，Cargo.toml 文件定义了一个虚拟清单，它指定了工作空间的成员：

```toml
[workspace]
members = ["crates/*"]
```

## 真实项目包结构

一个真实项目中的包（Package），会包含多个二进制类型箱（Crate），这些箱文件会被放在 `src/bin` 目录下，每一个箱文件都是独立的二进制类型箱，同时该包中也会包含一个库类型箱（Crate），该箱只能存在一个 `src/lib.rs` 文件：

```bash
$ tree my_project
my_project
├── Cargo.toml
├── Cargo.lock
├── src
│   ├── main.rs
│   ├── lib.rs
│   └── bin
│       └── main1.rs
│       └── main2.rs
├── tests
│   └── some_integration_tests.rs
├── benches
│   └── simple_bench.rs
└── examples
    └── simple_example.rs
```

- 唯一库类型箱：`src/lib.rs`
- 默认二进制类型箱：`src/main.rs`，编译后生成的可执行文件与包（Package）同名。
- 其余二进制类型箱：`src/bin/main1.rs` 和 `src/bin/main2.rs` ，它们会分别生成与文件同名的二进制可执行文件。
- 集成测试文件：`test` 目录下。
- 基准性能测试 `benchmark` 文件：`benches` 目录下。
- 项目示例：`examples` 目录下。