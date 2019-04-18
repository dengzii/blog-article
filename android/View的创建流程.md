
在 Android App中, 所有的数据内容都是通过 View 展示给用户的, Android 通过一系列机制和流程将这些承载着各种交互控件和展示数据的 View 展示出来. 
在开发中, 我们也经常需要用到自定义 view, 因此, 我们非常有必要学习一下 view 的创建流程, 
本文从源码出发, 循序渐进介绍了了 Activity, PhoneWindow, DecorView, View 和 ViewGroup 的关系, 是如何一步步绘制最终展示出来的, 重点介绍了 View 的 measure, layout, draw 这几个方法. 

## 一. Activity
---
注意, AppCompatActivity 是有所区别的.

通常一个 App 是有许多 Activity 组成的， 我们在创建一个 Activity 的时候, 通常首先就是重写 onCraete 方法并调用 setContentView 传入相应的布局资源, 用于加载相应的 XML 布局文件,
 如果没有设置, 则该 Activity 就只有一个 ActionBar 和一个背景色(一般为白色), 我们看到的内容就是 Window 中的 DecorView， Window 是 Activity 的顶层容器， 其次是 DecorView, 而 DecorView，
 中则包含了 ActionBar, 和我们的 XML 布局了, 他们的结构大致如下图所示

        Android UI 层级
    
    |--------------------|
    |    Activity         |
    |--------------------|
    |    PhoneWindow      |
    |--------------------|
    |    DecorView        |
    |--------------------|
    |    | ActionBar |    |
    |    |-----------|    |
    |    |ContentView|    |
    |                     |
    |--------------------|
    

*Activity.setContentView*

    public void setContentView(@LayoutRes int layoutResID) {
        getWindow().setContentView(layoutResID);
        initWindowDecorActionBar();
    }

首先, 它调用了 Window.setContentView, 这个 Window 就是 PhoneWindow 了, Activity 中我们看到的 View 都封装在这个类中.

接着初始化 ActionBar 

    private void initWindowDecorActionBar() {
        Window window = getWindow();
        window.getDecorView();
        if (isChild() || !window.hasFeature(Window.FEATURE_ACTION_BAR) || mActionBar != null) {
            return;
        }
        mActionBar = new WindowDecorActionBar(this);
        mActionBar.setDefaultDisplayHomeAsUpEnabled(mEnableDefaultActionBarUp);
        mWindow.setDefaultIcon(mActivityInfo.getIconResource());
        mWindow.setDefaultLogo(mActivityInfo.getLogoResource());
    }

在这  window.getDecorView() 这句有点奇怪，刚开始想这个方法不是获取 window 的 decorview 吗？ 跟进去才发现，这方法里是会给 window 安装 decorview 和初始化 ContentParent 的, 因为在 window 的 decorview 为空的情况下是会创建的.

*PhoneWinow.getDecorView*

    @Override
    public final View getDecorView() {
        if (mDecor == null || mForceDecorInstall) {
            // 初始化 DecorView , ContentParent
            installDecor();
        }
        return mDecor;
    }
    
由此可见, Activity 直接参与用户界面绘制并不多.

## 二.PhoneWindow 
---
Window 代表着一个抽象窗口, PhoneWindow 是 Window 的具体实现, 且只有 PhoneWindow 这一个实现类, Window 并不具备 View 的一些特性, 比如可见, 长宽高这些. 正如它的类名, 它代表着 App 中一个窗口的抽象, 它控制着一些和窗口相关的操作, 和包含着
一些窗口的属性, 比如, 过度动画的绘制, 管理菜单, 标题, 以及它拥有的一个重要成员变量 DecorView.

PhoneWindow 这个类位于 com.android.internal 包下, 这个包中的类我们的 SDK 是没有的, 所以就算你下载了相应的 SDK 源码也看不到 PhoneWindow 的代码.

https://github.com/anggrayudi/android-hidden-api, 下载这个仓库中对应版本的.android.jar 替换 sdk\platforms\android-{version}\android.jar, 你就可以看到了.

PhoneWindow 伴随着 Activity 的创建而创建, 而 ActivityThread 掌握着 Activity 的创建, 我们看看 Window 是如何创建并与 Activity 关联的, 下面是 ActivityThread 中的 performLaunchActivity 方法

*ActivityThread.performLaunchActivity*

    private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
        ...
		// 创建 Activity 的上下文
        ContextImpl appContext = createBaseContextForActivity(r);
        Activity activity = null;
        try {
            java.lang.ClassLoader cl = appContext.getClassLoader();
			// Activity 创建了
            activity = mInstrumentation.newActivity(cl, component.getClassName(), r.intent);
            ...
        } catch (Exception e) {
            ...
        }
        try {
            Application app = r.packageInfo.makeApplication(false, mInstrumentation);
            ...
            if (activity != null) {
                ...
                Window window = null;
                if (r.mPendingRemoveWindow != null && r.mPreserveWindow) {
					// window 创建了
                    window = r.mPendingRemoveWindow;
                    r.mPendingRemoveWindow = null;
                    r.mPendingRemoveWindowManager = null;
                }
                appContext.setOuterContext(activity);
				// 初始化 Activity 相关的内容, Activity 的这个方法中关联了 window 
                activity.attach(appContext, this, getInstrumentation(), r.token,
                        r.ident, app, r.intent, r.activityInfo, title, r.parent,
                        r.embeddedID, r.lastNonConfigurationInstances, config,
                        r.referrer, r.voiceInteractor, window, r.configCallback);

                ...
                if (r.isPersistable()) {
                    mInstrumentation.callActivityOnCreate(activity, r.state, r.persistentState);
                } else {
                    mInstrumentation.callActivityOnCreate(activity, r.state);
                }
                ...
            }
        } catch (SuperNotCalledException e) {
            throw e;
        } catch (Exception e) {
            ...
        }
        return activity;
    }

