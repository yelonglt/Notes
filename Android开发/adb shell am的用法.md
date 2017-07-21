##am命令
am全称activity manager，你能使用am去模拟各种系统的行为，例如启动一个activity，service，发送广播进程，强制停止进程，修改屏幕属性等。

###启动activity
adb shell am start [-D][-W] intent

1. -D：enable debugging
2. -W: wait for launch to complete
3. intent启动参数 -n(类名) com.wm.dmall/com.wm.dmall.LaunchActivity

例如：
查看应用的启动时间：adb shell am start -W -n com.wm.dmall/com.wm.dmall.LaunchActivity

打开浏览器：adb shell am start -a android.intent.action.VIEW

###启动service
adb shell am startservice intent

###启动broadcast
adb shell am broadcast intent

例如：
模拟发送手机低电环境：adb shell am broadcast -a Android.intent.action.BATTERY_CHANGED --ei "level" 3 --ei "scale" 100

###杀死进程

1. 强制停止指定的package应用：adb shell am force-stop com.wm.dmall
2. 杀死所有的后台进程：adb shell am kill-all