随着公司的发展, 项目的更新迭代越来越快, 一个项目往往由一个团队的不同角色来回周转协调后才能上线, 在开发交付并上线的过程中, 很多时候以为一些无意义重复的琐事浪费大量时间, 而 Jenkins 就是为了解决这些琐事而生的, Jenkins 集成代码检查, 测试, 构建, 打包, 上线等一系列操作, 实现了一键交付, 提升项目稳定性, 交接速度, 省去了交接过程的重复劳动. 作为一个 Jeknins 初学者, 本文记录了我使用 Jeknins 构建 Android 项目一些经验.

### 目录

- 什么是 Jenkins , And why
   + 起因
   + Jenkins 简介
   + 其他类似持续集成工具
- 基本配置 Jenkins 
   + 下载安装
   + 安装插件
   + 配置一些属性
- 在 Jenkins 中用 Gradle 构建 Android 项目 
- Gradle 与 Jenkins 的结合, 实际应用
   + 参数化构建过程, 如何传递给 Gradle
   + APK 归档
   + 配置 钉钉 构建通知

## 什么是 Jenkins , And why

### 起因

不知你是否有这样的经历: 项目马上就要上线了, 测试那边发现的问题一个接着一个, 你不断的改, 然后又不断打包, 手忙脚乱.
在你身心投入地写着代码的时候, 测试叫你打个 XXX 环境的包放到 xxx 目录, 然后你只能停下双手离开键盘等待打包完毕. 在你打着包的时候, 测试不断地催促你, 不断问你打包好了没有.

Jenkins 的出现, 让你更加愉快的写代码,在你打包的时候也可以干其他事情,它可以帮你定时打包, 或者在你提交到某个分支时打包, 或者让测试自己打包, 可以帮你进行代码测试, 它还可以帮你上线, 检查代码质量等等. 那些重复劳动, 他都可以帮你完成.

### Jenkins 简介

Jenkins是一个开源的、可扩展的持续集成、交付、部署（软件/代码的编译、打包、部署）的基于web界面的平台。允许持续集成和持续交付项目，无论用的是什么平台，可以处理任何类型的构建或持续集成。

软件开发的流程

    编码 --> 构建 --> 集成 --> 测试 --> 交付 --> 部署

### 其他类似持续集成工具

- Travis CI
- Circle CI
- TeamCity
- GitLab CI

## 基本配置 Jenkins 

### 下载安装

在这个地址下载 https://jenkins.io/zh/download/ , 如果下载的是 war 包, 直接运行以下命令

    java -jar jenkins.war

然后打开浏览器, 进入 localhost:8080 即可开始安装, Jenkins 的安装步骤也没啥难点.

