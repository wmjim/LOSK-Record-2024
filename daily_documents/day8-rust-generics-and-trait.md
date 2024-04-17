# 泛型与特征

## 泛型 

泛型（Generics）是编程语言中的一种特性，允许你编写可重用的代码组件，这些组件可以适用于多种数据类型，而不是只针对单一数据类型。

一个简单泛型使用代码示例：

```rust
fn largest<T>(list: &[T]) -> T {}
```

首先 `largest<T>` 对泛型参数 `T` 进行了声明，其中泛型参数名称 `T` 可以随意起，出于惯例使用 `T` （Type）作为首选。之后，函数参数中使用该泛型参数 `list: &[T]` ，最后该函数的返回值类型也是 `T` 。

### 结构体中使用泛型

结构体中的字段类型可以使用泛型来定义：

```rust
struct Point<T> {
    x: T,
    y: T,
}
```

当需要 `x` 和 `y` 类型不同时，可以使用不同的泛型参数：

```rust
struct Point<T, U> {
	x: T,
    y: U,
}
```

### 枚举中使用泛型

泛型枚举允许你在不同枚举的变体中使用不同的类型：

```rust
enum Option<T> {
    Some(T),
    None,
}

enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

`Option` 主要用于函数返回值，`Result` 关注的是值的正确性。

- `Option<T>` 表示一个值可能是 `Some(T)` ，其中 `T` 是任何类型，或者没有值 `None` 。
- `Result<T, E>` 表示一个操作的成功结果 `Ok(T)` 和错误结果 `Err(E)` ，其中 `T` 是操作成功时的结果类型，`E` 是操作失败时的错误类型。

### 方法中使用泛型

**为泛型结构体定义方法：**

```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}
```

使用泛型参数前，依然需要提前声明：`impl<T>` ，只有提前声明了，才能在 `Point<T>` 中使用它，这样 Rust 才知道 `Point` 的尖括号中的类型是泛型而不是具体类型。需要主义的是，这里的 `Point<T>` 不再是泛型声明，而是一个完整的结构体类型`Point<T>`。



在此示例中，`Point` 是一个泛型结构体，它有一个类型参数 `T` 。在 `impl` 块中，我们为 `Point<T>` 定义了一个方法 `x` 。这个方法返回 `x` 字段的引用，由于 `Point` 是泛型的，这个 `x` 方法可以适用于任何类型的 `Point` 实例。

**为具体的泛型类型实现方法：**

```rust
impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}
```

`Point<f32>` 类型有方法 `distance_from_origin` ，其它 `T` 不是 `f32` 类型的 `Point<T>` 实例则没有定义此方法。

## 特征

`trait` 告诉 Rust 编译器某个特定类型拥有可能与其他类型共享的功能。可以通过 `trait` 以一种抽象的方式定义共享的行为。

### 定义特征

一个类型的行为由其可供调用的方法构成。如果可以对不同类型调用相同的方法的话，这些类型就可以共享相同的行为了。`trait` 定义是一种将方法签名组合起来的方法，目的是定义一个实现某些目的所必需的行为的集合。

```rust
pub trait Summary {
    fn summarize(&self) -> String;
} 
```

### 为类型实现特征

特征只定义行为看起来是什么样的，因此我们需要为类型实现具体的特征，定义行为具体是怎么样的。

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}
pub struct Post {
    pub title: String,
    pub author: String,
    pub content: String,
}

impl Summary for Post { // 为 Post 类型实现 Summary 特征
    fn summarize(&self) -> String {
        format!("文章{}，作者是{}", self.title, self.author)
    }
}

pub struct Weibo {
    pub username: String,
    pub content: String,
}

impl Summary for Weibo {
    fn summarize(&self) -> String {
        format!("{}发表了微博{}", self.username, self.content)
    }
}
```

> **孤儿规则：** 如果你想要为类型 `A` 实现特征 `T` ，那么 `A` 或者 `T` 至少有一个是在当前作用域中定义的。

例如，可以为上面的 `Post` 类型实现标准库中的 `Display` 特征，或者为 `String` 类型实现 `Summary` 特征，这是因为 `Post` 和 `Summary` 都是定义在当前作用域内。反例，不能为 `String` 类型实现 `Display` 特征。

### 默认实现

在特征定义具有默认实现的方法，其它类型无需再实现该方法，当然也可以选择重载该方法。

