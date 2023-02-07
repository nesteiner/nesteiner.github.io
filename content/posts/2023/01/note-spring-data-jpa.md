+++
title = "[Note] Spring Data JPA 笔记"
date = 2023-01-23T19:30:00+08:00
lastmod = 2023-02-07T12:29:12+08:00
tags = ["数据库", "笔记", "Mindmap"]
categories = ["Spring"]
draft = false
toc = true
+++

{{< figure src="/ox-hugo/2023-01-24_16-05-35_screenshot.png" >}} <br/>


## 引入依赖 {#引入依赖}


## 简单查询 {#简单查询}


### 预生成方法 {#预生成方法}


### 自定义简单查询 {#自定义简单查询}


#### 属性查询 {#属性查询}


#### 组合查询 {#组合查询}


### 自定义SQL查询 {#自定义sql查询}


#### HQL {#hql}


#### SQL {#sql}


#### 修改和删除 {#修改和删除}

1.  @Modifying <br/>
2.  @Transactional <br/>


### <span class="org-todo done CANCEL">CANCEL</span> 已命名查询 {#已命名查询}


#### 定义命名查询 {#定义命名查询}


#### 调用命名查询 {#调用命名查询}


#### 运行验证 {#运行验证}


## 复杂查询 {#复杂查询}


### 分页查询 {#分页查询}

pageable <br/>


### 排序和限制 {#排序和限制}


### 动态条件查询 {#动态条件查询}

```kotlin
@Test
fun testSpecification() {
    userRepository.deleteAll()

    val users = listOf<User>(
	User(null, "hello", 1),
	User(null, "world", 2),
	User(null, "holy", 3),
	User(null, "shit", 4)
    )

    userRepository.saveAll(users)

    val matchUsers = userRepository.findAll { root, query, criteriaBuilder ->
	  val p1 = criteriaBuilder.like(root.get("name"), "w%")
	  val p2 = criteriaBuilder.greaterThan(root.get("age"), 0)
	  criteriaBuilder.and(p1, p2)
    }

    matchUsers.forEach { println(it.name) }
}
```


## 实体关系映射 {#实体关系映射}


### 关系映射注解 {#关系映射注解}


#### 实体类注解 {#实体类注解}

1.  @JoinColumn <br/>
2.  @JoinTable <br/>


#### 关系映射注解 {#关系映射注解}

1.  @OneToOne <br/>
2.  @OneToMany <br/>
    mappedBy = "variable-name" <br/>
3.  @ManyToOne <br/>
4.  @ManyToMany <br/>

如何构造映射类型， <br/>
猜想: 先默认，再lazy? <br/>


### 一对一 {#一对一}


### 一对多和多对一 {#一对多和多对一}


### 多对多 {#多对多}

```kotlin
@ManyToMany(fetch = FetchType.EAGER)
@JoinTable(
    name = "StudentRole",
    joinColumns = [JoinColumn(name = "userid", referencedColumnName = "id")],
    inverseJoinColumns = [JoinColumn(name = "roleid", referencedColumnName = "id")]
)
val roles: List<Role>
```

@JoinTable 中间表的描述 <br/>

```kotlin
@Entity(name = "Permissions")
class Permission(
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    val id: Long?,
    @Column(nullable = false)
    val name: String,
    @Column(nullable = false)
    val type: String,
    @Column(nullable = false)
    val url: String,
    @Column(nullable = false)
    val code: String,

    @ManyToMany(mappedBy = "permission", fetch = FetchType.LAZY)
    val roles: Set<Role>
)

@Entity(name = "Roles")
class Role(
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    val id: Long?,
    @Column(nullable = false)
    val name: String,
    @OneToMany(mappedBy = "role", fetch = FetchType.LAZY, cascade = [CascadeType.ALL])
    val users: Set<User>,

    @ManyToMany(cascade = [CascadeType.MERGE], fetch = FetchType.LAZY)
    @JoinTable(name = "Role_Permission")
    val permissions: Set<Permission>
)
```