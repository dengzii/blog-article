# AndroidStudio 插件开发记

插件作为提升工作效率的工具, 很大程度的加快了我们的开发速度, 给我们的工作带来了极大的便利. 在编码的时候, 可曾设想过如果 IDE 有 xx 功能
该多好, 相信大部分人都有过这种想法. 这篇文章记录了我学习开发 IntelliJ IDEA 插件的过程, 我是如何为 
AndroidSutdio 开发一个翻译插件.

**开发环境**

- 系统: Windows 10 工作站版
- 工具: IntelliJ IDEA 2019.2.1 Community Edition
- SDK: Java 8
- AndroidStudio: Android Studio 3.5

### 插件的大致分类

#### 语言支持

For example, Gradle, Scala, Groovy, 这些插件 IDEA, AndroidStudio 都是自带有的, 所以我们才能在编写这些语言代码的时候有语法高亮检测.

语言插件主要的一些功能是 文件类型识别, 语法高亮检测, 格式化, 语法提示等等.

#### 框架集成

AndroidStudio 就是一个例子, 他集成了 AndroidSDK 的一系列功能,  比如, 资源文件识别组织与提示, 集成 Gradle, Debug, ADB, 打包APK, 让我们可以更好的开发 Android 应用. 类似的插件还有 Gradle, Maven, Spring Plugin等. 

框架集成插件的主要功能是某种语言特定代码的识别, 直接访问特定框架的功能

#### 工具集成

例如, 翻译工具, Markdown View, WiFi ADB等.

#### UI增强

对 IDE 的主题颜色样式做一些更改, 比如 MaterialTheme.

### 开始一个项目

#### 两种工具开发插件项目

**Gradle**

本着紧随时代潮流的想法, 刚开始我是用 Gradle 构建项目的, 但是, 我在 IDEA Community 2019.2.1 版本中, 无论如何都无法成功, Gradle 提示找不到 PsiJavaFile 这些类, 但项目中是可以引用的. 我尝试了换 Gradle版本, 5.4, 4.5.1 两个版本, 把 jar 包移到 libs 中依赖, 均以失败告终, 如果有人可以编译运行, 请千万要告诉我. 

**DevKit**

创建项目:

    New Project => IntelliJ Platform Plugin => Input Project Name => Finish

配置项目: 

    File => Project Structure
				Project => Project SDK => IntelliJ IDEA Community Edition IC-xxx
				Module => Select your module => Tab Dependencies => Module SDK => Project


创建完毕后, 你的目录结构应如下

	resource/
		META-INF/
			plugin.xml	// plugin config file
	src/	// source code directory

#### plugin.xml

	<idea-plugin url="https://www.your_plugin_home_page.com">	

		<name>Your plugin name</name>

		<id>com.your_domain.plugin_name</id>

		<description>Your will see it at plugin download page</description>

		<change-notes>What's update</change-notes>

		<version>1.0.0</version>
	</idea-plugin>

### 几个概念

#### 线程规则

在 IntelliJ IDEA 平台中, 分为 UI 线程和后台线程, 这点和 Android 开发类似, 不同的是, 

**读**取操作可以在任何线程进行, 但在其他线程中读取需要使用 ***ApplicationManager.getApplication().runReadAction()*** 或者 ***ReadAction.run/compute*** 方法

**写**操作只允许在 UI 线程进行, 必须使用 ***ApplicationManager.getApplication().runWriteAction()*** 或 ***WriteAction.run/compute*** 进行写操作

ModalDialog

Background process

Topic

### 基本组件

#### Action

#### UI Component

#### Service 

#### Application






