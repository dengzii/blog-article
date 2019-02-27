通知栏通知在Android APP中的使用极为频繁，比如短信通知，QQ，微信消息通知，App 更新进度转态显示，截图时后图片进行删除或分享，查看操作等等。本篇文章记录了如何使用 Notification 显示消息, 设置提示音，呼吸灯，震动，以及响应用户对消息的处理动作。

# Notification 状态栏通知

- 建造者模式构建通知类： Notification.Builder
- 通知管理器(用于发出通知)： NotificationManager
- 通知通道(API 26新增，用户可以选择性屏蔽通知)：NotificationChannel 
- 通知动作(用户点击滑动通知等)：PaddingIntent

**发送通知的步骤**

1. 获取 NotificationManager
2. 创建 Notification
3. 给 Notification 设置参数
4. 使用 NotificationManager 发送通知

**Android 7.0 新内容**

Notification.Builder.serPriority: 设置通知优先级
Notification.Builder.setStyle: 设置通知扩展布局
MessagingStyle: 快速回复

**Android 8.0 新内容**

NotificationChannel：用户可以自定义通知的显示，以及关闭某个通知的提示音震动等

# 构建一个最简单的 Notification

一个简单的通知包含，一个小图标，一个标题和内容如图：

[暂无图片]

代码
    
    // 获取系统 通知管理 服务
    NotificationManager notificationManager = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
    
    // 构建 Notification
    Notification.Builder builder = new Notification.Builder(this);
    builder.setContentTitle("ContentTitle")
            .setSmallIcon(R.drawable.ic_launcher_background)
            .setContentText("Content Text Here");
    
    // 兼容  API 26，Android 8.0
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O){
        // 第三个参数表示通知的重要程度，默认则只在通知栏闪烁一下
        NotificationChannel notificationChannel = new NotificationChannel("AppTestNotificationId", "AppTestNotificationName", NotificationManager.IMPORTANCE_DEFAULT);
        // 注册通道，注册后除非卸载再安装否则不改变
        notificationManager.createNotificationChannel(notificationChannel);
        builder.setChannelId("AppTestNotificationId");
    }
    // 发出通知
    notificationManager.notify(1, builder.build());

# 控制 Notification 的震动，呼吸灯、提示音以及如何显示

构建 Notification 代码
   
    // 构建 Notification
    Notification.Builder builder = new Notification.Builder(this);
    builder.setContentTitle("ContentTitle")
            .setSmallIcon(R.drawable.ic_launcher_background)
            .setContentText("Content Text Here")
            .setDefaults(Notification.DEFAULT_ALL);
            
这里关键的在于 SetDefaults 这个方法，它接收一个 int，可选值如下：

- Notification.DEFAULT_VIBRATE ：震动提示
- Notification.DEFAULT_SOUND：提示音
- Notification.DEFAULT_LIGHTS：三色灯
- Notification.DEFAULT_ALL：以上三种全部

## 控制震动时长

setVibrate 接收一个long[], 下标为奇数为延时，偶数为震动时长，本例中为，延时0ms,震动300ms,延时200ms,震动300

    builder.setVibrate(new long[]{0, 300, 200, 300});
    // Android 8.0 需用以下方法
    // notificationChannel.enableVibration(true);
    // notificationChannel.setVibrationPattern(new long[]{0,300,300,500,300,800});
    
## 控制灯颜色
    
三个参数依次是 int ARGB 颜色值、亮的时长、不亮的时长。
    
    builder.setLights(0xFFFF0000, 1000,1000);
    // Android 8.0 需用以下方法，不可设置时长
    // notificationChannel.enableLights(true);
    // notificationChannel.setLightColor(0xFFFF0000);

## 控制声音
    
    builder.setSound(Uri.parse("android.resource://" + getPackageName() + "/" + R.raw.a))
    // Android 8.0 需用以下方法， 这里用的是加载res/raw的声音资源
    // notificationChannel.setSound(Uri.parse("android.resource://" + getPackageName() + "/" + R.raw.a), null);
    
## 在锁屏上的显示方式
    
这个方法需要使用锁屏并在设置中隐藏敏感信息才能生效

    builder.setVisibility(Notification.VISIBILITY_PUBLIC);
    // Android 8.0 需用一下方法
    // notificationChannel.setLockscreenVisibility(Notification.VISIBILITY_PUBLIC);
    
