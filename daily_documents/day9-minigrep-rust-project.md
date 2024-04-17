# 文件搜索命令行工具

**目标**：从命令行参数中读取指定的文件名和字符串，然后在相应的文件中找到包含该字符串的内容，最终打印出来。

```bash
cargo run -- searchstring example-filename.txt
```

## 基本功能

1. **创建一个新的项目`minigrep`** 

```bash
cargo new minigrep
```

2. **接收命令行参数**

```rust
// in main.rs
use std::env;

fn main() {
    // env::args 方法会读取并分析传入的命令行参数
    // collect 方法输出一个集合类型
    let args: Vec<String> = env::args().collect();
    // dbg! 宏输出读取到的数组内容
    dbg!(args);
}
```

无参数输入执行：

```bash
$ cargo run
   Compiling minigrep v0.1.0 (/home/wm/items/rust-learn-path/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.75s
     Running `target/debug/minigrep`
[src/main.rs:8:5] args = [
    "target/debug/minigrep",
]
```

有参数输入执行：

```bash
$ cargo run -- needle haystack
    Finished dev [unoptimized + debuginfo] target(s) in 0.01s
     Running `target/debug/minigrep needle haystack`
[src/main.rs:8:5] args = [
    "target/debug/minigrep",
    "needle",
    "haystack",
]
```

3. **存储读取到的参数**

```rust
fn main() {
    // env::args 方法会读取并分析传入的命令行参数
    // collect 方法输出一个集合类型
    let args: Vec<String> = env::args().collect();
    // dbg! 宏输出读取到的数组内容
    // dbg!(args);
    let query = &args[1];   // 待搜索的字符串
    let file_path = &args[2];   // 存储文件路径

    println!("Searching for {}", query);
    println!("In file {}", file_path); 
}
```

运行测试：

```bash
$ cargo run -- test sample.txt
   Compiling minigrep v0.1.0 (/home/wm/items/rust-learn-path/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.56s
     Running `target/debug/minigrep test sample.txt`
Searching for test
In file sample.txt
```

4. **文件读取**

项目根目录创建文件 `poem.txt` ，并继续增加代码：

```rust
use std::env;
use std::fs;

fn main {
    ... // 省略之前的代码
    // fs::read_to_string 读取指定的文件内容
    let contents = fs::read_to_string(file_path)
    .expect("Should have been able to read the file.");

    println!("With text:\n{contents}");
}
```

运行测试：

```bash
$ cargo run -- the poem.txt
   Compiling minigrep v0.1.0 (/home/wm/items/rust-learn-path/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.69s
     Running `target/debug/minigrep the poem.txt`
Searching for the
In file poem.txt
With text:
I'm nobody! Who are you?
我啥也不是，你呢？
Are you nobody, too?
牛逼如你也是无名之辈吗？
Then there's a pair of us - don't tell!
那我们就是天生一对，嘘！别说话！
They'd banish us, you know.
你知道，我们不属于这里。
How dreary to be somebody!
因为这里属于没劲的大人物！
How public, like a frog
他们就像青蛙一样呱噪，
To tell your name the livelong day
成天将自己的大名
To an admiring bog!
传遍整个无聊的沼泽！
```

## 增加模块化和错误处理

### 分离 main 函数

Rust 社区给出处理庞大 `main` 函数的统一指导方案：

- 将程序分割为 `main.rs` 和 `lib.rs` ，并将程序的逻辑代码移动到 `lib.rs` 。

按照此方案，整理 `main` 函数应该包含的功能：

1. 解析命令行参数
2. 初始化其它配置
3. 调用 `lib.rs` 中的 `run` 函数，以启动逻辑代码
4. 如果 `run` 返回一个错误，则需要进行错误处理

> `main.rs` 负责启动程序，`lib.rs` 负责逻辑代码的运行。

1. **分离命令行解析**

将命令行解析分离到一个单独的函数中，并将该函数放置在 `main.rs` 中：

```rust
// in main.rs
...
fn main(){
    // env::args 方法会读取并分析传入的命令行参数
    // collect 方法输出一个集合类型
    let args: Vec<String> = env::args().collect();
    // dbg! 宏输出读取到的数组内容
    // dbg!(args);
    let (query, file_path) = parse_config(&args);
    ...
}

fn parse_config(args: &[String]) -> (&str, &str) {
    let query = &args[1];   // 待搜索的字符串
    let file_path = &args[2];   // 存储文件路径

    (query, file_path)
}
```

