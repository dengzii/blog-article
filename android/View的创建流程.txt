
在 Android App中, 所有的数据内容都是通过 View 展示给用户的, Android 通过一系列机制和流程将这些承载着各种交互控件和展示数据的 View 展示出来. 
在开发中, 我们也经常需要用到自定义 view, 因此, 我们非常有必要学习一下 view 的创建流程, 
本文从源码出发, 循序渐进介绍了了 Activity, PhoneWindow, DecorView, View 和 ViewGroup 的关系, 是如何一步步绘制最终展示出来的, 重点介绍了 View 的 measure, layout, draw 这几个方法. 

## 一. Activity

注意, AppCompatActivity 是有所区别的.

通常一个 App 是有许多 Activity 组成的， 我们在创建一个 Activity 的时候, 通常首先就是重写 onCraete 方法并调用 setContentView 传入相应的布局资源, 用于加载相应的 XML 布局文件,
 如果没有设置, 则该 Activity 就只有一个 ActionBar 和一个背景色(一般为白色), 我们看到的内容就是 Window 中的 DecorView， Window 是 Activity 的顶层容器， 其次是 DecorView, 而 DecorView，
 中则包含了 ActionBar, 和我们的 XML 布局了, 他们的结构大致如下图所示

        Android UI 层级
    
    |--------------------|
    |	Activity         |
    |--------------------|
    |	PhoneWindow      |
    |--------------------|
    |	DecorView        |
    |--------------------|
    |	| ActionBar |    |
    |	|-----------|    |
    |	|ContentView|    |
    |	                 |
    |--------------------|
    

Activity 的 setContentView 方法：

	public void setContentView(@LayoutRes int layoutResID) {
        getWindow().setContentView(layoutResID);
        initWindowDecorActionBar();
    }

首先, 它调用了 Window.setContentView, 这个 Window 就是 PhoneWindow 了, Activity 中我们看到的 View 都封装在这个类中.

接着初始化 ActionBar 

	/**
     * Creates a new ActionBar, locates the inflated ActionBarView,
     * initializes the ActionBar with the view, and sets mActionBar.
     */
    private void initWindowDecorActionBar() {
        Window window = getWindow();
        // Initializing the window decor can change window feature flags.
        // Make sure that we have the correct set before performing the test below.
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

PhoneWinow 的 getDecorView 方法：

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

> Window 代表着一个抽象窗口, PhoneWindow 是 Window 的具体实现, 这个类位于 com.android.internal 包下, 这个包中的类我们的 SDK 是没有的, 所以就算你下载了相应的 SDK 源码也看不到 PhoneWindow 的代码.

https://github.com/anggrayudi/android-hidden-api, 下载这个仓库中对应版本的.android.jar 替换 sdk\platforms\android-{version}\android.jar, 你就可以看到了.

我们顺着 Activity 源码中的 getWindow().setContentView(layoutResID) 这行代码找到PhoneWindow 中的 setContentView, 在这个方法中, 

    public void setContentView(int layoutResID) {
        if (mContentParent == null) {
			// 初始化 DecorView 和 ViewGroup
            installDecor();	
        } else if (!hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
            mContentParent.removeAllViews();
        }
        if (hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
            final Scene newScene = Scene.getSceneForLayout(mContentParent, layoutResID, getContext());
            transitionTo(newScene);
        } else {
			// 开始填充我们设置的 XML 布局
            mLayoutInflater.inflate(layoutResID, mContentParent);
        }
        mContentParent.requestApplyInsets();
        final Callback cb = getCallback();
        if (cb != null && !isDestroyed()) {
            cb.onContentChanged();
        }
        mContentParentExplicitlySet = true;
    }
	
阅读上面代码可知, mContentParent 就是装载我们所有内容的根容器(ViewGroup)了.

installDecor 这个方法中, 初始化了 DecorView, mContentParent, 初始化了比如标题,icon, logo, 是否全屏等该 window 的一些基础属性, 这些属性我们都可以在 style 中定义, 例如 WindowNoTitle, 设置该 Activity 没有标题.

事实上, Activity, Dialog, Toast, PopupWindow 都对应着一个 Window.

我们再来看看 Window 是如何创建的, 下面是 ActivityThread 中的 performLaunchActivity 方法

	private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
		...
		ContextImpl appContext = createBaseContextForActivity(r);
        Activity activity = null;
        try {
            java.lang.ClassLoader cl = appContext.getClassLoader();
            activity = mInstrumentation.newActivity(
                    cl, component.getClassName(), r.intent);
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
                    window = r.mPendingRemoveWindow;
                    r.mPendingRemoveWindow = null;
                    r.mPendingRemoveWindowManager = null;
                }
                appContext.setOuterContext(activity);
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

Activity.attach

	final void attach(Context context, ActivityThread aThread,
            Instrumentation instr, IBinder token, int ident,
            Application application, Intent intent, ActivityInfo info,
            CharSequence title, Activity parent, String id,
            NonConfigurationInstances lastNonConfigurationInstances,
            Configuration config, String referrer, IVoiceInteractor voiceInteractor,
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

## 三. DecorView 和 ViewRootImpl

ViewRootImpl 是 用户界面, 视图, View 层级的最顶部. 是让 View 和 WindowManager 沟通的桥梁, 

由于在 Activity 的 setContentView 方法中调用了 getWindow.setContentView, 我们看看 PhoneWindow 的 setContentView 方法, 这个方法中初始化了 DecorView

	public void setContentView(int layoutResID) {
		if (mContentParent == null) {
			installDecor();
		} else if (!hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
			mContentParent.removeAllViews();
		}

		if (hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
			final Scene newScene = Scene.getSceneForLayout(mContentParent, layoutResID,
					getContext());
			transitionTo(newScene);
		} else {
			mLayoutInflater.inflate(layoutResID, mContentParent);
		}
		mContentParent.requestApplyInsets();
		final Callback cb = getCallback();
		if (cb != null && !isDestroyed()) {
			cb.onContentChanged();
		}
		mContentParentExplicitlySet = true;
	}

首先是判断 mContentParent 是否为空, 这个 mContentParent 就是我们的 DecorView 了, 当他为空的时候, 就初始化了, 


























	
