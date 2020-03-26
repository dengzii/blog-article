# Gradle 入门

Gradle 作为一个现代的基于 JVM 自动化构建工具, 它抛弃了 Maven 和 Ant 使用 xml 配置项目的繁琐形式,
使用 Groovy DSL, 或 Kotlin DSL 来配置构建项目, 它非常强大, 高可定制, 快速, 可用于构建 Java,
Android, C++, Kotlin, JavaScript 项目, 是 Android 官方构建工具.

[Gradle 官方用户引导](https://gradle.org/guides/#getting-started)

在这里, 我使用 Gradle 5.6.1 作为示例, Groovy DSL 作为例子, Groovy 非常简单, 如果你还
没有了解过, 可以前往 [官方文档](http://groovy-lang.org/documentation.html) 进行快速学习,
建议着重学习 **Closure (闭包)** 相关的内容, 如果想要深入了解 gradle, 那么了解**元编程**的知识
也是必要的.

## 什么是 DSL

DSL 是 Domain Specific Language 的缩写, 意思是领域特定语言, 但它不一定真的是一门独立语言, 更多
的时候, 它是一种风格或规则, 利用一语言的特性, 实现一种专门用于做某件事的框架, 它可以更好地描述
特定领域的对象, 规则和运行方式. DSL一般都非常简洁, 可以让人快速入门, 或者即使没有学过, 也可以
大概理解其意思. 与DSL 相反的是 GPL (General Purpose Language).

**为什么需要 DSL**

为了高效以及清晰的表述我们需要做的事情, 但是只是在特定的领域能够如此, 并且也只能应用于特定的领域.

想象一下我们用 Java 去构建 Android 项目, 原本简洁的 testImplementation 'xxx' 要替换成
 project.dependencies.test.add('xxx'), 当项目达到一定的大小, 构建脚本越来越多, 相信你能理解.

在 gradle 中, 有基于 groovy dsl 和 kotlin dsl 的两种项目配置声明方式, 在语法上有写不同, 这是由于
 groovy 和 kotlin 两种语言的特性决定的, 但基本原理完全一样, 结构也大体上相似. 相对于 kotlin dsl, 
 groovy 更加简洁, 但对于不熟悉 groovy 的人来说, kotlin 则更加易懂.

## 什么是 Gradle

**什么是构建工具?**

构建工具就是把软件项目的源码和资源通过一些工具帮我们实现自动化打包成最终的可执行应用程序的工具.
构建工具帮我们实现编译, 合并, 下载依赖的库, 单元测试, 部署等任务, 免去我们去做一些重复简单的事.

**为什么需要 Gradle**

在一些简单的项目中, 比如用 Java 实现一个打印 helloworld 的程序, 这很简单, 因为中间的构建步骤非常少,
我们只需要执行 javac 和 jar 两条命令即可打包成 jar. 但是安卓的构建步骤则非常复杂, 中间使用了 javac,
d8, dx , ProGuard, aapt 等一系列工具进行打包, 打包步骤颇为繁琐, 如果手动则会浪费大量的时间和精力. 并
且一旦项目形成一定的规模, 组织构建步骤, 顺序, 依赖会变得极其困难.

Gradle 是一个开源的自动化构建工具, 非常灵活, 几乎可以构建任何类型的软件项目. 

**Gradle 的几个特点**

1. 增量构建在执行任务时会检查输入以及输出是否变化以及使用可选的缓存, 使得 Gradle 构建项目变的非常快.
2. Gradle 是运行在 JVM 的. 也是用 Java 实现的, Java 程序员会比较喜欢.
3. 可扩展性很高, 可以自定义任务, 或者构建模型, Android 就是一个例子.
4. 几乎所有的IDE都内置对 Gradle 的支持.
5. 构建输出详细的 Log 可以方便快速定位构建时的问题.

虽然 Gradle 功能强大, 但是使用它依旧是非常简单易用的, 我们不必花费大量的时间去学习或编写构建脚本, 
这一点非常重要, 因为 Gradle 已经帮你一些准备工作, 它提前假设了你可能要做的事情以及构建的步骤.

## 基本概念

在 Gradle 中, 所有概念都建立在 project 和 task 这两个基本概念之上.

### Project

在 Gradle 中, build.gradle 文件所在的目录则为一个 Project, 一个 Project 可以包含多个 Project, 最外层
的 settings.gradle (这个文件时可选的, 当没有这个文件时表示单一项目构建, 有多个SubProject需要构建则必须有
个文件)所在的 Project 则为 RootProject, 例如在 Android 项目中, 项目目录是一个项目, 内部还包含若干模
块项目.

Project 代表什么取决你用 Gradle 构建什么,例如一个 jar, 或者一个 Android 项目, 或者什么构建产物都没有, 
只是为了完成某件事.

下面这个文件声明了三个SubProject, 相对路径分别是 /projectA, /projectB, /dir1/projectC.

settings.gradle
	
	include ':projectA', ':projectB', ':dir1:projectC'

### Task

每个 Project 都包含一个或者多个 Task, 每个 Task 都是为了做某件事情的细分任务, 比如编译一些 class, 打包
一个 Android APK, 或者清理缓存数据, 或者上传发布一个应用. 

Task 由 Actions, Inputs, Outputs 三部分组成, Actions 是 Task 的细分, Inputs 表示 Task 需要操作的值, 文件,
或者源码,目录等, Outputs 表示 Task 的产出文件, 或者目录. 这三个部分都是可选的, 这取决于你的 Task 需要做
什么.

Task 也可以依赖其他 Task, 当我们执行某个 Task  的时候, 如果他依赖了其他 Task, 这些 Task 也会一并执行,
就比如在 Android 我们执行 assembleRelease 打包 APK, 打包前必须先通过 appt 编译 layout, values 等资源文件, 
通过 D8 编译 java 字节码文件等等任务.

**几个固定构建步骤**

1. 初始化: 初始化构建环境, 确定哪些 Project 将参与构建.
2. 配置: 创建并配置需要执行的 Task , 确定执行顺序.
3. 执行: ...配置完就执行 Task


## 初识 Task

### 创建和运行Task

我们在任意一空文件夹创建一个 build.gradle, 这代表一个 Project 的入口, 必须是这个文件名, 我们在 build.gradle
中创建一个 Task, 如下代码.

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
	
执行 Task 的命令是 gradle TaskPath, 路径以 : 分隔, 比如 A 项目下的 C Task 则表示为 :A:C.

### Task的结构

在上面的例子中, 为什么要写在 doFirst 里呢, 这与 Task 执行顺序有关系, doFirst 表示最先执行
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

### Task 的状态(执行结果)

每当 Gradle 执行完一个 task, 针对执行结果会以不同的标签标记这个 task, 以下所说的执行均指的是 task 
的 actions 被执行.

1. EXECUTED : 这个 task 的已经执行.
2. UP-TO-DATE : 这个 task 的 **outputs** 没有发生改变, 前提是这个 task 有 inputs 和 outputs 并且他们没
有发生改变.
3. FROM-CACHE: 这个 task 的 **outputs** 使用上次构建的结果.
4. SKIPPED: 这个 task 没有被执行, 设置了 enable = false
5. NO-SOURCE: 这个 task 没有被执行, 原因是找不到 **inputs** (指定了inputs的情况下).

### 增量构建

**什么是增量构建?**

上面说到了 Task 的五种状态, 其中部分就是与增量构建相关的, Task 的输入和产出决定了 task 是否需要执行.在
项目比较庞大复杂的时候, 处理各种资源文件, 编译代码等等任务通常需要耗费大量的时间, 我们不可能在仅仅修改
了一行代码的情况下重新执行所有构建任务, 这个时候增量构建就显得尤为重要. 

大多数情况 task 都伴随着资源的处理, 比如将源码(inputs)编译成 class (outputs) 文件, 在上一次构建后没有改
变的文件, 通常是不需要再次编译的.

有个很关键的问题就是如何定义 inputs 和 outputs, 比如, 更改JDK的版本是否需要重新构建? jdk 的版本算不算 
inputs? jdk 的版本会影响输出 class, 所以算. 但是使用 win10 或者 macOS 就不算. 另外增量构建必须在至少有
一个输出的情况下才能正常工作.

**buildSrc**

为了方便编写和组织我们的自定义构建逻辑, 我们需要在项目根目录新建一个目录:
	
	/yourProject
	    ....
	    /buildSrc
	        /src/main/groovy/
	            ...
	        build.gradle

buildSrc/build.gradle

	repositories {
		google()
		jcenter()
	}
	apply {
		plugin 'groovy'
		plugin 'java-gradle-plugin'
	}
	dependencies {
		implementation gradleApi()
		implementation localGroovy()
		implementation "commons-io:commons-io:2.6"
	}

在这个目录中的代码可以在我们项目的任意脚本中访问(6.0之后无法再settings.gradle中访问).

**使用增量构建**

想要使用增量构建, 我们需要自定义一个 Gradle Task Type, 这个 Task 可以继承 DefaultTask , 
或者继承已存在的 Task, 比如 Copy, 然后告诉 Gradle 哪些属性将影响输出并标记为为输入, 哪些
属性是最终结果并标记为输出. 然后我们再写一个 Task 指定它的 type 为我们自己定义的类型.

接下来看一个简单的例子:

	class TestTask extends DefaultTask{
		private FileCollection imInput
		private File imOutput
		private Nested nested

		@Nested
		public Nested getNested() {
			return nested
		}
		@InputFiles
		public FileCollection getImInput() {
			return imInput
		}
		@OutputFile
		public File getImOutput() {
			return imOutput
		}
		public void setNested(Nested nested) {
			this.nested = nested
		}
		public void setImInput(FileCollection imInput) {
			this.imInput = imInput
		}
		public void setImOutput(File imOutPut) {
			this.imOutput = imOutPut
		}
	}
	class Nested{
		private String a
		private String b
		Nested(String a, String b){
			this.a = a
			this.b = b
		}
		@Input
		public String getA() {
			return a
		}
		public void setA(String a) {
			this.a = a
		}
		@Input
		public String getB() {
			return b
		}
		public void setB(String b) {
			this.b = b
		}
    }

在上面这个例子中, 我们使用了 Nested, OutputFile, InputFiles 三个注解表示输入和输出, 要使用增量构建, 则
我们必须在 getter 方法上添加相应的注解, 在 Gradle 中, 支持的输入输出类型有三种, 一种是实现了 Serializable
接口的类型, 二是文件, 三是组合值, 即一个组合其他输入输出的数据类, 更多注解请参考 <Gradle用户手册>, 其他注解
也都是基于这三种类型.

我们创建一个 task, 并且使这个 task 的 type 为 TestTask, 并给这些属性赋值, 两次执行这个 task 则会发现该 task 
已被标记为 UP-TO-DATE.

	task test(type: TestTask){
		imInput = new File('a.txt')
		imOutput = new File('output')
		nested = new Nested('a', 'b')
		v = 'a'
		doLast{
			println 'test execute'
		}
	}

### 执行 Task

一般情况, 我们会使用绝对路径指定需要执行的 Task, 例如在 projectA 下的 uploadRelease.

	gradle :projectA:uploadRelease

以上是一般情况, 但是一些情况下我们需要执行多个项目中的同名的 Task, 比如 projectA 和 projectB 都需要执行 
uploadRelease, 这时我们即可以省略路径.

	gradle uploadRelease

如果嫌 uploadRelease 太长, 你甚至可以这样
	
	gradle uR
	// 或者这样
	// gradle up

第一种是使用驼峰命名法时每个单词的开头, 第二种是匹配 task 名的开头

**在 SubProject下执行命令**

如果在根项目中执行非绝对路径的 Task, 则所有名字匹配的 Task 都会执行, 但我们可以转到需要执行的 task 的 subproject
中执行命令, 则只会执行该 subproject 的匹配 task. 需要注意的是, 在 subproject 中执行 task, 除了不用指定绝对路径, 
其他过程与在 rootProject 中执行没有差别, 构建步骤不会因此改变.

## Project 的概念

在 Gradle 中 Project 表述了项目如何构建, 项目的所有构建属性. 一个 Project 对应一个 build.gradle 脚本, 在
build.gradle 中, 所有的代码都属于 Project Configure 块, 在 build.gradle 中的代码将在 Project 配置阶段执行.
另外, build.gradle 或其引入的其他脚本中, 已在 Project 的作用域内, 默认我们可以不带前缀直接引用 Project 的
方法或者属性, 比如我们创建 task 时使用的 task 方法, 按住 Ctrl 点击该方法即可跳转到源码中 Project 的对应方法上.

本质上, Project 就是一堆 Task 的集合, 每个 Task 完成一个细分任务. 

**获取一个Project**
	
	public Project project(String path);

### Apply插件或脚本

Project 可以导入构建脚本或插件, 导入其他脚本用于拆分逻辑, 拆分公共脚本. 通过依赖插件我们可以添加远程依赖拓展功能,
 比如一个打包上传到 maven 的工具.

	// apply from: 'a.gradle' // 导入其他脚本
	apply plugin: 'maven' // 这个是内置插件

当 Gradle 执行任意一个构建脚本时, 都会先将他编译成一个实现了 Script 接口的类, 并且该脚本内代码都在 Script 的作用域内,
我们可以访问 Script 接口中所有的属性和方法, 例如 apply 导入其他脚本, delete 删除文件, mkdir 新建文件夹.

导入插件时, 如果是非内置插件, 则还需要在 buildscript 块中添加依赖, buildscript 块中的代码会在 apply 块之前执行.

	buildscript {
		dependencies {
			classpath "com.example.plugin:you-plugin-id:1.0.0"
		}
	}

### 扩展属性

扩展属性可定义该扩展属性的 Project 的作用域中读写, Task 或者子 Project, 引入的脚本中都可以使用.

	apply from:'b.gradle'
	// 定义一个名为 extraName 的扩展属性
	project.ext.extraName = '123'
	// 另一种定义方式 
	//project.ext {
	//	extraName = '1234'
	//}
	task A{
		println ext.extraName
		ext.extraName = '567'
	}
	// b.gradle
	task B {
		doLast {
			// 依旧可以使用
			println project.ext.extraName
		}	
	}
	
### 几个实用的 API

**project.allprojects**

这个方法可以遍历所有 Project, 另外还有一个 subprojects 方法用于遍历所有SubProject, 如果我们想快速给所有 Project 
添加一个相同的 Task, 则需要用到它.

	allprojects {
		println it
		beforeEvaluate {
			println 'project evaluate ' + it.name
		}
	}

这个例子将在所有 project 配置前打印名字.

**project.buildscript**



**project.configure**

添加配置代码到 project

	// 给单个 Project 添加
	void configure(Closure c)
	// 给多个添加
	void configure(Iterator<?> i, Closure c)

**project.dependencies**



## 构建顺序

### 简介

项目中的所有 task 组成了一个有向无环图, task 的执行顺序则是按照有向无环图执行. 项目, task主要有三步, 初始化, 
配置, 执行.

**1. 初始化**

这个阶段, 首先配置执行全局初始化脚本(GRADLE_HOME/init.d, 如果有), 然后导入项目的 gradle.properties 文件中的
值(如果有).

然后如果存在 settings.gradle 文件, 则表示这是一个多项目构建, 执行这个脚本, 导入其中配置的项目.

**2. 配置**

在 Project 配置阶段, 代码从上往下执行(导入其他脚本则也是), 但 buildscript 块除外, 这个块一般用于配置 classpath,
会在配置阶段最先执行. 

Project 中的 Task 也是从上往下按顺序执行 task 的 configure closure.

**3. 执行**

如果我们指定了一个任务执行或者指定了 defaultTasks, 则这个阶段开始执行 task

### 监听project和Task

构建脚本中, 我们可以为构建添加监听.

**Project**

Project 有两个可以监听的接口, afterEvaluate 和 beforeEvaluate, 分别可以监听 project 构建开始和结束.

build.gradle

	allprojects {
		beforeEvaluate { p ->
			println "${p.name} beforeEvaluate"
		}
		afterEvaluate { p ->
			println "${p.name} afterEvaluate"
		}
	}

**Task**

同样的 Task 也可以.

	// task 被添加(配置完毕)
	tasks.whenTaskAdded {
		println name
	}
	// task 执行后
	gradle.taskGraph.afterTask {
        println it
    }
	// task 执行前
    gradle.taskGraph.beforeTask {
        println it
    }
	// 所有 Task 都配置完毕
	gradle.taskGraph.whenReady {
		println "All Task Ready"
	}

### 监听Gradle构建

Gradle 对象定义了许多对 Gradle 的方法, 我们可以通过 project.gradle 获取该对象. 下面这个例子用它来监听构建过程.

	// settings.gradle 中的 project 被加载时调用
	gradle.projectsLoaded {
		println "projectsLoaded $it"
	}
	// project 配置完毕
	gradle.afterProject {
		println "afterProject $it"
	}
	// project 配置前
	gradle.beforeProject {
		println "beforeProject $it"
	}
	// 所有构建完成
	gradle.buildFinished {
		println "buildFinished $it"
	}

## Gradle Daemon

由于启动 Gradle 的时间比较长, 为了节约时间, Gradle 通常在首次构建的时候会启动一个守护进程(Gradle Daemon). 它会
将项目配置缓存在内存中, 以加快我们的构建速度. 在 3.0 以上的版本中, Gradle Daemon 将会默认启用. 使用 
gradle --status 即可看到关于守护进程的信息

但守护进程启动后, 如果在后台超过3小时不活动则会关闭, 我们也可以使用 gradle --stop 命令手动关闭.

默认情况下, 一个守护进程会分配 512MB 内存, 我们也可以在项目的 gradle.properties 中配置大小, 例如设置 1024MB :
 org.gradle.jvmargs=-Xmx1024m.

