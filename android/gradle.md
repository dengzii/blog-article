# Gradle 入门

Gradle 作为一个现代的基于 JVM 构建工具, 它抛弃了 Maven 和 Ant 使用 xml 配置项目的繁琐形式,
使用 Groovy DSL, 或 Kotlin DSL 来配置构建项目, 它非常强大, 高可定制, 快速, 可用于构建 Java,
Android, C++, Kotlin, JavaScript 项目, 是 Android 官方构建工具.

[Gradle 官方用户引导](https://gradle.org/guides/#getting-started)

在这里, 我使用 Gradle 5.6.1 作为示例, 使用 Groovy DSL 作为例子, Groovy 非常简单, 如果你还
没有了解过, 可以查看前往 [官方文档](http://www.groovy-lang.org/syntax.html) 进行快速学习,
建议着重学习 **Closure (闭包)** 相关的内容, 如果想要深入了解 gradle, 那么了解**元编程**的
知识也是必要的.

## 什么是 DSL

DSL 是 Domain Specific Language 的缩写, 意思是领域特定语言, 但它不一定真的是一门独立语言, 更多
的时候, 它是一种风格或规则, 利用一语言的特性, 实现一种专门用于做某件事的框架, 它可以更好地描述
特定领域的对象, 规则和运行方式. DSL一般都非常简洁, 可以让人快速入门, 或者即使没有学过, 也可以
大概理解其意思. 与DSL 相反的是 GPL (General Purpose Language).

**为什么需要 DSL**

为了高效以及清晰的表述我们需要做的事情, 但是只是在特定的领域能够如此, 并且也只能应用于特定的领域.

想象一下我们用 Java 去构建 Android 项目, 原本简洁的 testImplementation 'xxx' 要替换成
 project.dependencies.test.add('xxx'), 相信你能理解.

在 gradle 中, 有基于 groovy dsl 和 kotlin dsl 的两种实现, 在语法上有写不同, 这是由于 groovy 和 kotlin 
两种语言的特性决定的, 但基本原理完全一样, 结构也大体上相似.

## 基本概念

在 Gradle 中, 所有概念都建立在 project 和 task 这两个基本概念之上.

## Project

在 Gradle 中, 构建脚本所在的目录为 rootProject, 根项目可以有一个以上子 Project, 例如在 Android 项目中, 
项目目录是一个项目, 内部还包含若干模块项目, project 代表什么取决你用 gradle 构建什么,例如一个 jar, 或者一个 
Android 项目, 或者什么构建产物都没有, 只是为了完成某件事, Project 可以依赖其他 Project.

## Task

每个 Project 都包含一个或者多个 Task, Task 是 Project 的细分任务, 比如编译一些 class, 打包
一个 Android APK, 或者清理缓存数据, 或者上传发布一个应用. 

Task 也可以依赖其他 Task, 当我们执行某个 Task  的时候, 它也许依赖了很多 Task, 这些 Task 也会一并执行,
就比如在 Android 我们执行 assembleRelease 打包 APK, 打包前必须先通过 appt 编译 layout, values 等资源文件, 
通过 D8 编译 java 字节码文件等等任务.

## 初识 Task

### 创建和运行Task

我们在任意一空文件夹创建一个 build.gradle. 每一个项目下都有一个 build.gradle , 这个文件必
须有且必须是这个文件名, 我们在 build.gradle 中创建一个 Task.

build.gradle

	task myTask {
		doFirst {
			println 'Hi, gradle'
		}
	}
	// Task 也可以这样添加
	// task "myTask" {
	// ...
	// 或者这样
	// tasks.create("myTask"){ 
	// ...

在改目录下打开终端, 执行该 Task 表示我们将会看到以下效果, 我们也可以加上 -q 标识纯净输出, 
而不打印其他 log.

	$ gradle :hello

	> Task :hello
	Hi, gradle
	
执行 Task 的语法是 gradle TaskPath, 路径以 : 分隔, 比如 A 项目下的 C Task 则表示为 :A:C, 
当该 Task 在当前项目下时, 即路径为 :taskName 时, 冒号可省略.

### Task的结构

在上面的例子中, 为什么要写在 doFirst 里呢, 这与 Task 内部执行顺序有关系 doFirst 表示最先执行
这个块的代码, doLast 则表示最后执行, 而写在 Task 块中的则是配置该项目时执行的, 看下面的例子.

build.gradle 

	task hello {
		doFirst {
			println 'Hi, gradle'
		}
		println 'config task hello'
	}

	task run {
		doFirst {
			println 'task run doFirst....'
		}
		println 'config task run...'
		doLast {
			println 'task run deLast...'
		}
	}

我们执行 gradle run 命令运行 run 这个 Task, 结果如下, 显而易见, Task 块中的代码是配置 Project
 的时候就会执行的, 无论是否运行该 Task, 而 doFirst 和 doLast 则是 Task 真正的代码.

	$ gradle run

	> Configure project :
	config task hello
	config task run...

	> Task run
	task run doFirst....
	task run deLast...

	BUILD SUCCESSFUL in 466ms
	1 actionable task: 1 executed


### Task 依赖

Task 可以依赖其他 Task, 也就是, 依赖的 Task 执行完, 然后执行自己, 不同 project 下的 Task 也可以依赖.

	task run {
		dependsOn hello
	}
	// 也可以这样 
	// task run(dependsOn:['hello']){
	// ...
	// 或者在 build.gradle
	// run.dependsOn hello
	// 还可以这样
	/// run.dependsOn {  [hello]  }
	
这样, 在执行 run Task 的时候, 会先执行 hello.

### 动态操作 Task

Task 可以在构建的任意时刻创建, 如下例所示.

build.gradle

	// 这段代码执行在 project 的 Configure 阶段
	3.times { i ->
		task "task$i" {
			println "This is Task$i"
		}
	}

在终端运行命令: gradle, 会发现, 三个 Task 成功创建

通过上面的一些案例, 你也许发现了, Task 可以动态操作, 只要能访问到该 Task, 该 Task 已经创建, 我们
就可以直接操作该 Task 的所有属性.

build.gradle

	task a(dependsOn:['myTask']){
		doFirst{
			println 'task a'
		}
	}
	task "myTask" {
		doFirst{
			println 'mytask'
		}
		a.doLast {
			println 'a.doLast...'
		}
	}

例如, 上面的这个例子, 我们在 myTask 中给 a 添加了一个 doLast, 执行 gradle a 运行该命令将可以看到
a.doLast... 被打印, 除了添加 doLast, 我们还可以设置 dependsOn, doFirst, configure 等等.

一个 Task 可以添加多个 doFirst 和 doLast, 并在执行该 Task 时按添加顺序执行.

### Task 常用 API


在 project 的 build.gradle 中可以访问 project 的所有字段和方法, project 可以省略

	// 创建 Task
	project.task(String name)
	project.task(Map<String, ?> args, String name)
	project.task(String name, Closure configureClosure)
	// 通过 TaskContainer 创建
	project.tasks.create(String name)
	// 配置项目的默认 Task, 这些 Task 将在 Project configure 完成的时候执行
	defaultTasks(String... name)
	// 获取当前 project 的所有默认 task
	project.getDefaultTasks()
	// 获取 TaskContainer
	project.getTasks()
	// 查找 Task
	project.tasks.findByName()
	// 通过 path 查找 Task
	tasks.getByPath(String path)
	// 监听project Task 添加
	project.tasks.whenTaskAdded(Closure closure)
	
	// 添加自定义属性, 只要能引用该 Task 即可获取这个自定义属性, Project 同理
	ext.myProperty = 'abc'
	// 获取自定义属性
	[project|taskName].ext.myProperty
	// 添加 Task 结束代码块
	doLast(Closure c)
	// 添加 Task 开始代码块
	doFirst(Closure c)
	// 配置 Task
	configure(Closure c)
	// 是否启用
	setEnabled(boolean c)
	// 设置依赖的 Task
	dependsOn(Object... c)
	// 获取 Task 所在的 project
	getProject()
	// 设置 Task 超时时间, 如果任务超过这个时长任务会标记为 FAILED, 然后继续执行后面的任务
	// 不能响应中断的 Task 设置超时无效
	setTimeOut(long mil)
	
### Task 类型

不同类型的 Task 可以帮助我们完成不同的任务, 在创建 Task 的时候我们可以指定类型, 若未指定类型
则默认为 DefaultTask, 在指定了Task 的类型后, 这个 Task 则完全复制且拥有了指定类型的 Task 的所
有行为, 这与继承类似. Gradle 中预设了一些常用的类型, 我们创建 Task 的时候可以使用它们.

例如 Copy 类型用于复制任务

	task copy(type: Copy) {
	   from 'build/apk' // 源目录 
	   into 'archive'  // 目标目录
	   include('*.apk') // 包含
	   exclued('*.json')// 不包含
	}

安卓 clean Task 的类型是 Delete

	task clean(type: Delete) {
		delete rootProject.buildDir
	}

**自定义 Task 类型**

build.gradle

	

### Task 的执行顺序

很多时候时候, Task 是需要按顺序执行的, 比如 build 一定在 clean 后运行, 在打包前验证资源文件是否
正确, 在所有单元测试任务后生成单元测试测试报告. 决定运行顺序的两个 Task 可以没有任何依赖关系,执
行顺序和依赖的不同就是, 顺序只决定了执行先后, 而依赖则确保了依赖的 Task 一定会执行且比被依赖的
 Task 先执行.

使 taskY 一定在 taskX 前运行:

	taskX.mustRunAfter taskY

除了 mustRunAfter , Task 还有一个方法 shouldRunAfter , 从方法名可开出, 它没有 mustRunAfter 那么严
格, 那什么情况是使这个方法不生效的特例呢? 一是会导致循环的情况, 比如, taskA, taskB 相互依赖, 或者
更多个排序形成循环. 第二种情况是使用平行任务且除 showRunAfter 外所有依赖都已满足, 当 Task 的顺序
不是很重要的时候应该使用 shouldRunAfter.


## Task 进阶

### 增量构建

**什么是增量构建?**

在项目比较庞大复杂的时候, 处理各种资源文件, 编译代码等等任务通常需要耗费大量的时间, 我们不可能在仅仅
修改了一行代码的情况下重新执行所有构建任务, 这个时候增量构建就显得尤为重要. 





## Task 的状态 




