下面就是 Jeknins 的界面了, 这里我用到了一款主题[jenkins-material-theme](http://afonsof.com/jenkins-material-theme) 需要安装 **Simple Theme Plugin** 这个插件, 主题在  **Manage Jenkins => Configure System **的 theme 选项中配置, 
这里配置了一个 css url: [http://afonsof.com/jenkins-material-theme/dist/material-light.css](http://afonsof.com/jenkins-material-theme/dist/material-light.css)

![jenkins][jenkins]

### 安装插件

进入菜单 **Manage Jenkins => Manage Plugins**, 这个地方就是管理插件的地方, 由于自带的源无法访问, 这里使用下面这个地址, 如果你在浏览器可以打开这个链接, 说明没问题, 否则请自行百度其他源地址

    http://mirror.esuni.jp/jenkins/updates/update-center.json

好了, 插件已经可以正常下载了, 以下是需要安装的插件

- Git plugin 
- GitHub API Plugin
- GitHub plugin 
- Gradle Plugin 
- Localization: Chinese (Simplified) 

基本插件都装好了

### 配置一些属性

一般情况下, 在 **Global Tool Configuration** 中, 我们可以配置插件, 比如JDK, GIT的安装路径 
在 **Configure System** 中, 我们可以配置 Jeknins 和插件的一些参数, 比如环境变量, 样式, Git令牌, 管理员邮件等等.

进入**Manage Jenkins => Configure System**, 参考下图配置 ANDROID_SDK 环境变, 具体值根据你的sdk路径设置, 但是必须三个键都要有且键名必须一样.

![android_sdk][android_sdk]

进入 **Manage Jenkins => Global Tool Configuration**, 参考下图配置 JDK 和 Git, 这两个都是根据具体安装路径来配置, 我这直接 配置 git.exe 是因为我将 git 配置到 Windows 的环境变量里了, 在命令行直接可以使用 git 命令

![jdk_git][jdk_git]

配置 gradle, name 最好取得有意义

![gradle][gradle]

Ok, 现在基本已经配置好了.

## 在 Jenkins 中用 Gradle 构建 Android 项目

进入 Jenkins 的首页, 点击 **新建Item**, 输入项目名称, 点击 **FreeStyle Project** 新建一个自由风格的项目, 然后点击创建, 可以看到如下界面

![test_project][test_project]

在这个界面就是配置项目, 下面我们添加一个 GitHub 仓库, **源码管理 >> Git** 按照下图添加一个仓库添加一个凭据, 凭据类型为 Username with password, 然后填入你的 GitHub 用户名密码,
最好也填一下, 然后就能在凭据下拉框中找到配置的账号密码了, Jenkins 将通过这个账号密码拉取配置仓库的代码.

在下图中 **Branches to build** 字段指定了哪个分支会被构建, 默认为 master 分支.

![repository][repository]

![git_password][git_password]

在构建触发器这块中, 我们可以配置如何出发构建, 比如访问某个链接, 提交代码到某个分支.

接着, 我们在构建中选择 **增加构建步骤 >> Invoke Gradle script**, 这就是调用 gradle 构建项目的步骤了, 选择我们之前配置的 gradle 版本, 然后 输入要运行的 **task**, 先 clean 之前的构建缓存, 然后执行 app 模块的 assemblRelease task, 构建所有的 release apk.

![invoke_gradle][invoke_gradle]

OK, 一个最基本的配置就好了, 保存, 进入到 job/TestProject/ 这个页面, 点击 **Build Now** 开始构建．在**Build History** 列表中, 最上面的就是了.

![building][building]

点击这个构建任务我们就可以进入到这个任务的详细情况了, 更变记录中有所有所有的提交记录(你构建分支的), 控制台输出就是本次构建执行的所有命令了

![build_detail][build_detail]

控制台log, 

![log][log]

在这里可以看到所有执行的命令和输出. 我们可以把命令在 cmd 中运行, 这对快速排查错误很有帮助, 比如 gradle 报错, 我们可以将下面这句命令在项目根目录中运行, 排查错误

    cmd.exe /C "D:\AndroidStudio\gradle\gradle-5.1.1\bin\gradle.bat :clean app:assembleRelease && exit %%ERRORLEVEL%%"

## Gradle 与 Jenkins 的结合, 实际应用

在上面的例子中, 我们只是简单的配置了, 并运行了一个最基础了构建. 下面我们配置一些更加实用的功能.

### APK 归档

我们构建完项目是不够的, 还需要把 APK 归档才行, 不然还得去项目目录下找, 打开项目配置, 在**构建后的操作** 一栏中, 增加一个构建后的步骤, 选择 **Archive the artifacts**, 如果我们没有
配置gradle 的 apk 输出路径, 那默认的就是 app/build/outputs/apk/**/*.apk, 如下图配置

![archive_artifacts][archive_artifacts]

其中, \*\*表示任意目录, \*.apk 代表以.apk 结尾的任意文件, 配置这个选项后, 每次构建成功后, Jeknins 都会将和你输入的规则相匹配的文件归档到构建结果中, 如下图

![build_result][build_result]

我们将可以在这里下载到打包好的 apk 文件

### 参数化构建过程, 如何传递给 Gradle

参数化构建过程可以在 Jenkins 中将参数传递给 Gradle, 就和我们在 gradle.properties 中配置属性一样, 然后我们可以在 gradle 脚本中再通过参数进行构建的步骤进行配置,

比如, 我们可以指定构建类型, debug 或 release, 指定风味, 等等

进入项目的配置页面, 在 **General** 栏中勾选 **This project is parameterized** , 表示这个项目是参数化的, 我们需要在 Jeknins 中吧参数传给项目构建脚本.

点击添加参数, 选择我们需要的参数类型, 其中 列表, 用换行表示列表项, 还可以给字段添加描述. 

![parameterized][parameterized]

之后, 我们还需要在 **构建** 栏中配置 gradle , 点击 **Invoke gradle script** 的 **高级** 勾选 **Pass all job parameters as Project properties** ,

表示传递构建任务的参数作为项目参数, 也就是我们刚刚在 **This project is parameterized** 中配置的参数, 我们还可以在 **Project properties** 中配置属性, 但是在这里配置的是固定的, 

不能在每次发起构建时选择参数值.

![gradle_properties][gradle_properties]

OK, 配置完后, 进入我们的项目详情页面, **Build Now** 按钮变成了 **Build with Parameters**, 点击后就不是马上构建而是让我们选择我们在配置中添加的参数, 在其他情况触发构建时(比如Git提交触发), 我们没法选择参数, 那 Jeknins 将使用默认值, 列表选择第一个值, 没有默认值的则为空.

![build_parameters][build_parameters]

然后, 我们就可以在 gradle 中使用这些参数了例如指定版本号和版本名
	
     defaultConfig {
        applicationId "com.example.jenkins"
        minSdkVersion rootProject.ext.minSdkVersion
        targetSdkVersion rootProject.ext.targetSdkVersion
        versionCode VERSION_CODE
        versionName VERSION_NAME
	}

同时, 我们还需要在 gradle.properties 中配置相应的属性, 因为我们不用 Jeknins 构建时, gradle 找不到这些参数会报错, 或者我们将 CI 脚本与平时编码的分开, 或新建一个分支用于构建

通过参数配置风味和构建类型, 在这里, 我们将版本名和版本号统一配置在了 ext 中, 然后通过 IS_JENKINS 判断是否为 jenkins 构建, 修改相应的值

	IS_JENKINS = Boolean.valueOf(IS_JENKINS)

	task jenkinsBuild (type:GradleBuild){

		if (!IS_JENKINS){
			return
		}
		rootProject.ext.versionCode = Integer.valueOf(VERSION_CODE)
		rootProject.ext.versionName = VERSION_NAME
		def flavors = BUILD_FLAVOR.trim().split("_")
		def allTasks = []
		flavors.each {
			def taskName = "assemble${it.capitalize()}${BUILD_TYPE.capitalize()}".toString()
			if (!allTasks.contains(taskName)){
				allTasks.add taskName
			}
		}
		println("""
		============= Build Info =============
		build tasks      :  $allTasks
		build versionName:  $rootProject.ext.versionName
		build versionCode:  $rootProject.ext.versionCode
		============= Build Info END =============
		""")

		tasks = allTasks
	}

到此为止, 参数化构建过程完毕. 如果我们开始构建后, 在控制台中能看到如下输出, 说明我们的配置生效了

![parameter_log][parameter_log]

### 配置 钉钉 构建通知

在钉钉群中, 进入 **群设置 >> 群机器人 >> 添加机器人 >> 添加自定义机器人 >> 配置** , 添加好后, 在机器人管理中选择添加的机器人, 保存在 webhook 中链接中的 access_token= 后的参数

进入 Jeknins, 在 **插件管理** 中搜索到 [Dingding JSON Pusher](https://plugins.jenkins.io/dingding-json-pusher) 并安装, 安装完后打开项目配置, 在 **构建后操作** 一栏中, 增加构建后的步骤, 

进入项目管理, 选择 **钉钉通知器配置** , 参考下图配置, 钉钉access token就填入我们申请的

![dingding][dingding]

好了, 每次构建时都将在 钉钉中收到通知了

![dingding_notify][dingding_notify]

点击卡片便可进入我们的构建详情中

(完)

[jenkins]: /static/jenkins.png

[android_sdk]: /static/android_sdk.png

[archive_artifacts]: /static/archive_artifacts.png

[build_detail]: /static/build_detail.png

[build_parameters]: /static/build_parameters.png

[build_result]: /static/build_result.png

[building]: /static/building.png 

[dingding]: /static/dingding.png

[dingding_notify]: /static/dingding_notify.png

[git_password]: /static/git_password.png

[gradle]: /static/gradle.png

[gradle_properties]: /static/gradle_properties.png

[invoke_gradle]: /static/invoke_gradle.png

[jdk_git]: /static/jdk_git.png

[jenkins]: /static/jenkins.png

[log]: /static/log.png

[parameter_log]: /static/parameter_log.png

[repository]: /static/repository.png

[test_project]: /static/test_project.png
