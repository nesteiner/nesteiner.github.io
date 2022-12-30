+++
title = "Serde Json 简单使用"
date = 2022-12-30T17:59:00+08:00
lastmod = 2022-12-30T18:44:34+08:00
tags = ["序列化"]
categories = ["Rust"]
draft = false
toc = true
+++

## Serde Json {#serde-json}


### 添加依赖 {#添加依赖}

```bash
cargo add serde --features derive
cargo add serde_json
cargo add serde_derive
```


### 结构体的序列化与反序列化 {#结构体的序列化与反序列化}

```rust
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize, Debug)]
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let point = Point {x: 1, y: 2};
    let json: String = serde_json::to_string(&point).unwrap();
    println!("{}", json);

    let point: Point = serde_json::from_str(&json).unwrap();
    println!("{:#?}", point);
}
```


### 枚举的序列化与反序列化 {#枚举的序列化与反序列化}


#### 第一种枚举类型 {#第一种枚举类型}

参照结构体的序列化和反序列化，如果对一个枚举进行序列化，我们发现 <br/>

```rust
use serde::{Serialize, Deserialize};

#[derive(Debug, Serialize, Deserialize)]
enum Week {
    Monday,
    Tuesday,
    Wednesday,
    Thursday,
    Friday,
    Saturday,
    Sunday,
}

fn main() {
    let json: String = serde_json::to_string(&Week::Friday).unwrap();
    println!("{}", json);

    let week: Week = serde_json::from_str(&json).unwrap();
    println!("{:#?}", week);
}
```

运行结果如下 <br/>

```bash
"Friday"
Friday
```

这个不是我们想的JSON格式，我们可以添加一个 **serde(tag)** 宏 <br/>

```rust
use serde::{Serialize, Deserialize};

#[derive(Debug, Serialize, Deserialize)]
#[serde(tag = "WeekDay")]
enum Week {
    Monday,
    Tuesday,
    Wednesday,
    Thursday,
    Friday,
    Saturday,
    Sunday,
}

fn main() {
    let json: String = serde_json::to_string(&Week::Friday).unwrap();
    println!("{}", json);

    let week: Week = serde_json::from_str(&json).unwrap();
    println!("{:#?}", week);
}
```

运行结果如下 <br/>

```bash
{
    "WeekDay": "Friday"
}

Friday
```

哟系，结果不错 <br/>


#### 第二种枚举类型 {#第二种枚举类型}

```rust
use serde::{Serialize, Deserialize};

#[derive(Debug, Serialize, Deserialize)]
enum IP {
    IPv4(String),
    IPv6(String)
}

fn main() {
    let json: String = serde_json::to_string(&IP::IPv4("127.0.0.1".to_string())).unwrap();
    println!("{}", json);

    let ip: IP = serde_json::from_str(&json).unwrap();
    println!("{:#?}", ip);
}
```

运行结果如下 <br/>

```bash
{
    "IPv4": "110.26.73.83"
}
IPv4(
    "110.26.73.83",
)
```

我们可以添加一些操作，让其结构变的更合理一些 <br/>

```rust
use serde::{Serialize, Deserialize};

#[derive(Debug, Serialize, Deserialize)]
#[serde(tag = "type", content = "ip")]
enum IP {
    IPv4(String),
    IPv6(String)
}

fn main() {
    let json: String = serde_json::to_string(&IP::IPv4("127.0.0.1".to_string())).unwrap();
    println!("{}", json);

    let ip: IP = serde_json::from_str(&json).unwrap();
    println!("{:#?}", ip);
}
```

运行结果如下 <br/>

```bash
{
    "type": "IPv6",
    "ip": "::ffff:110.26.73.83"
}
IPv6(
    "::ffff:110.26.73.83",
)
```


### Unit Struct 的序列化与反序列化 {#unit-struct-的序列化与反序列化}

```rust
use serde::{Serialize, Deserialize};

#[derive(Debug, Serialize, Deserialize)]
struct UnitStruct;

fn main() {
    let json: String = serde_json::to_string(&UnitStruct).unwrap();
    println!("{}", json);

    let n: UnitStruct = serde_json::from_str(&json).unwrap();
    println!("{:#?}", n);
}
```

运行结果如下 <br/>

```bash
null
UnitStruct
```


### 一些选项 {#一些选项}


#### 忽略某个字段 {#忽略某个字段}

1.  **#[serde(skip_serialize)]** 在序列化时忽略该字段 <br/>
2.  **#[serde(skip_deserialize)]** 在反序列化时忽略该字段 <br/>
3.  **#[serde(skip)]** 同时忽略这个字段 <br/>

可是这样以后，结构体在反序列化时会造成运行时错误 <br/>

```rust
#[macro_use]
extern crate serde_derive;

use serde::{Serialize, Deserialize};

#[derive(Debug, Serialize, Deserialize)]
struct Point {
    x: f64,
    #[serde(skip_serializing)]
    y: f64
}

fn main() {
    let point = Point {
	x: 1.0,
	y: 2.0
    };

    let json: String = serde_json::to_string(&point).unwrap(); // this is ok
    println!("{}", json);

    let point: Point = serde_json::from_str(&point).unwrap(); // this is not ok
    println("{:#?}", point);
}
```


#### 提供默认值 {#提供默认值}

在上面的例子中，我们可以通过提供默认值来解决错误 <br/>

```rust
#[derive(Debug, Serialize, Deserialize)]
struct Point {
    x: f64,
    #[serde(skip_serializing, default)]
    y: f64
}
```

也可以手动设置提供值的函数 <br/>

```rust
#[derive(Debug, Serialize, Deserialize)]
struct Point {
    x: f64,
    #[serde(skip_serializing, default="default_y")]
    y: f64
}

fn default_y() -> f64 {
    5.0
}
```


#### 重命名字段 {#重命名字段}

```rust
#[derive(Debug, Serialize, Deserialize)]
struct Point {
    #[serde(rename = "X")]
    x: f64,
    #[serde(rename = "Y")]
    y: f64,
}
```