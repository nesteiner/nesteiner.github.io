+++
title = "SprintBoot with Redis 简单使用"
date = 2023-01-05T17:28:00+08:00
lastmod = 2023-02-07T12:29:55+08:00
tags = ["Redis"]
categories = ["SpringBoot"]
draft = false
toc = true
+++

## SprintBoot with Redis 简单使用 {#sprintboot-with-redis-简单使用}


### 定义 redisTemplate {#定义-redistemplate}

```kotlin
@Configuration
class RedisConfigure {
    @Bean
    fun redisTemplate(connectionFactory: LettuceConnectionFactory): RedisTemplate<String, User> {
	val redisTemplate = RedisTemplate<String, User>();
	redisTemplate.keySerializer = StringRedisSerializer()
	redisTemplate.valueSerializer = GenericJackson2JsonRedisSerializer()
	redisTemplate.setConnectionFactory(connectionFactory)
	return redisTemplate
    }
}
```


### 定义数据类型 {#定义数据类型}

```kotlin
class User(
    val id: Long,
    val name: String,
    val sex: String,
) {
    constructor() : this(0L, "", "man")
}
```


### 操作 {#操作}

```kotlin
@SpringBootTest
class SpringbootRedisApplicationTests {
    @Autowired
    lateinit var redisTemplate: RedisTemplate<String, User>

    @Test
    fun testSerializable() {
	val user = User(1L, "hello", "man")
	val ops = redisTemplate.opsForValue()
	val commands = redisTemplate.connectionFactory?.connection?.serverCommands()
	commands?.flushDb()
	ops.set("user", user)
	val otherUser = ops.get("user")
	println(otherUser)
    }

    @Test
    fun testOtherOperations() {
	val ops = redisTemplate.opsForList();
	ops.leftPush("users", User(1L, "hello", "man"))
	ops.leftPush("users", User(2L, "world", "female"))
    }
}
```