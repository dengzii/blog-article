# AndroidStudio 插件开发记

插件作为提升工作效率的工具, 很大程度的加快了我们的开发速度, 给我们的工作带来了极大的便利. 在编码的时候, 可曾设想过如果 IDE 有 xx 功能
该多好, 相信大部分人都有过这种想法. 这篇文章记录了我学习开发 IntelliJ IDEA 插件的过程, 我是如何为 
AndroidSutdio 开发一个翻译插件.

**开发环境**

- 系统: Windows 10 
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

#### PSI

#### VFS

#### Editor

### 基本组件

#### Action

Action 定义了用户的一个动作, 快捷键, 我们创建一个 Action 需要一个类继承 AnAction, 并重写 actionPerformed(AnActionEvent anActionEvent) 方法, 之后在 plugin.xml 中注册该 Action,  基本上我们常用的数据上下文信息都可以在 anActionEvent 中获取.

定义一个 Action, 打印项目名, 路径, 及正在编辑的文件名

	public class MainAction extends AnAction {

		@Override
		public void actionPerformed(@NotNull AnActionEvent anActionEvent) {
			Project project = anActionEvent.getProject();
			PsiFile psiFile = anActionEvent.getData(LangDataKeys.PSI_FILE);
			System.out.println("Project Name:" + project.getName());
			System.out.println("Project Path" + project.getProjectFilePath());
			System.out.println("Editor File Name:" + psiFile.getName());
		}
	}

在 plugin.xml 中注册该 action, 所有的 Action 都定义在 <actions></actions> 中

	<actions>

        <action id="your_id_usually_is_doaim_and_action_name" class="com.your_domain.MainAction"
                text="This is action name"
                description="This is description" keymap="$default">
            <add-to-group group-id="ToolsMenu" anchor="first"/> 
            <keyboard-shortcut first-keystroke="alt G" keymap="$default"/>
        </action>
	</actions>

group-id 定义了该 Action 出现的位置, 这里是在菜单 Tools 的第一个位置,  first-keystroke 为快捷键, 组合键用空格分开, 比如 "ctrl shift alt G".

我们在 Tools 第一个选项即可看到 "This is action name" 这个选项, 点击或按快捷键即可出发该 Action.

#### UI Component

**ToolWindow**

ToolWindow 就是底部 Logcat, Event Log 依附在左右两侧或底部的窗口, 可以最小化成一个按钮, 或展开, 改变大小和位置关闭. 
在菜单栏中 View => Tool Window 列表中可以看到当前所有的 ToolWindow.

定义一个 ToolWindow, 显示当前项目名, 包上点击右键 new => Swing Ui Designer => GUI Form => TestToolWindow

点击 TestToolWindow.form 编辑界面, 添加一个 JLabel, 然后编辑 TestToolWindow, 让他实现 ToolWindowFactory 接口.

	public class TestToolWindow implements ToolWindowFactory {

		private JPanel rootPanel;
		private JLabel label1;

		public JPanel getContent() {
			return rootPanel;
		}

		@Override
		public void createToolWindowContent(@NotNull Project project, @NotNull ToolWindow toolWindow) {
			ContentFactory contentFactory = ContentFactory.SERVICE.getInstance();
			Content content = contentFactory.createContent(getContent(), "TestToolWindow", false);
			toolWindow.getContentManager().addContent(content);
		}
	}

在 plugin.xml 中注册, ToolWindow 需要放在 extensions 标签中.

    <extensions defaultExtensionNs="com.intellij">
        <toolWindow id="TestToolWindow"
                    canCloseContents="false"
                    factoryClass="com.your_domain.TestToolWindow"
                    anchor="bottom"/>
    </extensions>

其中, id 是 ToolWindow 的标题, canCloseContents 设置是否可以关闭, factoryClass 就是实现了 ToolWindowFactory 的该 ToolWindow 的工厂类. anchor 为显示位置

在之前写的 Action 中添加以下代码, 触发该 Action, ToolWindow 就弹出了并显示了项目的名称.

	public void actionPerformed(@NotNull AnActionEvent anActionEvent) {
	
		ToolWindow toolWindow = ToolWindowManager.getInstance(project).getToolWindow("TestToolWindow");
		toolWindow.show(new Runnable() {
			@Override
			public void run() {}
		});
		JTextField field = (JLabel) toolWindow.getContentManager()
				.getContent(0).getComponent().getComponent(0);
		if (field!=null){
			field.setText(project.getName());
		}
	}

**Dialog**

**Notification**

#### Service 

#### Application

#### 



