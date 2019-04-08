# View 的事件分发

标签（空格分隔）：Android View

---

事件分发可以说是 Android 中非常非常重要的知识了, 如果作为一个 Android 开发人员没有了解过事件分发, 那他就不能是一个合格的 Android 程序员, 作为一个
中高级 Android 开发工程师，是必须要深入了解的。学习事件分发主要解决的问题是多层 ViewGroup ，View 嵌套事件冲突, 这篇文章主要分析的是手指的事件, 而
对于其他例如鼠标, 触控笔, 等的事件, 是有略微不同的.

## 什么是事件分发 

在用户界面中, 许多 View 和 ViewGroup 都有需要处理的事件, ViewGroup 中有 View 和 ViewGroup, 这样不断包裹形成层级, 互相重叠. 用户
在对屏幕进行某些特定的动作时, ViewGroup 按一定的顺序获取该动作, 并且可以选择向子 ViewGroup 传递, 或者拦截该动作事件, View 则可以响应, 消费该动作事件.

几个典型涉及事件分发例子, 比如点击一个菜单选项, 或滑动一个列表, 然而列表里可能还有其他列表, 列表外部可能是一个内容很多的容器,该容器也需要滚动, 
如何处理滚动的优先级呢? 再比如滑动一个列表而不是点击滑动时手指触动的那个 Button. 这就涉及事件分发了. 

## 事件分发的主角

事件分发的主角即是用户界面 UI, UI 由 View 和 ViewGroup ( ViewGroup 也是 View 的子类) 和 Activity.

### View 和 ViewGroup:

1. 继承于 View 并且实现了 ViewParent, ViewManager 这两个接口的 ViewGroup 及其子类. 例如常用的 LinearLayout, FrameLayout 等都是 ViewGroup 的子类.
2. 直接或间接继承于 View 为普通控件, 例如 Button, ImageView, TextView. 
 
### 另一个主角 Activity
 
在事件分发中, 还有一个主角, 即是 Activity, 首先接收到事件的是 Activity, 由 Activity 交给 Window 处理, 再将事件分发给 decorView, 最后交给我们
的布局了, decorView 为我们 setContentView 的容器, Activity 的根布局, 我们可以通过 Activity.getWindow().getDecorView() 获取.

## 几个基本概念

### MotionEvent

事件分发中，分发的事件就是这个类的实例，这个类用于描述动作事件，保存着移动事件的数据，比如动作，坐标等.
用户每次从手指接触 View 或 ViewGroup 到手指离开这段时间称为一次动作事件，还有非正常情况的 ACANCEL，用户还没抬起手指这个事件被中断时出发，
例如，在一个 RecyclerView 中有一个 Button， 按下这个 Button 的同时往下滑动，Button 的 onTouchEvent 会触发 ACTION_CANCEL. 

MotionEvent 的几种动作(Action)类型:

- MotionEvent.ACTION_DOWN : 按下时触发
- MotionEvent.ACTION_MOVE : 移动
- MotionEvent.ACTION_UP   : 离开屏幕
- MotionEvent.ACTION_CANCEL: 非正常取消
- MotionEvent.ACTION_OUTSIDE: 移动到 View 外 

### 事件序列
	
用户手指按下屏幕到离开发生的所有事件为一个事件序列, 以 ACTION_DOWN 开始, 以 ACTION_UP 结束, 中间可能还有若干 ACTION_MOVE 事件.
	
### dispatchTouchEvent(MotionEvent event)

Activity , View, ViewGroup 中都有这个方法, View 除 ViewGroup 及其子类外的子类没有这个方法, 这个方法负责派发事件, 返回结果为是否消费该事件，
如果事件能够传递给*当前* ViewGroup, 则一定会被调用, 这个方法中会调用下级 View, 或 ViewGroup 的 onTouchEvent, dispatchTouchEvent, 
并将下级返回的结果向上返回.

### onInterceptTouchEvent

只有 ViewGroup 有这个方法，在 dispatchTouchEvent 中会调用这个方法, 返回 true 则表示拦截该事件, 不会向下传递事件. 

### onTouchEvent(MotionEvent event)

所有事件分发的参与者都有该方法, 用于处理动作事件, 这个方法会在 dispatchTouchEvent 中调用, 返回结果表示是否消耗当前事件, 如果不消耗, 则在同一个
事件序列中, 这个 View 不会再次收到事件.

## 分发顺序及规则

### 典型情况的顺序

我们假设所有 ViewGroup 的 onInterceptTouchEvent 返回都为 false, 即不拦截改事件, 使改事件继续往下分发.

dispatchTouchEvent 的顺序:

	Activity  ->	Window(PhoneWindow)	-> DecorView	->	ViewGroup	->	View

所有情况下, Activity 总是第一时间获取到事件, 然后往下分发, 在 View, ViewGroup 中即不拦截(即 onInterceptTouchEvent 返回 false), 也不消费( onTouchEvent 返回false)
的情况下, 事件将通过 onTouchEvent 往回传递, 直到 Activity.
这里省略了 Window 和 DecorView:

	Activity ---dispatchTouchEvent--->	ViewGroup ---dispatchTouchEvent---onInterceptTouchEvent--->	View	
																									  |
										 Activity  <---onTouchEvent---  ViewGroup <---onTouchEvent---

*ViewGroup 中的 dispatchTouchEvent:*

