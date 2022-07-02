+++
title = "在 Flutter 中使用数据库"
date = 2022-07-02T01:44:00+08:00
lastmod = 2022-07-02T14:43:45+08:00
tags = ["Flutter数据库"]
categories = ["Flutter"]
draft = false
toc = true
+++

## 在 Flutter 中使用数据库 {#在-flutter-中使用数据库}


### 介绍 {#介绍}

以 sqlite 为例，flutter 在使用数据库时需要引入 sqflite 依赖 <br/>

```yaml
dependencies:
  flutter:
    sdk: flutter

  sqflite:
```

导入 `package:sqflite/sqflite.dart` 与 `package:sqflite/sql.dart` 后，可以在 openDatabase 中的 onCreate 参数中设置初始化 <br/>

```dart
openDatabase(
  'file:///home/steiner/workspace/playground/todolist/todolist.db',
  onCreate: (database, version) async {
    await database.execute(
      'create table if not exists TaskList('
      'id integer primary key autoincrement,'
      'name text not null'
      ');'
    );
  },

  version: 1
);


```

其余数据库操作参考 [中文文档](https://flutter.cn/docs/cookbook/persistence/sqlite) <br/>


### 组件中使用数据库 {#组件中使用数据库}

由于在 `Dart` 中数据库的操作是异步的，返回值是 `Future` 类型，对 `Future` 使用 `await` 需要在异步函数中进行， <br/>
各个组件的 `build` 方法又是同步的，无法使用 `Future` <br/>
不过 `flutter` 提供了 `FutureBuilder` 组件，为其提供 `futurue` 选项来构造组件 <br/>

```dart
FutureBuilder({
  this.future,
  this.initialData,
  required this.builder,
})
```

这里我们只需要用 `future` 与 `builder` 就好了 <br/>
其中 <br/>

-   `future` : `FutureBuilder` 依赖的 `Future` ，通常是一个异步耗时任务。 <br/>
-   `initialData` : 初始数据，用户设置默认数据。 <br/>
-   `builder` : `Widget` 构建器; 该构建器会在 `Future` 执行的不同阶段被多次调用，构建器签名如下: <br/>
    `Function (BuildContext context, AsyncSnapshot snapshot)` <br/>

组件由 `builder` 返回，所有数据的获取通过 `snapshot.data` ，由于是异步操作，可能会有错误结果， <span class="underline">需要多次调用 future</span> <br/>
此时可以通过 `snapshot` 的一些属性来判断状态 <br/>

-   `snapshot.hasError` <br/>
-   `snapshot.hasData` <br/>

要查看错误信息，调用 `snapshot.error` <br/>

来看一个例子 <br/>

-   定义一个异步函数，返回数据库对象的 Future 类型 <br/>
    ```dart
    Future<Database> loadDataBase() async {
      WidgetsFlutterBinding.ensureInitialized();
      return openDatabase(
        'file:///home/steiner/workspace/playground/todolist/todolist.db',
        onCreate: (database, version) async {
          await database.execute(
    	'create table if not exists TaskList('
    	    'id integer primary key autoincrement,'
    	    'name text not null'
    	    ');'
          );
    
          List<TaskList> listOfTaskList = [
    	TaskList(name: 'Hello', id: 0),
    	TaskList(name: 'World', id: 0),
    	TaskList(name: 'Fuck', id: 0),
    	TaskList(name: 'You', id: 0),
          ];
    
          listOfTaskList.forEach((tasklist) async {
    	await database.rawInsert(
    	  'insert into ${TaskList.TABLE}'
    	      '(name)'
    	      'values(?);',
    	  [tasklist.name]
    	);
          });
    
          await database.execute(
    	'create table ${Task.TABLE} ('
    	    'id integer primary key autoincrement,'
    	    'name text,'
    	    'listid integer,'
    	    'isdone boolean,'
    	    'foreign key(listid) references ${TaskList.TABLE} (id)'
    	    ');'
          );
    
          List<Task> listOfTask = [
    	Task(id: 0, name: "task1", isdone: false, listid: 1),
    	Task(id: 0, name: "task2", isdone: false, listid: 1),
    	Task(id: 0, name: "task3", isdone: false, listid: 1),
    	Task(id: 0, name: "task4", isdone: false, listid: 2),
    	Task(id: 0, name: "task5", isdone: false, listid: 2),
          ];
    
          listOfTask.forEach((task) async {
    	await database.rawInsert(
    	  'insert into ${Task.TABLE}'
    	      '(name, isdone, listid)'
    	      'values(?, ?, ?);',
    	  [task.name, task.isdone, task.listid]
    	);
          });
        },
    
        version: 1
      );
    }
    ```

-   在 `HomePage` 组件中定义异步函数 `loadTaskList` ，返回 `List<TaskList>` 类型 <br/>
-   使用 `FutureBuilder` ，传入 `future` <br/>
-   在 `builder` 中返回组件 <br/>

<!--listend-->

```dart
class HomePage extends StatelessWidget {

  Future<List<TaskList>> loadTaskList() async {
    final database = await loadDataBase();
    final maps =  await database.query(TaskList.TABLE);

    return List.generate(maps.length, (index) {
	Map<String, dynamic> record = maps[index];
	return TaskList(name: record['name'], id: record['id']);
    });
  }

  Widget build(BuildContext context) {
    // TODO: implement build
    return Scaffold(
      appBar: AppBar(title: Text('HomePage')),
      body: FutureBuilder(
	future: loadTaskList(),
	builder: (BuildContext context, AsyncSnapshot<List<TaskList>> snapshot) {
	  if(snapshot.hasError) {
	    return Text("fuck, error: ${snapshot.error}");
	  } else if(snapshot.hasData) {
	    List<TaskList> listOfTaskList = snapshot.data!;
	    return Column(
	      children: listOfTaskList.map((tasklist) => buildTaskList(context, tasklist)).toList(),
	    );
	  } else {
	    return Text("there is no data now");
	  }
	},
      ),
    );
  }

  Widget buildTaskList(BuildContext context, TaskList tasklist) {
    return OutlineButton(
      onPressed: () {
	Navigator.push(context, MaterialPageRoute(
	    builder: (context) => TaskPage(tasklist: tasklist)
	));
      },

      child: Row(
	mainAxisAlignment: MainAxisAlignment.spaceBetween,
	children: [
	  Text(tasklist.name),
	  Text(tasklist.id.toString()),
	],
      ),
    );
  }
}


```


### 使用 ORM 框架 {#使用-orm-框架}

在一个测试的目录下，有以下文件 <br/>

-   `database.dart` <br/>
-   `database.g.dart` <br/>
-   `main.dart` <br/>
-   `task.dart` <br/>
-   `task_dao.dart` <br/>


#### 准备工作 {#准备工作}

在 `pubspec.yaml` 中需要导入几个依赖 <br/>

-   `floor` <br/>
-   `builder_runner` <br/>
-   `floor_generator` <br/>

其中最重要的是 `floor_generator` ，没有他后面的代码生成不会成功 <br/>


#### 实体类的定义 `task.dart` {#实体类的定义-task-dot-dart}

需要为实体类重载两个方法 <br/>

-   `operator ==` <br/>
-   `get hashCode` <br/>

另外 `toString()` 可选 <br/>

```dart
import 'package:floor/floor.dart';

@entity
class Task {
  @PrimaryKey(autoGenerate: true)
  int? id;
  final String message;

  Task({
      this.id,
      required this.message,
  })

  @override
  bool operator ==(Object other) =>
  identical(this, other) ||
  other is Task &&
  runtimeType == other.runtimeType &&
  id == other.id &&
  message == other.message;

  @override
  int get hashCode => id.hashCode ^ message.hashCode;

  @override
  String toString() {
    // TODO: implement toString
    return 'Task{id: $id, message: $message}';
  }

}
```

在代码中有 <br/>

-   `@entity` 声明这个类是实体类 <br/>
-   `@PrimaryKey` 声明主键 <br/>
-   `bool operator ==` 重载 <br/>
-   `int get hashCode` 重载 <br/>

其中 `@PrimaryKey(autoGenerate = true)` 表示这个主键是自增序列， <br/>
在构造函数中，主键 `id` 被定义为可以为空，这样不用传入 `id` , `floor` 会自动帮我们补上，按照自增顺序定义 `id` <br/>


#### DAO 的定义 `task_dao.dart` {#dao-的定义-task-dao-dot-dart}

`task_dao` 可以看作对表 `Task` 的操作接口 <br/>

```dart
@dao
abstract class TaskDao {
  @Query('select * from task where id = :id')
  Future<Task?> findTaskById(int id) ;

  @Query('select * from task')
  Future<List<Task>> findAllTask();

  @Query('select * from task')
  Stream<List<Task>> findAllTasksAsStream();

  @insert
  Future<void> insertTask(Task task);

  @insert
  Future<void> insertTasks(List<Task> tasks);

  @update
  Future<void> updateTask(Task task);

  @update
  Future<void> updateTasks(List<Task> tasks);

  @delete
  Future<void> deleteTask(Task task);

  @delete
  Future<void> deleteTasks(List<Task> tasks);
}
```

在代码中，有 <br/>

-   `abstract class` 抽象类 <br/>
-   `@dao` 声明类是一个 Data Access Object <br/>
-   `@Query` 通过此函数来查询，传入查询语句表示函数的行为 <br/>
-   `@insert` 通过此函数来插入数据 <br/>
-   `@update` 通过此函数来更新数据 <br/>
-   `@delete` 通过此函数来删除数据 <br/>

其中，插入相同主键的数据，可能会产生冲突，从而程序崩溃 <br/>
默认的冲突解决方法是 `abort` ，也可以自己定义方法为 `relpace` <br/>

```dart
@Insert(onConflict: OnConflictStrategy.replace)
Future<void> insert_one(Task task);
```


#### 数据库定义 `database.dart` {#数据库定义-database-dot-dart}

在文件中， <br/>

```dart
part 'database.g.dart';

@Database(version: 1, entities: [Task])
abstract class FlutterDataBase extends FloorDatabase {
  TaskDao get taskDao;
}
```

-   `part` 表示 `database.g.dart` 是该文件/模块的一部分？ <br/>
-   `FlutterDataBase` 是抽象类，继承自 `FloorDatabase` <br/>
-   `FlutterDataBase` 中定义了一个 `getter` <br/>
-   `@Database` 这个类看作一个数据库 <br/>
-   其中 `entities` 表示访问的数据表，通过重载 `get` ，返回 `DAO` 对象来访问数据表 <br/>


#### 代码生成 {#代码生成}

在 `database.dart` 所在目录下，输入 <br/>
`flutter pub run build_runner build` <br/>
会生成 `database.g.dart` 文件 <br/>
接下来的数据库操作就会通过这个文件 <br/>

**注意** <br/>
在 `database.dart` 中需要这样导入 `sqflite` <br/>

```dart
import 'package:sqflite/sqflite.dart' as sqflite;
```

因为 `build_runner` 生成的文件中有 `sqflite.Database` 等类声明 <br/>


#### 创建数据库 `main.dart` {#创建数据库-main-dot-dart}

在异步的主函数中，首先确认初始化 <br/>
`WidgetsFlutterBinding.ensureInitialized()` <br/>
再通过 `database.g.dart` 中的 `$FloorFlutterDatabase` 来创建数据库，再获取 `DAO` 对象 <br/>

```dart
final database = await $FloorFlutterDataBase
.databaseBuilder('file://./flutter_database.db')
.build();

final dao = database.taskDao;
```

**注意** <br/>
可以在 `databaseBuilder` 中传入数据库地址 <br/>


### 在数据库更新时刷新组件 {#在数据库更新时刷新组件}

使用 `FutureBuilder` 构造组件只能用一次 `future` ，这样的话组件不会感知到数据库的更新 <br/>
为了解决这个问题，我们将获取数据库数据的结果定义为 `Stream` ，再用 `StreamBuilder` 来构造 <br/>

首先，是重新定义一个数据库的查询方法，在 `task_dao.dart` 中 <br/>

```dart
@Query('select * from task')
Stream<List<Task>> findAllTasksAsStream();
```

之后，重新生成代码 <br/>
`flutter pub run build_runner build` <br/>

再是 `StreamBuilder` 传入 `stream` 与 `builder` <br/>

```dart
StreamBuilder<List<Task>>(
  stream: dao.findAllTasksAsStream(),
  builder: (_, snapshot) {
    if (!snapshot.hasData) return Container();

    final tasks = snapshot.requireData;

    return ListView.builder(
      itemCount: tasks.length,
      itemBuilder: (_, index) {
	return TaskListCell(
	  task: tasks[index],
	  dao: dao,
	);
      },
    );
  },
),
```

**疑问** <br/>
`snapshot.requireData` 能否使用 `snapshot.data!` 来替代 ?