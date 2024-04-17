# 模块

## 创建嵌套模块

使用 `cargo new --lib restaurant` 创建一个库类型的 `Package`，然后在 `src/lib.rs` 中写入如下代码：

```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}

        fn seat_at_table() {}
    }

    mod serving {
        fn take_order() {}

        fn serve_order() {}

        fn take_payment() {}
    }
}
```

有以下几点需要注意：
- 使用 `mod` 关键字创建新模块，后面紧跟着模块名称。
- 模块可以嵌套。
- 模块中可以定义各种 Rust 类型，例如函数、结构体、枚举、特征等。
- 所有模块均定义在同一个文件内。

使用模块，我们就能将功能相关的代码组织到一起，然后通过一个模块名称来说明这些代码为何被组织在一起。

## 模块树

```bash
crate
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment

```
这颗树展示了模块之间彼此的嵌套关系，因此被称为模块树。

> 如果模块 A 包含模块 B，那么 A 是 B 的父模块，B 是 A 的子模块。

## 路径引用模块

### 绝对路径引用

**绝对路径**，从包根开始，路径名以包名或者 crate 作为开头

```rust
crate::front_of_house::hosting::add_to_waitlist();
```

### 相对路径引用

**相对路径**，从当前模块开始，以 `self`，`super` 或当前模块的标识符作为开头

**当前模块标识符引用：**

```rust
front_of_house::hosting::add_to_waitlist();
```

**super 引用：**

`super` 代表的是父模块为开始的引用方式，类似于文件系统的 `..` 语法。

```rust
fn serve_order() {}

mod back_of_house {
    fn fix_incorrect_order() {
        cook_order();
        super::serve_order();
    }
    fn cook_order() {}
}
```

**self引用：**

`self` 代表的是引用自身模块中的项。

```rust
fn serve_order() {
    self::back_of_house::cook_order()
}

mod back_of_house {
    fn fix_incorrect_order() {
        cook_order();
        crate::serve_order();
    }

    pub fn cook_order() {}
}
```

### 怎样选择路径引用？

遵循一个原则：**当代码被挪动位置时，尽量减少引用路径的修改**

如果不确定哪个好，你可以考虑优先使用绝对路径。

## 代码可见性

Rust 出于安全的考虑，默认情况下，所有的类型都是私有化的，包括函数、方法、结构体、枚举、常量，是的，就连模块本身也是私有化的。



Rust 还提供了 `pub` 关键字，通过它你可以控制模块和模块中指定项的可见性。

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}
```

这样 `add_to_waitlist` 即可对外可见。

### 结构体和枚举的可见性

- 将结构体设置为 `pub` ，但其所有字段依然是私有的。
- 将枚举设置为 `pub` ，其所有字段对外可见。

## 模块与文件分离

目前所有模块都定义在 `src/lib.rs` 中，当模块变多或变大时，需要将模块放入一个单独的文件中，让代码更好维护。

1. 首先把 `front_of_house` 分离出来，放入单独的文件中 `src/front_of_house.rs` 。

```rust
pub mod hosting {
    pub fn add_to_waitlist() {}
}
```

2. 之后在 `src/lib.rs` 留下如下代码

```rust
mod front_of_house;
pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
    hosting::add_to_waitlist(); 
}
```

`mod front_of_house;` 告诉 Rust 从另一个模块 `front_of_house` 文件中加载该模块的内容。

## 使用 use 

在 Rust 中，可以使用关键字 `use` 把路径提前引入当前作用域中，随后的调用即可省略该路径。

### 绝对路径引入模块

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();   
}
```

使用 `use` 和绝对路径的方式，将 `hosting` 模块引入到当前作用域中，然后只需通过 `hosting::add_to_waitlist` 的方式，即可调用目标模块中的函数。

### 相对路径引入模块

使用相对路径直接引入该模块中的 `add_to_waitlist` 函数：

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use front_of_house::hosting::add_to_waitlist;

pub fn eat_at_restaurant() {
    add_to_waitlist();
    add_to_waitlist();
    add_to_waitlist();
}
```

### 避免同名引用

**模块::函数**

```rust
use std::fmt;
use std::io;

fn function1() -> fmt::Result {
    // --snip--
}

fn function2() -> io::Result<()> {
    // --snip--
}
```

**as别名引用**

```rust
use std::fmt::Result;
use std::io::Result as IoResult;

fn function1() -> Result {
    // --snip--
}

