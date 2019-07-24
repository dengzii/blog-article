随着公司的发展, 项目的更新迭代越来越快, 一个项目往往由一个团队的不同角色来回周转协调后才能上线, 在开发交付并上线的过程中, 很多时候以为一些无意义重复的琐事浪费大量时间, 而 Jenkins 就是为了解决这些琐事而生的, Jenkins 集成代码检查, 测试, 构建, 打包, 上线等一系列操作, 实现了一键交付, 提升项目稳定性, 交接速度, 省去了交接过程的重复劳动. 作为一个 Jeknins 初学者, 本文记录了我使用 Jeknins 构建 Android 项目一些经验.

### 目录

- 引子
- 如何安装配置
- 构建 Android
- 集成 Git
- 在 Jeknins 使用 Gradle 

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

![jenkins](https://www.baidu.com/img/dong_962301698d83dc26fa8e7b38011d1705.gif)

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

![jenkins](https://www.baidu.com/img/dong_962301698d83dc26fa8e7b38011d1705.gif)

进入 **Manage Jenkins => Global Tool Configuration**, 参考下图配置 JDK 和 Git, 这两个都是根据具体安装路径来配置, 我这直接 配置 git.exe 是因为我将 git 配置到 Windows 的环境变量里了, 在命令行直接可以使用 git 命令

![jenkins](https://www.baidu.com/img/dong_962301698d83dc26fa8e7b38011d1705.gif)

配置 gradle, name 最好取得有意义

![jenkins](https://www.baidu.com/img/dong_962301698d83dc26fa8e7b38011d1705.gif)

Ok, 现在基本已经配置好了.

## 在 Jenkins 中用 Gradle 构建 Android 项目

进入 Jenkins 的首页, 点击 **新建Item**, 输入项目名称, 点击 **FreeStyle Project** 新建一个自由风格的项目, 然后点击创建, 可以看到如下界面

![jenkins](https://www.baidu.com/img/dong_962301698d83dc26fa8e7b38011d1705.gif)

在这个界面就是配置项目, 下面我们添加一个 GitHub 仓库, **源码管理 >> Git** 按照下图添加一个仓库添加一个凭据, 凭据类型为 Username with password, 然后填入你的 GitHub 用户名密码,
最好也填一下, 然后就能在凭据下拉框中找到配置的账号密码了, Jenkins 将通过这个账号密码拉取配置仓库的代码.

在下图中 **Branches to build** 字段指定了哪个分支会被构建, 默认为 master 分支.

![jenkins](https://www.baidu.com/img/dong_962301698d83dc26fa8e7b38011d1705.gif)

![jenkins](https://www.baidu.com/img/dong_962301698d83dc26fa8e7b38011d1705.gif)

在构建触发器这块中, 我们可以配置如何出发构建, 比如访问某个链接, 提交代码到某个分支.

接着, 我们在构建中选择 **增加构建步骤 >> Invoke Gradle script**, 这就是调用 gradle 构建项目的步骤了, 选择我们之前配置的 gradle 版本, 然后 输入要运行的 **task**, 先 clean 之前的构建缓存, 然后执行 app 模块的 assemblRelease task, 构建所有的 release apk.

![jenkins](https://www.baidu.com/img/dong_962301698d83dc26fa8e7b38011d1705.gif)

OK, 一个最基本的配置就好了, 保存, 进入到 job/TestProject/ 这个页面, 点击 **Build Now** 开始构建．在**Build History** 列表中, 最上面的就是了.

![jenkins](https://www.baidu.com/img/dong_962301698d83dc26fa8e7b38011d1705.gif)

点击这个构建任务我们就可以进入到这个任务的详细情况了, 更变记录中有所有所有的提交记录(你构建分支的), 控制台输出就是本次构建执行的所有命令了

![jenkins](https://www.baidu.com/img/dong_962301698d83dc26fa8e7b38011d1705.gif)

控制台log, 

![jenkins](https://www.baidu.com/img/dong_962301698d83dc26fa8e7b38011d1705.gif)

在这里可以看到所有执行的命令和输出. 我们可以把命令在 cmd 中运行, 这对快速排查错误很有帮助, 比如 gradle 报错, 我们可以将下面这句命令在项目根目录中运行, 排查错误

    cmd.exe /C "D:\AndroidStudio\gradle\gradle-5.1.1\bin\gradle.bat :clean app:assembleRelease && exit %%ERRORLEVEL%%"

## Gradle 与 Jenkins 的结合, 实际应用

在上面的例子中, 我们只是简单的配置了, 并运行了一个最基础了构建. 下面我们配置一些更加实用的功能.

### APK 归档

### 参数化构建过程, 如何传递给 Gradle

### 配置构建通知