根据该方法的名字可以得知, 这个方法是用于运行一个 Activity 的, 在这个方法中, 初始化了 Activity, 比如 Context, theme, PackageInfo 等, 调用了 Activity 的　onCreate 方法.
其中 activity = mInstrumentation.newActivity 这句代码实例化了一个新的 Activity,  activity.attach 这句代码实例化了 PhoneWindow 关联了 Context, 主线程等等.

*Activity.attach*

    final void attach(Context context, ActivityThread aThread,
			...
            Window window, ActivityConfigCallback activityConfigCallback) {
        attachBaseContext(context);
        
        mFragments.attachHost(null /*parent*/);
        
        mWindow = new PhoneWindow(this, window, activityConfigCallback);
        mWindow.setWindowControllerCallback(this);
        mWindow.setCallback(this);
        mWindow.setOnWindowDismissedCallback(this);
        mWindow.getLayoutInflater().setPrivateFactory(this);
        if (info.softInputMode != WindowManager.LayoutParams.SOFT_INPUT_STATE_UNSPECIFIED) {
            mWindow.setSoftInputMode(info.softInputMode);
        }
        if (info.uiOptions != 0) {
            mWindow.setUiOptions(info.uiOptions);
        }
        mUiThread = Thread.currentThread();
        
        ...
        
        mWindow.setWindowManager(
                (WindowManager)context.getSystemService(Context.WINDOW_SERVICE),
                mToken, mComponent.flattenToString(),
                (info.flags & ActivityInfo.FLAG_HARDWARE_ACCELERATED) != 0);
        if (mParent != null) {
            mWindow.setContainer(mParent.getWindow());
        }
        mWindowManager = mWindow.getWindowManager();
        mCurrentConfig = config;

        mWindow.setColorMode(info.colorMode);
    }

这就很清晰了, 在这初始化了 PhoneWindow 和 WindowManager, 并且关联到该 Activity, 在 ActivityThread 初始化完 Activity 后, 就调用 Activity 的 onCreate 方法了.

我们顺着 Activity 源码中的 getWindow().setContentView(layoutResID) 这行代码找到PhoneWindow 中的 setContentView, 在这个方法中, 

*PhoneWindow.setContentView*

    public void setContentView(int layoutResID) {
        if (mContentParent == null) {
            // 初始化 DecorView
            installDecor();    
        } else if (!hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
		    //如果没有过渡动画, 并且已经创建过 DecorView
            mContentParent.removeAllViews();
        }
        if (hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
		// 场景转换(过渡动画)
            final Scene newScene = Scene.getSceneForLayout(mContentParent, layoutResID, getContext());
            transitionTo(newScene);
        } else {
            // 开始填充我们设置的 XML 布局, 容器为 mContentParent
            mLayoutInflater.inflate(layoutResID, mContentParent);
        }
        mContentParent.requestApplyInsets();
        final Callback cb = getCallback();
        if (cb != null && !isDestroyed()) {
            cb.onContentChanged();
        }
        mContentParentExplicitlySet = true;
    }
    
阅读上面代码可知, mContentParent 就是装载我们所有内容的根容器(ViewGroup)了. 在这里, 最关键的代码就是 mLayoutInflater.inflate(layoutResID, mContentParent) , 它就是填充我们的布局的代码了.

installDecor 这个方法中, 初始化了 DecorView, mContentParent, 初始化了比如标题,icon, logo, 是否全屏等该 window 的一些基础属性, 这些属性我们都可以在 style 中定义, 例如 WindowNoTitle, 设置该 Activity 没有标题.

事实上,不止 Activity, 还有 Dialog, Toast 都对应着一个 Window.

## 三. PhoneWindow 中的 DecorView
---
DecorView 是承载视图的根布局, 它继承于 FrameLayout, 可以通过Window.getDecorView() 获取它

由于在 PhoneWindow.setContentView 这个方法中初始化了 DecorView, 粗略看看