fn function2() -> IoResult<()> {
    // --snip--
}
```

### 引入项再导出

当外部的模块项 `A` 被引入到当前模块中时，它的可见性自动被设置为私有的，如果你希望允许其它外部代码引用我们的模块项 `A`，那么可以对它进行再导出：

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
}
```

这里 `use` 代表引入 `hosting` 模块到当前作用域，`pub` 表示将该引入的内容再度设置为可见。

### 使用第三方包

1. 修改 `Cargo.toml` 文件，在 `[dependencies]` 区域添加一行：`rand = "0.8.3"`

2. `rand` 包被添加到依赖后，可以在代码中使用

```rust
use rand::Rng;

let secret_number = rand::thread_rng().gen_range(1..101);
```

- [lib.rs]([Lib.rs — home for Rust crates // Lib.rs](https://lib.rs/))：查找包，搜索功能强大，内容展示更合理。
- [crates.io]([crates.io: Rust Package Registry](https://crates.io/))：下载依赖包。

### 使用 {} 简化引用方式

```rust
use std::collections::HashMap;
use std::collections::BTreeMap;
use std::collections::HashSet;

use std::cmp::Ordering;
use std::io;
```

使用 `{}` 简化：

```rust
use std::collections::{HashMap,BTreeMap,HashSet};
use std::{cmp::Ordering, io};
```

### self

同时引入模块和模块中的项：

```rust
use std::io;
use std::io::Write;
```

可以使用 `{}` 和 `self` 的方式进行简化:

```rust
use std::io::{self, Write};
```

- `use self::xxx` 表示加载当前模块中的 `xxx`，此处 `self` 可省略。
- `use xxx::{self, yyy}` 表示加载当前路径下模块 `xxx` 本身，以及模块 `xxx` 下的 `yyy` 。

### 使用 * 引入所有项

使用 `*` 引入模块下的所有项：

```rust
use std::collections::*;
```

## 受限可见性

**受限可见性（Restricted Visibility）**是指一个模块或类型只能被模块内的代码访问，而不能被模块外的代码访问。

`pub` 有几种不同的形式，决定了项的可见性范围：

- `pub`：可见性无任何限制。
- `pub(crate)`：在当前包可见。
- `pub(self)`：在当前模块可见。
- `pub(super)`：在父模块可见。
- `pub(in <path>)`：在某个路径代表的模块中可见，其中 `path` 必须是父模块或者祖先模块。

假设我们有包结构如下：

```bash
crate root
  mod parent
  	mod child
      mod sub_child
```

```rust
// 在父模块中
mod parent {
    pub(crate) struct ParentOnly {
        value: i32,
    }

    pub(self) struct SelfOnly {
        value: i32,
    }

    pub(super) struct SuperOnly {
        value: i32,
    }

    pub(in crate::child) struct InChild {
        value: i32,
    }

    pub struct Public {
        value: i32,
    }
}

// 在子模块中
mod child {
    pub struct Public {
        value: i32,
    }
}

// 在孙模块中
mod sub_child {
    pub struct Public {
        value: i32,
    }
}

// 在根模块中
fn main() {
    // 正确访问不同可见性的结构体
    parent::ParentOnly { value: 1 }; // 正确，因为我们在parent模块内
    parent::SelfOnly { value: 1 };  // 正确，因为我们在parent模块内
    parent::SuperOnly { value: 1 }; // 正确，因为我们在parent模块内
    parent::InChild { value: 1 };  // 正确，因为我们在parent模块内
    parent::Public { value: 1 };   // 正确，因为它是pub，可以在任何地方访问
}

// 在子模块中
fn in_child() {
    parent::ParentOnly { value: 1 }; // 正确，因为我们在child模块内
    parent::SelfOnly { value: 1 };  // 正确，因为我们在child模块内
    parent::SuperOnly { value: 1 }; // 正确，因为我们在child模块内
    parent::InChild { value: 1 };  // 正确，因为我们在child模块内
    parent::Public { value: 1 };   // 正确，因为它是pub，可以在任何地方访问
}

// 在孙模块中
fn in_sub_child() {
    parent::ParentOnly { value: 1 }; // 编译时错误，因为ParentOnly是pub(crate)
    parent::SelfOnly { value: 1 };  // 编译时错误，因为SelfOnly是pub(self)
    parent::SuperOnly { value: 1 }; // 编译时错误，因为SuperOnly是pub(super)
    parent::InChild { value: 1 };  // 编译时错误，因为InChild是pub(in crate::child)
    parent::Public { value: 1 };   // 正确，因为它是pub，可以在任何地方访问
}
```