2. **聚合配置变量**

使用一个结构体来统一存放配置变量。

```rust
fn main() {
    ...
    let config = parse_config(&args);

    println!("Searching for {}", config.query);
    println!("In file {}", config.file_path);
    let contents = fs::read_to_string(config.file_path)
    	.expect("Should have been able to read the file.");
    ...
} 
struct Config {
    query: String,
    file_path: String,
}

fn parse_config(args: &[String]) -> Config {
    let query = args[1].clone();   // 待搜索的字符串
    let file_path = args[2].clone();   // 存储文件路径

    Config { query, file_path }
}
```

通过构造函数来初始化一个 `Config` 实例，而不是直接通过函数返回实例。

```rust
fn main() {
    ...
    let config = Config::new(&args);
    ...
}
...
impl Config {
    fn new(args: &[String]) -> Config {
        let query = args[1].clone();   // 待搜索的字符串
        let file_path = args[2].clone();   // 存储文件路径
    
        Config { query, file_path }
    }
}
```

### 错误处理

当用户不输入任何命令行参数时，`args` 数组索引会报错。

1. **改进错误信息**

将被动触发不可控的报错信息改为主动调用的形式：

```rust
...
fn new(args: &[String]) -> Config {
    if args.len() < 3 {
        panic!("not enough arguments");
    }
    ...
}
```

2. **返回 `Result` 替代直接 `panic`**

返回一个 `Result`：成功时包含 `Config` 实例，失败时包含一条错误信息。

```rust
impl Config {
    fn build(args: &[String]) -> Result<Config, &'static str> {
        if args.len() < 3 {
            return Err("not enough arguments")
        }
		...
        Ok(Config { query, file_path })
    }
}
```

`'static`是Rust中的一个特殊生命周期，它表示一个生命周期是静态的，即它的值在程序的整个运行期间都存在。

3. **处理返回的 `Result`**

对返回的 `Result` 进行处理，给出准确且友好的报错提示。

```rust
use std::env;
use std::fs;
use std::process;

fn main() {
    ...
    // 对 build 返回的 Result 进行处理
    let config = Config::build(&args).unwrap_or_else(|err|{
        println!("Problem parsing arguments: {err}");
        process::exit(1);
    });
    ...
}
```

`unwrap_or_else` 是定义在 `Result<T, E>` 上的常用方法，如果 `Result` 是 `Ok` ，那么该方法就类似于 `unwrap`：返回 `Ok` 内部的值；如果是 `Err` ，就调用闭包中的中的自定义代码对错误进行进一步处理。

### 分离主体逻辑

1. **`main` 函数仅保留主流程各个环节的调用。**

```rust
fn main() {
    ...
    run(config);
}

fn run(config: Config) {
    let contents = fs::read_to_string(config.file_path)
        .expect("Should have been able to read the file.");
    println!("With text:\n{contents}");
}
...
```

2. **使用 `?` 和特征对象来返回错误**

 ```rust
 // in main.rs
 use std::error::Error;
 ...
 fn run(config: Config) -> Result<(), Box<dyn Error>> {
     let contents = fs::read_to_string(config.file_path)?;
     println!("With text:\n{contents}");
     Ok(())
 }
 ...
 ```

程序无需返回任何值，但是为了满足 `Result<T,E>` 的要求，因此使用了 `Ok(())` 返回一个单元类型 `()`。

3. **处理返回的错误**

```rust
...
fn main() {
    ...
    if let Err(e) = run(config) {
        println!("Application error: {e}");
        process::exit(1);
    } 
}
```

### 分离逻辑代码到库包

1. **创建 `src/lib.rs` 文件，将所有非 `main` 函数都移动到其中。**

```rust
// in lib.rs
use std::fs;
use std::error::Error;

pub struct Config {
    query: String,
    file_path: String,
}

pub fn run(config: Config) -> Result<(), Box<dyn Error>> {
	...
}

pub impl Config {
    ...
}
```

2. **在`main.rs` 中引入 `lib.rs` 中定义的 `Config` 类型**

```rust
...
use minigrep::Config;
...
```

