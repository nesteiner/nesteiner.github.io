+++
title = "Rust Mongodb 简单使用"
date = 2023-01-01T12:28:00+08:00
lastmod = 2023-01-02T15:00:10+08:00
tags = ["Mongodb"]
categories = ["Rust"]
draft = false
toc = true
+++

## Rust Mongodb {#rust-mongodb}


### 开启同步和异步 {#开启同步和异步}

同步 <br/>

```toml
mongodb = {version = "2.3.1", features = ["tokio-sync"]}
```

```rust
use mongodb::sync::{Client, Collection};
```

异步 <br/>

```toml
mongodb = {version = "2.3.1"}
```

```rust
use mongodb::{Client, Collection};
```


### 连接 Mongodb {#连接-mongodb}

```rust
impl Client {
    pub fn with_uri_str(url: impl AsRef<str>) -> Result<Self>
}
```

```rust
impl Client {
    pub async fn with_uri_str(url: impl AsRef<str>) -> Result<Self>
}
```


### 连接到 database {#连接到-database}

```rust
impl Client {
    pub fn database(&self, name: &str) -> Database
}
```


### 连接到 collection {#连接到-collection}

```rust
impl Database {
    pub fn collection<T>(&self, name: &str) -> Collection<T>
}
```


### collection 操作 {#collection-操作}

异步和同步代码就差一个 `await` 操作，这里以同步代码为例 <br/>


#### find {#find}

<!--list-separator-->

-  方法定义

    ```rust
    impl <T> Collection<T> {
        pub fn find(
    	&self,
    	filter: impl Into<Option<Document>>,
    	options: impl Into<Option<FindOptions>>
        ) -> Result<Cursor<T>>
    }
    ```
    
    ```rust
    impl <T> Collection<T> {
        pub fn find_one(
    	&self,
    	filter: impl Into<Option<Document>>,
    	options: impl Into<Option<FindOneOptions>>
        )
    } -> Result<Option<T>>
    ```

<!--list-separator-->

-  [builder] FindOptions 与 FindOneOptions

    ```rust
    pub struct FindOptions {
        pub allow_disk_use: Option<bool>,
        pub allow_partial_results: Option<bool>,
        pub batch_size: Option<u32>,
        pub comment: Option<String>,
        pub cursor_type: Option<CursorType>,
        pub hint: Option<Hint>,
        pub limit: Option<i64>,
        pub max: Option<Document>,
        pub max_await_time: Option<Duration>,
        pub max_scan: Option<u64>,
        pub max_time: Option<Duration>,
        pub min: Option<Document>,
        pub no_cursor_timeout: Option<bool>,
        pub projection: Option<Document>,
        pub read_concern: Option<ReadConcern>,
        pub return_key: Option<bool>,
        pub selection_criteria: Option<SelectionCriteria>,
        pub show_record_id: Option<bool>,
        pub skip: Option<u64>,
        pub sort: Option<Document>,
        pub collation: Option<Collation>,
        pub let_vars: Option<Document>,
    }
    ```
    
    ```rust
    pub struct FindOneOptions {
        pub allow_partial_results: Option<bool>,
        pub collation: Option<Collation>,
        pub comment: Option<String>,
        pub hint: Option<Hint>,
        pub max: Option<Document>,
        pub max_scan: Option<u64>,
        pub max_time: Option<Duration>,
        pub min: Option<Document>,
        pub projection: Option<Document>,
        pub read_concern: Option<ReadConcern>,
        pub return_key: Option<bool>,
        pub selection_criteria: Option<SelectionCriteria>,
        pub show_record_id: Option<bool>,
        pub skip: Option<u64>,
        pub sort: Option<Document>,
        pub let_vars: Option<Document>,
    }
    ```

<!--list-separator-->

-  返回类型 Cursor

    <!--list-separator-->
    
    -  同步
    
        ```rust
        if let Ok(cursor) = collection.find(None, None) {
            for result in cursor {
        	println!("{:?}", result);
            }
        }
        ```
    
    <!--list-separator-->
    
    -  异步
    
        ```rust
        use futures::stream::{StreamExt, TryStreamExt};
        
        let mut cursor = collection.find(None, None).await?;
        while let Some(doc) = cursor.next().await {
            println!("{}", doc?);
        }
        
        let mut cursor = collection.find(None, None).await?;
        while let Some(doc) = cursor.try_next().await? {
            println!("{}", doc);
        }
        ```


#### insert {#insert}

