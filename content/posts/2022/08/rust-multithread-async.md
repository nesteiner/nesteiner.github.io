+++
title = "Rust 多线程程序和异步程序简单编写"
date = 2022-08-18T19:23:00+08:00
lastmod = 2022-08-18T20:39:50+08:00
tags = ["多线程", "异步"]
categories = ["Rust"]
draft = false
toc = true
+++

## 程序介绍 {#程序介绍}

这次我打算把一个 **select** 多路复用的程序改成多线程版本和异步版本的，他的工作流程如下 <br/>

-   从 `stdin` 读取数据，处理数据 <br/>
-   从 `socket` 读取数据，处理数据 <br/>

而多线程和异步版本的工作流程则是 <br/>

-   创建数据通道 `channel` <br/>
-   创建一个 线程/Future, 从 `stdin` 读取数据，将数据写入 `channel` <br/>
-   创建一个 线程/Future, 从 `socket` 读取数据，将数据写入 `channel` <br/>
-   创建一个 线程/Future, 从 `channel` 读取数据，处理数据 <br/>

我们马上来做一下看吧 <br/>


## 多线程程序 {#多线程程序}


### 从 socket 读取数据的线程 {#从-socket-读取数据的线程}

```rust
let thread1 = thread::spawn(move || {
    loop {
	let (mut stream, _) = server.accept().unwrap();
	let mut received = String::new();

	if let Err(_) = stream.read_to_string(&mut received) {
	    eprintln!("recv error occur");
	}

	if let Err(e) = sender1.send(received) {
	    eprintln!("send error occur: {}", e);
	}
    }
});

```


### 从 stdin 读取数据的线程 {#从-stdin-读取数据的线程}

```rust
let thread2 = thread::spawn(move || {
    loop {
	let mut input = io::stdin().lock().lines().next().unwrap().unwrap();
	if input == "quit" {
	    process::exit(0);
	}

	if let Err(e) = sender2.send(input) {
	    eprintln!("send error occur: {}", e);
	}
    }
});

```


### 从 channel 读取数据的线程 {#从-channel-读取数据的线程}

```rust
let manager_thread = thread::spawn(move || {
    loop {
	match receiver.recv() {
	    Ok(received) => {
		process(received);
	    },

	    Err(e) => {
		eprintln!("{}", e);
	    }
	}
    }
});
```


### 完整代码 {#完整代码}

```rust
use std::net::TcpListener;
use std::sync::{Arc, mpsc::{self, Sender, Receiver}};
use std::thread;
use std::io;
use std::io::{BufRead, Read};
use std::process;

fn main() {
    let server = TcpListener::bind("127.0.0.1:9999").unwrap();
    let (sender, receiver) = mpsc::channel::<String>();
    let sender1 = sender.clone();
    let sender2 = sender.clone();
    // recv from client
    let thread1 = thread::spawn(move || {
	loop {
	    let (mut stream, _) = server.accept().unwrap();
	    let mut received = String::new();
	    let _ = stream.read_to_string(&mut received);
	    if let Err(e) = sender1.send(received) {
		println!("send error: {}", e);
	    }
	}
    });


    // recv from stdin
    let thread2 = thread::spawn(move || {
	loop {
	    let mut input = io::stdin().lock().lines().next().unwrap().unwrap();
	    if input == "quit" {
		process::exit(0);
	    }

	    if let Err(e) = sender2.send(input) {
		eprintln!("send error: {}", e);
	    }
	}
    });

    let manager_thread = thread::spawn(move || {
	loop {
	    match receiver.recv() {
		Ok(received) => {
		    process(received);
		},

		Err(e) => {
		    eprintln!("{}", e);
		}
	    }
	}
    });

    thread1.join().unwrap();
    thread2.join().unwrap();
    manager_thread.join().unwrap();
}

fn process(string: String) {
    println!("recv {} bytes from peer end: {}", string.len(), string);
}
```


## 异步程序 {#异步程序}


### 注意 {#注意}

没事不要作死写 stdin + async 的程序，会出现莫名其妙的阻塞，就像我的代码 <br/>


### 从 socket 读取数据的 task {#从-socket-读取数据的-task}

```rust
let task1 = tokio::spawn(async move {
    loop {
	let (mut stream, _) = server.accept().unwrap();
	let mut received = String::new();

	if let Err(_) = stream.read_to_string(&mut received) {
	    eprintln!("recv error occur");
	    break;
	}

	if let Err(e) = sender1.send(received).await {
	    eprintln!("send error occur: {}", e);
	    break;
	}
    }
});
```


### 从 stdin 读取数据的 task {#从-stdin-读取数据的-task}

```rust
let task2 = tokio::spawn(async move {
    loop {
	let input = io::stdin().lock().lines().next().unwrap().unwrap();

	if input.trim() == "quit" {
	    process::exit(0);
	}

	println!("before send2 send");

	if let Err(e) = sender2.send_timeout(input, Duration::from_secs(10)).await {
	    eprintln!("send error occur: {}", e);
	    // sender2.send(String::from("quit"));
	    process::exit(0);
	}

	println!("after send2 send");
    }
});
```


### 从 channel 读取数据的 task {#从-channel-读取数据的-task}

```rust
let manager_task = tokio::spawn(async move {
    loop {
	if let Some(received) = receiver.recv().await {
	    process(received);
	}
    }
});

```


### 完整代码 {#完整代码}

```rust
use std::{process};
use std::io::{self, BufRead, Read};
use std::net::{TcpListener};
use std::time::Duration;
use tokio::sync::mpsc;
use tokio::sync::mpsc::Sender;

#[tokio::main]
async fn main() {
    let server = TcpListener::bind("127.0.0.1:9999").unwrap();
    let (sender, mut receiver) = mpsc::channel::<String>(1);
    let sender1 = sender.clone();
    let sender2 = sender.clone();

    let task1 = tokio::spawn(async move {
	loop {
	    let (mut stream, _) = server.accept().unwrap();
	    let mut received = String::new();

	    if let Err(_) = stream.read_to_string(&mut received) {
		eprintln!("recv error occur");
		break;
	    }

	    if let Err(e) = sender1.send(received).await {
		eprintln!("send error occur: {}", e);
		break;
	    }
	}
    });

    let task2 = tokio::spawn(async move {
	loop {
	    let input = io::stdin().lock().lines().next().unwrap().unwrap();

	    if input.trim() == "quit" {
		process::exit(0);
	    }

	    println!("before send2 send");

	    if let Err(e) = sender2.send_timeout(input, Duration::from_secs(10)).await {
		eprintln!("send error occur: {}", e);
		// sender2.send(String::from("quit"));
		process::exit(0);
	    }

	    println!("after send2 send");
	}
    });

    let manager_task = tokio::spawn(async move {
	loop {
	    if let Some(received) = receiver.recv().await {
		process(received);
	    }
	}
    });

    task1.await.unwrap();
    task2.await.unwrap();
    manager_task.await.unwrap();
}

fn process(string: String) {
    println!("read {} from peer: {}", string.len(), string);
}
```