这个方法接收一个 int

* Notification.VISIBILITY_PUBLIC 显示所有通知内容
- Notification.VISIBILITY_PRIVATE 只显示标题
- Notification.VISIBILITY_SECRET 不显示任何内容

## 如何取消通知

1. 用户清除
1. setAutoCancel()，点击通知会清除
1. NotificationManager.cancel(int id)
1. NotificationManager.cancel(String tag, int id)
1. NotificationManager.cancelAll() 清除所有该应用的通知

## 其他

**持续的通知**

如播放音乐，后台任务，下载, 当这个方法传入 true 时，表示它是一个持续的通知，用户无法删除它，只能在代码中让通知消失。

    builder.setOngoing(true);

**显示进度条**

在后台处理某个耗时任务时需要使用到进度条, 参数作用依次是 进度条最大值、当前进度、进度是否确定，indeterminate 表示任务的进度是否可以准确获取

    builder.setProgress(int max, int progress, boolean indeterminate);

**如何更新通知**

只需在发出通知时使用相同的 id 即可更新，如果这条通知已被移除则创建一个新的通知

    notificationManager.notify(int id, Notification.Builder);

# 如何响应用户动作

## 给 Notification 设置 PendingIntent 用于响应点击、清除动作

PendingIntent 是一种特殊的 Intent， 作用和 Intent 一样是用于启动一个 Activity或者Service，或发送一条 Broadcast。通知响应用户动作便是用这个， 当对通知做出一个动作后，系统便会调用 PendingIntent，启动一个活动，服务或广播，这取决于你获取的是那种 PendingIntent。

在 PendingIntent 中传入的 Context 销毁以后，PendingIntent 依旧有效，它一般使用在当 Context 销毁后需要执行 Intent的地方，一般不是用于立即执行的时候，比如在点击通知后唤醒一个 Activity。。

获取 PendingIntent 

    // 获取用于启动 Activity 
    PendingIntent.getActivity(Context context, int requestCode, Intent intent, int flags);
    // 获取用于发送 Broadcast
    PendingIntent.getBroadcast(Context context, int requestCode, Intent intent, int flags);
    // 获取用于启动服务
    PendingIntent.getService(Context context, int requestCode, Intent intent, int flags);
    PendingIntent.getActivities(Context context, int reqeustCode, Intent[] intents, int flags);
    PendingIntent.getForgroundService(Context, int reqeustCode, Intent intent, int flags)

各个参数的用途分别如下

- context, 当前上下文
- requestCode, 请求标识，如果多次获取时这个值相同，则返回的结果与 flags 参数相关
- intent, 请求意图
- flags, 控制获取 PendingIntent 的方式，可选值如下
    - PendingIntent.FLAG_CANCEL_CURRENT，如果当前已存在则取消当前的并返回一个新的 PendingIntent
    - PendingIntent.FLAG_UPDATE_CURRENT，如果已存在则更新之前的
    - PendingIntent.FLAG_NO_CREATE，如果已存在则返回当前存在的，否则返回 null
    - PendingIntent.FLAG_ONE_SHOT，表明这个 PendingIntent 只能用一次，触发一次后自动 cancel
    - PendingIntent.FLAG_IMMUTABLE，表明这个PendingIntent 不可变

其中，requestCode 和 flags 是相关联的，如果多次获取 PendingIntent 时 requestCode 相同，此时返回的结果就需要参考 flags 的值。例如

        Intent intent1 = new Intent("NotificationAction");
        intent1.putExtra("A", "value AA");
        intent1.putExtra("B", "value BB");

        Intent intent11 = new Intent("NotificationAction");
        intent11.putExtra("B", "BBB");

        PendingIntent pendingIntent = PendingIntent.getBroadcast(this, 1, intent1, PendingIntent.FLAG_CANCEL_CURRENT);
        PendingIntent pendingIntent11 = PendingIntent.getBroadcast(this, 1, intent11, PendingIntent.FLAG_UPDATE_CURRENT);
  
这两个 PendingIntent 的 requestCode 是相同的，第二个获取时传入的 flags 为 PendingIntent.FLAG_UPDATE_CURRENT ，表示存在 requestCode 为 1，的PendingIntent则更新这个PendingIntent 的 Intent 值，此时 getStringExtr("A") 的值为 null，getStringExtra("B"), 的值为 BBB。