```rust
pub trait Summary {
    fn summarize(&self) -> String {
        String::from("Read more...")
    }
}

// Post 选择默认实现
impl Summary for Post {}
// Weibo 重载该方法
impl Summary for Weibo {
    fn summarize(&self) -> String {
        format!("{}发表了微博{}", self.username, self.content)
    }
}
```

### 使用特征作为函数参数

```rust
pub fn notify(item: &impl Summary) { // 实现了 Summary 特征的 item 参数
	println!("Breaking news! {}", item.summarize());	
}
```

可以使用任何实现了 `Summary` 特征的类型作为该函数的参数，同时在函数体内，还可以调用该特征的方法。

### 特征约束

```rust
pub fn notify<T: Summary>(item: &T, item2: &T) {}
```

形如 `T:Summary` 被称为特征约束，其目的是为了限制泛型类型参数必须满足的条件。

**多重约束：**

可以指定多个约束条件

```rust
pub fn notify(item: &(impl Summary + Display)) {} 
// 使用特征约束的形式
pub fn notify<T: Summary + Display>(item: &T) {}
```

**where 约束：**

当特征约束变得很多时，可以使用 where 约束，使约束看起来清晰：

```rust
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {}
// 使用 where 约束
fn some_function<T, U>(t: &T, u: &U) -> i32
	where T: Dispaly + Clone,
		  U: Clone + Debug
{}
```

**使用特征约束有条件地实现方法或特征：**

```rust
trait Fly {
    fn fly(&self);
}

struct Bird;
struct Airplane;

impl Fly for Bird {
    fn fly(&self) {
        println!("The bird is flying.");
    }
}

impl Fly for Airplane {
    fn fly(&self) {
        println!("The airplane is flying.");
    }
}

fn make_it_fly<T: Fly>(item: T) {
    item.fly();
}
```

通过这种方式，`make_it_fly` 函数可以接受任何实现了 `Fly` 特征的对象，无论是 `Bird` 还是 `Airplane` 。

### 函数返回中的 impl Tarit

通过 `impl Trait` 来说明一个函数返回了一个类型，该类型实现了某个特征：

```rust
fn returns_summarizable() -> impl Summary {
    Weibo {
        username: String::from("MAC M1"),
        content: String::from(
            "m1 max太厉害了，电脑再也不会卡",
        )
    }
}
```

因为 `Weibo` 实现了 `Summary` ，因此这里可以用它来作为返回值。

## 派生特征

### 用于开发者输出的 Debug

`Debug` 特征可以让指定对象输出调试格式的字符串，通过在 `{}` 占位符中增加 `:?` 表明：

```rust
use std::fmt;

struct Point {
    x: i32,
    y: i32,
}

// 为 Point 结构体自动派生 Debug 特征
#[derive(Debug)]
struct Rectangle {
    top_left: Point,
    bottom_right: Point,
}

let rect = Rectangle {
    top_left: Point { x: 0, y: 0 },
    bottom_right: Point { x: 10, y: 10 },
};
println!("rect: {:?}", rect);
```

### 等值比较的 PartialEq 和 Eq

`PartialEq `特征是一个标准库提供的特征，它允许比较两个值是否相等。这个特征通常用于定义自定义类型的相等性规则，例如结构体或枚举。`PartialEq` 特征允许比较两个值是否相等，但不要求比较操作是可交换的（即 `a == b` 和 `b == a `可能会有不同的结果）。

```rust
use std::cmp::PartialEq;

struct Point {
    x: i32,
    y: i32,
}

// 为 Point 结构体自动派生 PartialEq 特征
#[derive(ParialEq)]
struct Rectangle {
    top_left: Point,
    bottom_right: Point,
}

let rect1 = Rectangle {
    top_left: Point { x: 0, y: 0 },
    bottom_right: Point { x: 10, y: 10 },
};
let rect2 = Rectangle {
    top_left: Point { x: 0, y: 0 },
    bottom_right: Point { x: 10, y: 10 },
};

if rect1 == rect2 {
    println!("Both rectangles are equal.");
} else {
    println!("The rectangles are not equal.");
}
```

`Eq` 特征是一个标准库提供的特征，它允许比较两个值是否完全相等。与 `PartialEq` 特征相比，`Eq` 特征要求比较操作是可交换的，即 `a == b` 和 `b == a` 的结果必须相同。`Eq` 特征通常用于定义自定义类型的完全相等性规则，例如结构体或枚举。

