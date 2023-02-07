+++
title = "SpringBoot with MongodbRepository"
date = 2023-01-05T00:19:00+08:00
lastmod = 2023-02-07T12:29:40+08:00
tags = ["Mongodb"]
categories = ["SpringBoot"]
draft = false
toc = true
+++

## Spring with MongodbRepository 简单使用 {#spring-with-mongodbrepository-简单使用}

这篇笔记参考自 [这篇文章](https://chikuwa-tech-study.blogspot.com/2021/05/spring-boot-mongorepository.html) <br/>


### 定义数据类型 {#定义数据类型}

```kotlin
class Product(
    val id: String,
    val name: String,
    val price: Int
)
```

{{< figure src="/ox-hugo/2023-01-05_16-41-37_screenshot.png" >}} <br/>

我们可以 <br/>

1.  手动定义对象存储的集合位置 <br/>
    ```kotlin
    @Document(collection = "products")
    class Product(
    
    )
    ```
2.  手动定义对象的主键 <br/>
    ```kotlin
    class Product(
        @MongoId
        val id: String
    )
    ```


### 定义 Repository {#定义-repository}

```kotlin
@Repository
interface ProductRepository: MongoRepository<Product, String> {
    fun findAllByNameLike(name: String): List<Product>
    fun findById(id: String): Product?
    fun existsByName(name: String): Bool
}
```

继承自 `MongoRepository` 的接口类与 `JPA` 相似，定义好方法即可， CRUD 方法也与之类似 <br/>


### 排序 {#排序}

```kotlin
@Repository
interface ProductRepository: MongoRepository<Product, String> {
    fun findAllByNameLike(name: String, sort: Sort): List<Product>
}
```

```kotlin
val sort = Sort.by(Sort.Direction.ASC, "price")
val sort = Sort.by(
    Sort.Order.asc("price"),
    Sort.Order.desc("name")
)
```


### 原生查询语法 {#原生查询语法}

```kotlin
@Query("{'name': {'$eq': '?0'}}")
fun f1(name: String): Product?

@Query("{'name': {\$regex: '?0'}}")
fun f2(name: String): Product?
```