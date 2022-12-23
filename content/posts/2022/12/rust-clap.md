+++
title = "Clap 简单使用"
date = 2022-12-23T16:33:00+08:00
lastmod = 2022-12-23T19:57:31+08:00
tags = ["命令行解析"]
categories = ["Rust"]
draft = false
toc = true
+++

## 添加参数 {#添加参数}


### Basic {#basic}

以 `myapp run app --release --message hello` 为例 <br/>
这里的参数分为几种 <br/>

1.  `run` 子命令 <br/>
2.  `app` 固定位置参数 <br/>
3.  `--release` flag 参数 <br/>
4.  `--message hello` 命名选项参数 <br/>


#### 子命令 {#子命令}

```rust
use clap::{Parser, Subcommand};

#[derive(Parser)]
#[command(author, version, about, long_about = None)]
struct Cli {
    #[command(subcommand)]
    command: Commands
}

#[derive(Subcommand)]
enum Commands {
    Add {
	name: Option<String>
    }
}

fn main() {
    let cli = Cli::parse();
    match &cli.command {
	Commands::Add {name} => {
	    println!("myapp add was used, name is: {:?}", name)
	}
    }
}
```

1.  在结构中使用 `#[command(subcommand)]` 表示以下结构是一个子命令 <br/>
2.  用 `#[derive(Subcommand)]` 来声明一个子命令结构 <br/>
3.  调用的时候，使用 `myapp add hello` 即可 <br/>


#### 固定位置参数 {#固定位置参数}

```rust
use clap::Parser;

#[derive(Debug, Parser)]
struct Cli {
    a: u32,
    b: u32,
    c: u32,
    name: String
}

fn main() {
    let cli = Cli::parse();
    println!("{:?}", cli);
}
```

程序的解析顺序好像是按照结构体中字段的声明顺序决定的 <br/>
这个程序应该这样调用 <br/>

```bash
myapp 1 2 3 hello
```


#### 命名选项参数 {#命名选项参数}

使用这个的时候，需要在字段上添加一个 `#[arg(short, long)]` ，就可以生成短选项和长选项 <br/>

```rust
use clap::Parser;

#[derive(Debug, Parser)]
struct Cli {
    a: u32,
    b: u32,
    c: u32,
    #[arg(long)]
    name: String
}

fn main() {
    let cli = Cli::parse();
    println!("{:?}", cli);
}
```

这样调用程序 <br/>

```bash
myapp 1 2 3 --name hello
```


#### Flag 参数 {#flag-参数}

```rust
use clap::Parser;

#[derive(Debug, Parser)]
struct Cli {
    a: u32,
    b: u32,
    c: u32,
    #[arg(long)]
    name: String,

    #[arg(long)]
    verbose: bool
}

fn main() {
    let cli = Cli::parse();
    println!("{:?}", cli);
}
```

这个很简单，先添加 `#[arg(long)]` 或者 `#[arg(short)]` 宏，再将字段类型改为 `bool` 即可 <br/>
这样调用程序 <br/>

```bash
myapp 1 2 3 --name hello --verbose
```


#### 可选的参数 {#可选的参数}

这个也很简单，将字段类型改为 `Option` 即可，这样程序找不到匹配的项时，会将值设置为 `None` <br/>

```rust
use clap::Parser;

#[derive(Debug, Parser)]
struct Cli {
    a: u32,
    b: u32,
    c: Option<u32>,

    #[arg(long)]
    name: String,

    #[arg(long)]
    verbose: bool,
}

fn main() {
    let cli = Cli::parse();
    println!("{:?}", cli);
}
```

这样调用程序 <br/>

```bash
myapp 1 2 --name hello --verbose
```


### Advanced {#advanced}

更进一步的，我们可以对参数进行验证和添加默认值 <br/>


#### 默认值 {#默认值}

根据文档上的描述，只需要在字段上加上 `#[arg(default_value_t = ?)]` <br/>

```rust
use clap::Parser;

#[derive(Parser)]
struct Cli {
    #[arg(default_value_t = 2020)]
    port: u16
}

fn main() {
    let cli = Cli::parse();
    println!("port: {:?}", cli.port);
}
```


#### 验证 {#验证}

我们手动制定一个验证器函数，这个函数要返回一个 `Result` 类型，接受一个 `&str` 参数 <br/>

```rust
use std::ops::RangeInclusive;

const PORT_RANGE: RangeInclusive<usize> = 1..=65535;

fn port_in_range(s: &str) -> Result<u16, String> {
    let port: usize = s
	.parse()
	.map_err(|_| format!("`{}` isn't a port number", s))?;

    if PORT_RANGE.contains(&port) {
	Ok(port as u16)
    } else {
	Err(format!(
	    "Port not in range {} - {}",
	    PORT_RANGE.start(),
	    PORT_RANGE.end()
	))
    }
}
```

`Result` 的左值类型要与字段类型匹配 <br/>

```rust
#[derive(Debug)]
struct Cli {
    #[arg(value_parser = port_in_range)]
    port: u16
}

fn main() {
    let cli = Cli::parse();
    println!("port = {}", cli.port);
}
```

想必你也发现了，这个验证器不仅是个验证器，他也是个转换器，我们自己做个转换器，把输入数字的负数打印出来 <br/>

```rust
use clap::Parser;

#[derive(Debug, Parser)]
struct Cli {
    a: u32,
    b: u32,
    #[arg(value_parser = convert)]
    c: i32,

    #[arg(long)]
    name: String,

    #[arg(long)]
    verbose: bool,

}

fn convert(s: &str) -> Result<i32, String> {
    let number: i32 = s
	.parse()
	.map_err(|_| format!("`{}` isn't a valid number", s))?;

    return -number as i32;
}

fn main() {
    let cli = Cli::parse();
    println!("{:?}", cli);
}
```