```rust
use std::cmp::PartialEq;

struct Point {
    x: i32,
    y: i32,
}

#[derive(PartialEq)]
struct Rectangle {
    top_left: Point,
    bottom_right: Point,
}

let rect1 = Rectangle {
    top_left: Point { x: 0, y: 0 },
    bottom_right: Point { x: 10, y: 10 },
};
let rect2 = Rectangle {
    top_left: Point { x: 0, y: 0 },
    bottom_right: Point { x: 10, y: 10 },
};

if rect1 == rect2 {
    println!("Both rectangles are equal.");
} else {
    println!("The rectangles are not equal.");
}
```

### 次序比较的 PartialOrd 和 Ord

`PartialOrd `特征是一个标准库提供的特征，它允许比较两个值是否可以相互比较大小，但不要求比较操作是可交换的（即 `a < b` 和 `b < a` 可能会有不同的结果）。`PartialOrd` 特征通常用于定义自定义类型的部分排序规则，例如结构体或枚举。

```rust
use std::cmp::PartialOrd;

struct Point {
    x: i32,
    y: i32,
}

#[derive(PartialOrd)]
struct Rectangle {
    top_left: Point,
    bottom_right: Point,
}

let rect1 = Rectangle {
    top_left: Point { x: 0, y: 0 },
    bottom_right: Point { x: 10, y: 10 },
};
let rect2 = Rectangle {
    top_left: Point { x: 0, y: 0 },
    bottom_right: Point { x: 10, y: 10 },
};
if rect1 < rect2 {
    println!("rect1 is less than rect2.");
} else if rect1 > rect2 {
    println!("rect1 is greater than rect2.");
} else {
    println!("Both rectangles are equal.");
}
```

`Ord` 特征是一个标准库提供的特征，它允许比较两个值是否可以完全相互比较大小，并且比较操作是可交换的，即 `a < b `和 `b < a `的结果必须相同。`Ord` 特征通常用于定义自定义类型的完全排序规则，例如结构体或枚举。

```rust
use std::cmp::Ord;

struct Point {
    x: i32,
    y: i32,
}

#[derive(Ord)]
struct Rectangle {
    top_left: Point,
    bottom_right: Point,
}

let rect1 = Rectangle {
    top_left: Point { x: 0, y: 0 },
    bottom_right: Point { x: 10, y: 10 },
};
let rect2 = Rectangle {
    top_left: Point { x: 0, y: 0 },
    bottom_right: Point { x: 10, y: 10 },
};
if rect1 < rect2 {
    println!("rect1 is less than rect2.");
} else if rect1 > rect2 {
    println!("rect1 is greater than rect2.");
} else {
    println!("Both rectangles are equal.");
}
```

### 复制值的 Clone 和 Copy

`Clone `特征是一个标准库提供的特征，它允许一个类型的实例通过复制其数据来创建一个新的实例。这意味着一个类型必须实现 `Clone ` 特征，才能被复制。

```rust
use std::clone::Clone;

struct Point {
    x: i32,
    y: i32,
}

#[derive(Clone)]
struct Rectangle {
    top_left: Point,
    bottom_right: Point,
}

let rect = Rectangle {
    top_left: Point { x: 0, y: 0 },
    bottom_right: Point { x: 10, y: 10 },
};

let rect_clone = rect.clone();
```

`Copy` 特征是一个标准库提供的特征，它允许一个类型的实例通过简单的值复制来创建一个新的实例。这意味着一个类型必须实现 `Copy` 特征，才能被复制。与 `Clone` 特征不同，`Copy` 特征不涉及任何深层数据结构的复制，它只是简单地将原始值复制到新位置。

```rust
use std::marker::Copy;

struct Number(i32);

#[derive(Copy)]
struct Point {
    x: i32,
    y: i32,
}

let point = Point { x: 0, y: 0 };
// 复制`Point`实例
let point_clone = point;
```

### 固定大小的值映射的 Hash

`Hash` 特征是一个标准库提供的特征，它允许一个类型的实例通过哈希函数来生成一个哈希值。这个哈希值可以用来进行哈希表的键值对存储，或者作为对象的唯一标识符。

```rust
use std::hash::Hash;

struct Point {
    x: i32,
    y: i32,
}

#[derive(Hash)]
struct Rectangle {
    top_left: Point,
    bottom_right: Point,
}

let rect = Rectangle {
    top_left: Point { x: 0, y: 0 },
    bottom_right: Point { x: 10, y: 10 },
};
// 使用`Hash`特征的`hash`方法生成哈希值
let hash_value = rect.hash();
```

