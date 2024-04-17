# 枚举与 Option

## 枚举

> 当值可能代表多种事物，**枚举**就很有用。

### 枚举的定义

定义枚举的语法格式如下：

```rust
enum enum_name {
    variant1,
    variant2,
    variant3,
}
```
定义一个简单的枚举例子：

```rust
enum PokerSuit {
    Clubs,
    Spades,
    Diamonds,
    Hearts,
}
```

### 使用枚举

使用枚举的语法格式如下：

```rust
enum_name::variant
```
例如上面的枚举，使用 `Clubs` ，那么赋值语法如下；

```rust
let selected = PokerSuit::Clubs;
// 当需要指明类型时
let selected: PokerSuit = PokerSuit::Clubs;
```

### 一些枚举的不同情况

当需要枚举值带值时，可以使用如下简洁写法：

```rust
enum PokerSuit {
    Clubs(u8),
    Spades(u8),
    Diamonds(u8),
    Hearts(u8),
}
let c1 = PokerCard::Spades(5);
```

或者同一枚举不同成员持有不同的数据类型：

```rust
enum PokerSuit {
    Clubs(u8),
    Spades(u8),
    Diamonds(char),
    Hearts(u8),
}
let c1 = PokerCard::Spades(5);
let c2 = PokerCard::Diamonds('A');
```

任何类型的数据都可以放入枚举成员中：

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
let m1 = Message::Quit;
let m2 = Message::Move{x:1, y:1};
let m3 = Message::ChangeColor(255, 255, 0);
```
- `Quit` 没有任何关联数据
- `Move` 包含一个匿名结构体
- `Write` 包含一个 `String` 字符串
- `ChangeColor` 包含三个 `i32`


## Option 枚举 

### Option 的定义

`Option` 是 Rust 内置的枚举，其具体定义如下：

```rust
enum Option<T> {
    Some(T),    // 用于返回一个值
    None,       // 用于返回 null
}
```
- `Some(T)`，其中 `T` 是泛型参数，`Some(T)` 表示该枚举成员的数据类型是 `T` 。
- `None` 作为没有的意思。

Rust 语言并不支持 `null` 关键字，取而代之的是使用 `None` 作为没有的意思。

### Option 的示例

定义一个函数 `is_even()` 使用 `Option` 枚举作为它的返回值类型。

```rust
fn is_even(no: i32) -> Option<bool> {
    if no % 2 == 0 {
        Some(true)
    } else {
        None
    }
}
```

## 模式匹配

### match 匹配

`match` 的通用形式：

```rust
match target {
    模式1 => 表达式1,
    模式2 => {
        语句1;
        语句2;
        表达式2
    },
    _ => 表达式3,
}
```
match 的匹配必须穷举所有可能，这里的 `_` 代表未列出的所有可能性。

一个简单的示例：

```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => {
            println!("Lucky penny");
            1
        },
        Coin::Nickel => 5,
        Coin::Nickel => 10,
        Coin::Quarter => 25,
    }
}
```

每个分支相关联的代码是一个表达式，而表达式的结果值将作为整个 `match` 表达式的返回值。

### 使用 match 表达式赋值

`match` 本身也是一个表达式，因此可以用它来赋值：

```rust
enum IpAddr {
    Ipv4,
    Ipv6,
}

let ip1 = IpAddr::Ipv6;
let ip_str = match ip1 {
    IpAddr::Ipv4 => "1217.0.0.1"，
    _ => "::1"
}
```
这里结构匹配到了 `_` 分支，因此将 `"::1"` 赋值给 `ip_str` 。

### if-let 匹配

```rust
let v = Some(3u8);
match v {
    Some(3) => println!("three"),
    _ => (),
}
```

上面代码完全可以等效为：

```rust
if let Some(3) = v {
    println!("there");
}
```

你只要匹配一个条件，且忽略其他条件时就用 `if let` ，否则都用 `match`。


## 枚举的使用情况

以下是一些常见适合考虑使用枚举的情况：

1. **表示一组固定的选项：** 当一个变量只能是一组固定的值之一时，比如**方向**（上、下、左、右）、**操作**（添加、删除、修改）等，使用枚举可以清晰、类型安全地表示这些选项。
2. **替代多个布尔变量：** 当需要使用多个布尔变量来表示不同的情况，枚举可以更清晰地表示这些情况，并且更容易扩展。
3. **创建可选类型：** 当有一个可能存在或可能不存在的值时，应该使用 `Option<T>` 。
4. **处理错误：** 在可能产生错误的情况下，使用 `Result` 枚举是处理错误的推荐方式。
5. **定义状态机：** 枚举可以用来定义状态机，其中每个变体代表一个可能的状态。在游戏开发、网络协议处理等方面非常有用。
6. **创建复杂的数据结构：** 枚举可以携带数据，用来创建复杂的数据结构，如链表、树、图等。
7. **匹配多个条件：** 使用 `match` 表达式可以针对枚举的不同变体执行不同的代码块，在处理多个条件分支时比使用多个 `if-else` 更清晰。 
8. **增加代码的可读性和可维护性：** 枚举可以提高代码的可读性。

### 结构体和枚举的区分

- 当需要表示一个对象或实体，并且这个对象和实体具有固定数量的字段，使用**结构体**。
- 当需要表示一组有限的选项和状态，尤其是当这些选项和状态需要附带不同类型的数据时，使用**枚举**。



## 一个综合示例

```rust
enum Message {
    Quit,
    Move(Point),
    Echo(String),
    ChangeColor(u8, u8, u8),
}

struct Point {
    x: u8,
    y: u8,
}

struct State {
    color: (u8, u8, u8),
    position: Point,
    quit: bool,
    message: String
}

impl State {
    fn change_color(&mut self, color: (u8, u8, u8)) {
        self.color = color;
    }

    fn quit(&mut self) {
        self.quit = true;
    }

    fn echo(&mut self, s: String) { self.message = s }

    fn move_position(&mut self, p: Point) {
        self.position = p;
    }

    fn process(&mut self, message: Message) {
        match message {
            Message::Quit => self.quit(),
            Message::Echo(s) => self.echo(s),
            Message::Move(p) => self.move_position(p),
            Message::ChangeColor(r, g, b) => self.change_color((r, g, b)),
        }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_match_message_call() {
        let mut state = State {
            quit: false,
            position: Point { x: 0, y: 0 },
            color: (0, 0, 0),
            message: "hello world".to_string(),
        };
        state.process(Message::ChangeColor(255, 0, 255));
        state.process(Message::Echo(String::from("hello world")));
        state.process(Message::Move(Point { x: 10, y: 15 }));
        state.process(Message::Quit);

        assert_eq!(state.color, (255, 0, 255));
        assert_eq!(state.position.x, 10);
        assert_eq!(state.position.y, 15);
        assert_eq!(state.quit, true);
        assert_eq!(state.message, "hello world");
    }
}
```

