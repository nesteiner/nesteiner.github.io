+++
title = "Android Room 简单使用"
date = 2022-07-02T14:38:00+08:00
lastmod = 2022-07-02T14:47:34+08:00
tags = ["Room"]
categories = ["Android"]
draft = false
toc = true
+++

## Android Room 简单使用 {#android-room-简单使用}


### 说明 {#说明}

这次没有用内置的 `SQLite` 库，而是直接使用了 `Room` ，原因之一是 `SQLite` 在创建数据库的时候出错了，排查不到哪一步出错 <br/>
二是这个 `Room` 与 **Flutter** 中的 `floor` 框架非常相似，可以无缝迁移 <br/>


### 添加依赖 {#添加依赖}

在 `src/build.gradle` 中 <br/>

```groovy
dependencies {
    def room_version = "2.4.2"
    implementation "androidx.room:room-runtime:$room_version"
    annotationProcessor "androidx.room:room-compiler:$room_version"
}
```


### 配置类 {#配置类}

`Room` 包含三个主要组件 <br/>

-   数据库类 <br/>
-   数据实体 <br/>
-   数据访问对象(DAO) <br/>

他们的定义如下 <br/>


#### 数据实体 {#数据实体}

```java
@Entity
public class User {
    @PrimaryKey
    public int uid;

    @ColumnInfo(name = "first_name")
    public String firstName;

    @ColumnInfo(name = "last_name")
    public String lastName;
}
```


#### 数据访问对象 {#数据访问对象}

```java
@Dao
public interface UserDao {
    @Query("SELECT * FROM user")
    List<User> getAll();

    @Query("SELECT * FROM user WHERE uid IN (:userIds)")
    List<User> loadAllByIds(int[] userIds);

    @Query("SELECT * FROM user WHERE first_name LIKE :first AND " +
	   "last_name LIKE :last LIMIT 1")
    User findByName(String first, String last);

    @Insert
    void insertAll(User... users);

    @Delete
    void delete(User user);
}
```

注意 <br/>

-   他只需要一个接口来表示 `DAO` 类型，代码会自动生成，不需要担心 <br/>
-   变量名替换用冒号 `:`  <br/>


#### 数据库类 {#数据库类}

```java
@Database(entities = {User.class}, version = 1)
public abstract class AppDatabase extends RoomDatabase {
    public abstract UserDao userDao();
}
```


### 投入使用 {#投入使用}

这里不使用官方教程提供的实例，而是 <br/>

```java
AppDatabase database = Room.databaseBuilder(
		getApplicationContext(),
		AppDatabase.class,
		"test.db")
		.allowMainThreadQueries().build();

UserDao userDao = database.userDao();
```

不知道为什么，直接使用 `build` 方法会产生错误，需要添加一个 `allowMainThreadQueries`  <br/>
另外我们添加了一个 `Button` 和 `TextView` 来实现一些交互，简单来说，按钮来添加数据， `TextView` 来显示数据 <br/>

```java
Button button = findViewById(R.id.button_add);
TextView textView = findViewById(R.id.textView);
button.setOnClickListener(view -> {
	userDao.insertAll(
			  new User(1, "hello", "world"),
			  new User(2, "fuck", "you"),
			  new User(3, "holy", "shit")
			  );

	List<User> users = userDao.getAll();
	textView.setText(users.stream().map(x -> x.toString()).collect(Collectors.joining("\n")));
    });

```

别忘了给 `User` 添加构造函数 <br/>

现在问题来了，第一次点击按钮是没有问题的，第二次点击就会出错 <br/>
这是因为数据库主键重复了，我们重新设置下数据模型类 <br/>

```java
@Entity
public class User {
    @PrimaryKey(autoGenerate = true)
    public Integer uid;

    @ColumnInfo(name = "first_name")
    public String firstName;

    @ColumnInfo(name = "last_name")
    public String lastName;
}
```

像 `floor` 框架一样，将主键的值设为 `null` ，程序会在数据库中自动添加主键，这样以后修改关于添加的代码 <br/>

```java
button.setOnClickListener(view -> {
	userDao.insertAll(
			  new User("hello", "world"),
			  new User("fuck", "you"),
			  new User("holy", "shit")
			  );

	List<User> users = userDao.getAll();
	textView.setText(users.stream().map(x -> x.toString()).collect(Collectors.joining("\n")));
    });
```

再次运行你会发现 **又出错了** ，错误有关数据库迁移 `Migration` <br/>
这时因为原来的数据表有更改，需要修改原来的数据来兼容现有的数据库版本，这里我们简单处理，直接删掉重建一个 <br/>
在创建数据库时 <br/>

```java
AppDatabase database = Room.databaseBuilder(
	getApplicationContext(),
	AppDatabase.class,
	"test.db")
	.fallbackToDestructiveMigration()
	.allowMainThreadQueries().build();

```