拦截部分代码片段:

	...
	final int action = ev.getAction();
	final int actionMasked = action & MotionEvent.ACTION_MASK;
	// Handle an initial down.
	if (actionMasked == MotionEvent.ACTION_DOWN) {
		cancelAndClearTouchTargets(ev);
		resetTouchState();
	}
	// Check for interception.
	final boolean intercepted;
	if (actionMasked == MotionEvent.ACTION_DOWN
			|| mFirstTouchTarget != null) {
		final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
		if (!disallowIntercept) {
			intercepted = onInterceptTouchEvent(ev);
			ev.setAction(action); // restore action in case it was changed
		} else {
			intercepted = false;
		}
	} else {
		// There are no touch targets and this action is not an initial down
		// so this view group continues to intercept touches.
		intercepted = true;
	}
	...
	
从上代码片段中可知, 首先会进行手势检测, 如果在一个事件序列发生的中途出现手势检测, 则在这之前的事件都会被重置和取消, 然后, 将会判断是否拦截该事件,
mFirstTouchTarget 为下一个需要派发事件的 View, 如果他为空, 表示事件不交给子 View 处理, 则 intercepted.

继续, 往下分发事件的代码 ...

	if (!canceled && !intercepted) {
		...
		if (actionMasked == MotionEvent.ACTION_DOWN
						|| (split && actionMasked == MotionEvent.ACTION_POINTER_DOWN)
						|| actionMasked == MotionEvent.ACTION_HOVER_MOVE) {
			...
			if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {
				// Child wants to receive touch within its bounds.
				mLastTouchDownTime = ev.getDownTime();
				if (preorderedList != null) {
					// childIndex points into presorted list, find original index
					for (int j = 0; j < childrenCount; j++) {
						if (children[childIndex] == mChildren[j]) {
							mLastTouchDownIndex = j;
							break;
						}
					}
				} else {
					mLastTouchDownIndex = childIndex;
				}
				mLastTouchDownX = ev.getX();
				mLastTouchDownY = ev.getY();
				newTouchTarget = addTouchTarget(child, idBitsToAssign);
				alreadyDispatchedToNewTouchTarget = true;
				break;
			}
			...
		}
	}
	 // Dispatch to touch targets.
	if (mFirstTouchTarget == null) {
		// No touch targets so treat this as an ordinary view.
		handled = dispatchTransformedTouchEvent(ev, canceled, null,
				TouchTarget.ALL_POINTER_IDS);
	} else {
		// 此处为遍历 View 分发事件的代码
	}
	...
	
当事件没有被取消, 也没有被拦截, 并且事件为 DOWN, 则继续往子 View 派发.

ViewGroup 中向子 View 分发事件主要是在 dispatchTransformedTouchEvent 中进行的, 在这个方法中会调用子 view 的 dispatchTouchEvent.

### 常见顺序概要
										 
只要上层 ViewGroup 将事件分发下来的没有被*拦截*, 则 dispatchTouchEvent 一定会被调用, 在 dispatchTouchEvent 中, 首先会调用 onInterceptTouchEvent 
判断是否拦截该事件, 如果拦截, 则不再往下分发, 这个事件就交给这个 ViewGroup处理, 执行 onTouchEvent, 那么它的下级将无法收到任何事件. 否则继续往下分发, 
分发到最底层的 ViewGroup,和 View 时, onTouchEvent 将从最底层依次往上层被调用, 如果其中有某个 View 或 ViewGroup 的 onTouchEvent 方法返回了 true, 
则该 View 将消费该事件, 更上层的 View 和 ViewGroup 将无法接收到该事件, 并且, 消费了该事件的 View 或 ViewGroup 的子 View 将无法收到该事件序列 
ACTION_DOWN 后剩余的事件.

### Activity 中的 dispatchTouchEvent

以下是 Activity 中的分发, getWindow().superDispatchTouchEvent(ev), 由此可见, Activity 将 事件分发给 Window.

    public boolean dispatchTouchEvent(MotionEvent ev) {
        if (ev.getAction() == MotionEvent.ACTION_DOWN) {
            onUserInteraction();
        }
        if (getWindow().superDispatchTouchEvent(ev)) {
            return true;
        }
        return onTouchEvent(ev);
    }

### 设置了 onTouchListener 或 onClickListener

onTouchListener 将会在 onTouchEvent 前调用, 如果 onTouchListener 中的 onTouch 返回了 true, 则 onTouchEvent 不会调用.

在 View 的 dispatchTouchEvent 中, 我们可以找到以下片段, li.mOnTouchListener.onTouch(this, event) 返回 true, 则 result 赋值 true, onTouchEvent 将不会执行:

	public boolean dispatchTouchEvent(MotionEvent event) {
		...
		if (onFilterTouchEventForSecurity(event)) {
			if ((mViewFlags & ENABLED_MASK) == ENABLED && handleScrollBarDragging(event)) {
				result = true;
			}
			//noinspection SimplifiableIfStatement
			ListenerInfo li = mListenerInfo;
			if (li != null && li.mOnTouchListener != null
					&& (mViewFlags & ENABLED_MASK) == ENABLED
					&& li.mOnTouchListener.onTouch(this, event)) {
				result = true;	
			}
			if (!result && onTouchEvent(event)) {
				result = true;
			}
		}
		...
	}
	
而 onClickListener 的 onClick 方法则会在 onTouchEvent 调用, 可见 onClickListener 的优先级低于 onTouchListener.
	
### 其他情况

多个 View 重叠, 比如在 FrameLayout 中, 则在布局文件最下面的 View 优先级比较高.

## 总结和其他

- 只有在 clickable 和 longClickable 都为 false 的情况, View 才无法接收事件

(完)