<!--list-separator-->

-  方法定义

    ```rust
    impl <T> Collection<T> {
        pub fn insert_many(
    	&self,
    	docs: impl IntoIterator<Item = impl Borrow<T>>,
    	options: impl Into<Option<InsertManyOptions>>
        ) -> Result<InsertManyResult>
    }
    ```
    
    ```rust
    impl <T> Collection<T> {
        pub fn insert_one(
    	&self,
    	doc: impl Borrow<T>,
    	options: impl Into<Option<InsertOneOptions>>
        ) -> Result<InsertOneResult>
    }
    ```

<!--list-separator-->

-  [builder] InsertManyOptions 与 InsertOneOptions

    ```rust
    pub struct InsertManyOptions {
        pub bypass_document_validation: Option<bool>,
        pub ordered: Option<bool>,
        pub write_concern: Option<WriteConcern>,
    }
    ```
    
    ```rust
    pub struct InsertOneOptions {
        pub bypass_document_validation: Option<bool>,
        pub write_concern: Option<WriteConcern>,
    }
    ```

<!--list-separator-->

-  返回类型 InsertManyResult 与 InsertOneResult

    ```rust
    pub struct InsertManyResult {
        pub inserted_ids: HashMap<usize, Bson>,
    }
    ```
    
    ```rust
    pub struct InsertOneResult {
        pub inserted_id: Bson,
    }
    ```


#### delete {#delete}

<!--list-separator-->

-  方法定义

    ```rust
    impl <T> Collection<T> {
        pub fn delete_many(
    	&self,
    	query: Document,
    	options: impl Into<Option<DeleteOptions>>
        ) -> Result<DeleteResult>
    }
    ```
    
    ```rust
    impl <T> Collection<T> {
        pub fn delete_one(
    	&self,
    	query: Document,
    	options: impl Into<Option<DeleteOptions>>
        ) -> Result<DeleteResult>
    }
    ```

<!--list-separator-->

-  [builder] DeleteOptions

    ```rust
    pub struct DeleteOptions {
        pub collation: Option<Collation>,
        pub write_concern: Option<WriteConcern>,
        pub hint: Option<Hint>,
        pub let_vars: Option<Document>,
    }
    ```

<!--list-separator-->

-  返回类型 DeleteResult

    ```rust
    pub struct DeleteResult {
        pub deleted_count: u64,
    }
    ```


#### update {#update}

<!--list-separator-->

-  方法定义

    ```rust
    impl <T> Collection<T> {
        pub fn update_many(
    	&self,
    	query: Document,
    	update: impl Into<UpdateModifications>,
    	options: impl Into<Option<UpdateOptions>>
        ) -> Result<UpdateResult>
    }
    ```
    
    ```rust
    impl <T> Collection<T> {
        pub fn update_one(
    	&self,
    	query: Document,
    	update: impl Into<UpdateModifications>,
    	options: impl Into<Option<UpdateOptions>>
        ) -> Result<UpdateResult>
    }
    ```

<!--list-separator-->

-  [builder] UpdateModifications

    Both **Document** and **Vec&lt;Document&gt;** implement Into&lt;UpdateModifications&gt;, so either can be passed in place of constructing the enum case <br/>

<!--list-separator-->

-  返回类型 UpdateResult

    ```rust
    pub struct UpdateResult {
        pub matched_count: u64,
        pub modified_count: u64,
        pub upserted_id: Option<Bson>,
    }
    ```


#### replace {#replace}

<!--list-separator-->

-  方法定义

    ```rust
    impl <T> Collection<T> {
        pub fn replace_one(
    	&self,
    	query: Document,
    	replacement: impl Borrow<T>,
    	options: impl Into<Option<ReplaceOptioins>>
        ) -> Result<UpdateResult>
    }
    ```

<!--list-separator-->

-  [builder] ReplaceOptions

    ```rust
    pub struct ReplaceOptions {
        pub bypass_document_validation: Option<bool>,
        pub upsert: Option<bool>,
        pub collation: Option<Collation>,
        pub hint: Option<Hint>,
        pub write_concern: Option<WriteConcern>,
        pub let_vars: Option<Document>,
    }
    ```


### Document 类型 {#document-类型}

**Document** 类型使用 `doc!` 宏来构造，写的时候参考 `json` 就行了 <br/>

```rust
let doc = doc! {"title": {"$set": "hello"}};
let doc = doc! {"age": {"$eq": 1}};
```