*PhoneWindow.installDecor*

    private void installDecor() {
        mForceDecorInstall = false;
        if (mDecor == null) {
			// 这个方法生成了 DecorView
            mDecor = generateDecor(-1);
            mDecor.setDescendantFocusability(ViewGroup.FOCUS_AFTER_DESCENDANTS);
            mDecor.setIsRootNamespace(true);
            if (!mInvalidatePanelMenuPosted && mInvalidatePanelMenuFeatures != 0) {
                mDecor.postOnAnimation(mInvalidatePanelMenuRunnable);
            }
        } else {
            mDecor.setWindow(this);
        }
        if (mContentParent == null) {
			// 生成 Window 的容器
            mContentParent = generateLayout(mDecor);
            // Set up decor part of UI to ignore fitsSystemWindows if appropriate.
            mDecor.makeOptionalFitsSystemWindows();
            final DecorContentParent decorContentParent = (DecorContentParent) mDecor.findViewById(
                    R.id.decor_content_parent);
            if (decorContentParent != null) {
                mDecorContentParent = decorContentParent;
                mDecorContentParent.setWindowCallback(getCallback());
                if (mDecorContentParent.getTitle() == null) {
                    mDecorContentParent.setWindowTitle(mTitle);
                }
                final int localFeatures = getLocalFeatures();
                for (int i = 0; i < FEATURE_MAX; i++) {
                    if ((localFeatures & (1 << i)) != 0) {
                        mDecorContentParent.initFeature(i);
                    }
                }
            }
            ...
        }else {
            ...
        }
        ...
    }

这个方法就是用于生成 DecorView, 内容比较简单.
 
*PhoneWindow.generateDecor*

    protected DecorView generateDecor(int featureId) {
        Context context;
        if (mUseDecorContext) {
            Context applicationContext = getContext().getApplicationContext();
            if (applicationContext == null) {
                context = getContext();
            } else {
                context = new DecorContext(applicationContext, getContext().getResources());
                if (mTheme != -1) {
                    context.setTheme(mTheme);
                }
            }
        } else {
            context = getContext();
        }
		// 实例化了一个 DecorView
        return new DecorView(context, featureId, this, getAttributes());
    }

而 generateLayout 这个方法, 做的工作就稍微多一些, 

	protected ViewGroup generateLayout(DecorView decor) {
        // Apply data from current theme.
		// 获取当前 window 的 主题
        TypedArray a = getWindowStyle();
		... // 初始化样式, 例如 windowNoTitle, windowActionBar, windowIsTranslucent, windowSoftInputMode 等等
		
		mDecor.startChanging();
        mDecor.onResourcesLoaded(mLayoutInflater, layoutResource);
		... //设置 windowBackground, title, titleColor 等
		ViewGroup contentParent = (ViewGroup)findViewById(ID_ANDROID_CONTENT);
		mDecor.finishChanging();

        return contentParent;
	}
		

阅读installDecor这个方法的代码, 了解到在这个方法中生成了 DecorView, 顶级ViewGroup contentParent, 设置了 Window 标题, 设置了背景色前景色, 初始化了动画等等基础重要操作...

其中设置的很多 window 相关的属性, 我们都可以在 styles 的主题中配置, 比如我们常用的 windowActionBar 设置一个 Activity 是否隐藏 ActionBar, 还有 windowTranslucentStatus 设置透明状态栏等等.

当我们阅读到相关代码之后, 就会恍然大悟, 原来我们常用的那句神奇代码的原理是这样的, 就明白了之中的关联和逻辑, 脑袋中形成一个清晰的流程, 我们再要用到这些知识的时候就会得心应手, 这就是阅读源码的意义.

DocorView 中还有 ActionBar , 再看看他是如何成的, 我们之前分析到, Activity 在  getWindow().setContentView(view) 后紧接着  initWindowDecorActionBar().

*Activity.initWindowDecorActionBar*

	/**
     * Creates a new ActionBar, locates the inflated ActionBarView,
     * initializes the ActionBar with the view, and sets mActionBar.
     */
    private void initWindowDecorActionBar() {
        Window window = getWindow();
        // Initializing the window decor can change window feature flags.
        // Make sure that we have the correct set before performing the test below.
        window.getDecorView();
		// 判断 Activity 是否为子 Activity, 这个不用管, 是已废弃的 ActivityGroup 的遗留物
        if (isChild() || !window.hasFeature(Window.FEATURE_ACTION_BAR) || mActionBar != null) {
            return;
        }
        mActionBar = new WindowDecorActionBar(this);
        mActionBar.setDefaultDisplayHomeAsUpEnabled(mEnableDefaultActionBarUp);
        mWindow.setDefaultIcon(mActivityInfo.getIconResource());
        mWindow.setDefaultLogo(mActivityInfo.getLogoResource());
    }

注释很清楚, 创建一个新的 ActionBar, 定位已填充的 ActionBarView, 初始化并设置 ActionBar, 其中, getDecorView 方法是为了保证 DecorView 已创建, 之前也追踪过相关代码.

*WindowDecorActionBar 的构造方法*

    public WindowDecorActionBar(Activity activity) {
        mActivity = activity;
        Window window = activity.getWindow();
        View decor = window.getDecorView();
        boolean overlayMode = mActivity.getWindow().hasFeature(Window.FEATURE_ACTION_BAR_OVERLAY);
        init(decor);
        if (!overlayMode) {
            mContentView = decor.findViewById(android.R.id.content);
        }
    }

### DecorView

















    