当我们获取了 PendingIntent 之后，只需要给 builder 的特定方法传入就可以响应点击、清除动作了

设置点击，清除 PendingIntent 代码：
    
    builder.setContentIntent(PendingIntent intent);
    builder.setDeleteIntent(PendingIntent intent);

## 添加几个简单的按钮

在需要对通知内容进行简单操作的时候，比如短信的‘标记为已读’，‘删除’。我们可以对按钮设置图标、标题以及设置 PendingIntent 以便响应动作。这种按钮最多可以添加三个，按添加的顺序从左往右依次排列。
**api 23 之后图标不显示**

    Intent intent2 = new Intent("NotificationAction");
    intent2.putExtra("A", "Action button1 clicked");
    PendingIntent pendingIntent1 = PendingIntent.getBroadcast(this, 3, intent2, PendingIntent.FLAG_UPDATE_CURRENT);
    otification.Action.Builder actionBuilder = new Notification.Action.Builder(Icon.createWithResource(getApplicationContext(), R.drawable.ic_launcher_background), "Action1", pendingIntent1);

    builder.addAction(actionBuilder.build());

结果如图，点击 Action1 按钮后将发出一条广播

[暂无图片]

# 超长内容的 Notification, 各种 Style
    
超长内容的 Notification，比如内容为文字很多，或者一个大图片，一个自定义的 View，通过 builder.setStyle(Style style) 这个方法设置。设置 style 之后，之前的 setContent 设置的内容将会失效。各种 style 的高度最大为 256 dp。

## BigTextStyle，超长文本

    Notification.BigTextStyle bigTextStyle = new Notification.BigTextStyle().bigText(
                "This is big text notification. This is big text notification. " +
                "This is big text notification. This is big text notification. " +
                "This is big text notification. This is big text notification. " +
                "This is big text notification. This is big text notification. " +
                "This is big text notification. This is big text notification. " +
                "This is big text notification. This is big text notification. " +
                "This is big text notification. This is big text notification. " +
                "This is big text notification. This is big text notification. " +
                "This is big text notification. This is big text notification. ");
    builder.setStyle(bigTextStyle);

## InboxStyle，列表

inboxstyle 显示为一个文字列表，显示 addline 的顺序的文字，最多支持六行。

    Notification.InboxStyle inboxStyle = new Notification.InboxStyle();
    // 添加行
    inboxStyle.addLine("First line.");
    inboxStyle.addLine("Second line");
    inboxStyle.addLine("Third line");
    inboxStyle.addLine("Last line");
    
    // 设置标题以及简介文字
    inboxStyle.setBigContentTitle("ContentTitle");
    inboxStyle.setSummaryText("SummaryText");

    builder.setStyle(inboxStyle);

## BigPictureStyle, 大图片

    // 构建一个 Style
    Notification.BigPictureStyle bigPictureStyle = 
        new Notification.BigPictureStyle().bigPicture(BitmapFactory.decodeResource(getResources(),R.drawable.big));
    // 设置标题
    bigPictureStyle.setBigContentTitle("ContentTitle");
    // 设置副标题，简介文字
    bigPictureStyle.setSummaryText("SummaryText");
    // 设置大图标
    bigPictureStyle.bigLargeIcon(BitmapFactory.decodeResource(getResources(), R.drawable.big));
    
    builder.setStyle(bigPictureStyle);

## MediaStyle

略

# 快速回复 MessageNotification

有些时候我们需要对通知内容进行快速回复，比如收到一条邮件，我们可以快速回复。当点击通知的一个 Action 按钮后将会出现一个输入框，可以进行发送消息。我们需要添加一个 MessageStyle 和 一个带 RemoteInput 的 Action，这个通知便能支持快速回复。**这个功能需要 api 版本大于 24**

首先，构建一个 MessageStyle，并设置好

    Notification.MessagingStyle messagingStyle = new Notification
        .MessagingStyle("UserDisplayName").
        addMessage("MessageStyle", 1000, "sender");
    builder.setStyle(messagingStyle);

