# PopupWindow显示和消失动画，以及界面切换时动画所导致的问题

标签（空格分隔）： Android

---

Android开发的过程中如果遇到菜单的情况，除了使用`Dialog`，有时我们也会选择使用`PopupWindow`来实现，但是`PopupWindow`的显示和消失的动画太生硬，所以可以给它加入显示和消失时的动画效果。下面以底部菜单的效果为例。

首先，在`res`下的`anim`文件夹（没有就自己新建一个）中创建两个动画文件，一个是从底部向上平移显示，另一个是从底部向下平移消失。

底部向上平移显示：
```xml
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <translate android:fromYDelta="100%p" android:toYDelta="0" android:duration="500"/>
    <alpha android:fromAlpha="0" android:toAlpha="1.0" android:duration="500"/>
</set>
```

底部向下平移消失：
```xml
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <translate android:fromYDelta="0" android:toYDelta="100%p" android:duration="500"/>
    <alpha android:fromAlpha="1.0" android:toAlpha="0" android:duration="500"/>
</set>
```

然后在`styles.xml`文件中创建一个动画style：
```xml
<style name="PopupWindow">
        <item name="android:windowEnterAnimation">@anim/popup_menu_enter_from_bottom</item>
        <item name="android:windowExitAnimation">@anim/popup_menu_exit_to_bottom</item>
</style>
```

最后在创建`PopupWindow`时指定该动画：
```java
mPopupWindow.setAnimationStyle(R.style.PopupWindowFilter)
```

这样`PopupWindow`在show和dismiss时，就会有过渡的动画效果了。网上的很多教程到这里就结束了，但是真正坑的是在后面啊，因为这是`Window`切换的动画，所以如果遇到需要切换界面，但是`PopupWindow`暂时不需要dismiss的情况就懵逼了，因为在你切到另一个界面之前，`PopupWindow`会执行退出动画，再次切回来，`PopupWindow`又会再次执行一次显示的动画。我相信这个现象不是大家所希望看到的。

由于我正在开发的APP里大量的弹出菜单（吐槽一下设计），以前的做法都是在切换界面的时候手动把`PopupWindow`dismiss掉，但是现在需要新开发一个摇一摇截屏反馈的问题（再次吐槽设计。而且，目前还在被截屏问题困扰中，因为没有找到好的不需要root和系统权限并且能兼容5.0以下系统的截屏方案，`getDrawCache()`的方法无法截取`Dialog`和`PopupWindow`，所以舍弃），截取到的屏幕截图要新开一个界面显示并提供涂鸦功能。在弹出菜单未消失的时候截图会有界面切换的行为，此时返回到上一个界面`PopupWindow`会不存在了（已经手动dismiss掉），本来觉得没什么问题，但是测试同学不通过。所以只好上网寻找方法，未果，发帖求助，未果。所以自己看了一下`PopupWindow`的方法，发现一个解决方案。虽然方案确实很简单，但是也确实遇到这个不能忽视的问题，所以还是记录一下。

解决方案就是在`Activity` `onPause`的时候，手动取消`PopupWindow`的动画：
```java
    @Override
    public void onPause() {
        super.onPause();
        if (mPopupWindow != null && mPopupWindow.isShowing()) {
            mPopupWindow.setAnimationStyle(0);
            mPopupWindow.update()
        }
    }
```

但是切换回来，我们应该再次把动画效果加上：
```java
    @Override
    public void onResume() {
        super.onResume();
        if (mPopupWindow != null) {
            mHandler.postDelayed(new Runnable() {
                @Override
                public void run() {
                    mPopupWindow.setAnimationStyle(R.style.PopupWindowFilter);
                    mPopupWindow.update();
                }
            }, 200);
        }
    }
```
`onResume()`方法里之所以要延时200ms操作，是因为要在`onResume`行为结束后再将动画加上，否则会因为太早导致切换回来`PopupWindow`还会再次执行显示的动画。

整体来说非常的简单，但是再简单也得做不是，所以记录一下！