然后再创建一个 Action，Action 需添加一个 RemoteInput，一个响应发送动作的 PendingIntent，这个例子中点击发送按钮后将发送一条广播。

    Intent intent = new Intent("NotificationAction");
    PendingIntent pendingIntent = PendingIntent.getBroadcast(this, 11, intent, PendingIntent.FLAG_UPDATE_CURRENT);

    RemoteInput remoteInput = new RemoteInput
        .Builder("RemoteInputKey")
        .setLabel("RemoteInputLabel")
        .build();

    Notification.Action action = new Notification.Action
        .Builder(Icon.createWithResource(this, R.mipmap.ic_launcher_round), "ReplayAction", pendingIntent)
        .addRemoteInput(remoteInput)
        .build();
    // 发送该通知
    notificationManager.notify(22, builder.build());
    
接受输入数据用的广播，我们在获取到输入数据后需要手动取消该通知，否则不会自动取消。

    class MBroadcast extends Broadcast{
        @Override
        public void onReceive(Context context, Intent intent) {
            Bundle bundle = RemoteInput.getResultsFromIntent(intent);
            if (bundle != null){
                String replayContent = bundle.getString("RemoteInputKey");
                Log.d("MBroadcast", "onReceive: " + replayContent);
                // 取消该通知
                ((NotificationManager)getSystemService(Context.NOTIFICATION_SERVICE)).cancel(22);
            }    
        }
    }

# 给 Notification 添加一个 View 扩展视图

在通知栏进行对后台任务的控制的时候需要用到这个功能， 比如下载一个文件时可以 取消，暂停，后台播放音乐时可以 切换暂停和显示歌词等。通知的自定义 view 最大高度为 256 dp，且只有 **Android N 以上版本才支持** 自定义 view。

[暂无图片]
 
RemoteViews 只支持几个 View，在这里我使用 ConstraintLayout 作为根 Layout 的时候发现会出现 android.app.RemoteServiceException: Bad notification posted from package xxx 这个错误，换了 LinearLayout 后就不会出现这个问题。支持的 View 如下

- Layout
    - FrameLayout
    - LinearLayout
    - RelativeLayout
    - GridLayout
- View
    Button，ImageView，ImageButton，TextView，ProgressBar，ListView，GridView，StackView，ViewStub，AdapterViewFlipper，ViewFlipper，AnalogClock，Chronometer
    
首先构建一个通知，创建一个 RemoteViews，RemoteViews 构造函数接受一个包含 view 所在的包名和一个 View 资源。RemoteViews 意思就是远程 view，该 view 不存在与 当前 Activity，进程，通常用于通知中的自定义 view 和桌面小部件。

    Notification.Builder builder = new Notification.Builder(this)
        .setContentTitle("ContentTitle")
        .setTicker("ThisIsTicker")
        .setSmallIcon(R.drawable.ic_launcher_background)
        .setContentText("ContentText");
    
    RemoteViews remoteViews = new RemoteViews(getPackageName(), R.layout.notify_view);

这里的 notify_view 很简单，如下

    <LinearLayout ...>
        
        <ImageView id="@+id/notify_iv"
            .../>
        <TextView id="@+id/notify_tv"
            .../>
        <Button id="@+id/notify_bt"
            .../>
    </LinearLayout>

设置 CustomContentView 和 Style。

    builder.setCustomContentView(remoteViews);
    Notification.DecoratedCustomViewStyle viewStyle = new Notification.DecoratedCustomViewStyle();
    builder.setStyle(viewStyle);

## 如何监听自定义 view RemoteViews 中的点击事件，和更新 view

现在已经添加了一个 view，如何修改 view 中的数据以及响应 view 的事件呢？通过 RemoteViews.setOnClickPendingIntent(int r, PendingIntent intnet) 这个方法为 一个 view 绑定一个 PendingIntent，当点击了该view 后便执行相应的操作。

    Intent intent = new Intent("NotificationAction");
    intent.putExtra("action", "bt_click");
    
    PendingIntent pendingIntent = PendingIntent.getBroadcast(this, 12, intent, PendingIntent.FLAG_UPDATE_CURRENT);
    remoteViews.setOnClickPendingIntent(R.id.notify_bt, pendingIntent);

更新 RemoteView 中的数据，更新一个 ImageView中的图片，对各种 view 更新都有相应的方法。
    
    remoteViews.setBitmap(R.id.notify_iv, "setImageBitmap", BitmapFactory.decodeResource(getResources(), R.drawable.me));

（完）
个人博客链接 https://djh.red/blog/article/18/