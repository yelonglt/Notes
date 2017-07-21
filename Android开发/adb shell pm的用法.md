##pm命令
pm全称package manager，你能使用pm命令去模拟android行为或者查询设备上的应用等。在terminal里面输入adb shell pm查看使用说明

###查看安装的应用列表
adb shell pm list packages [options] \<FILTER>

选项参数

1. -f: 查看关联文件，即应用apk的位置和对应的包名
2. -d: 查看disabled packages
3. -e: 查看enable packages
4. -s: 查看system packages
5. -3: 查看第三方应用
6. -i: 查看package的对应安装者
7. -u: 查看曾被卸载的应用

###授权与取消应用的授权
注意：目标apk的minSdkVersion、targetSdkVersion也必须为23及以上

授权命令
adb shell pm grant packagename permission

取消授权命令
adb shell pm revoke packagename permission

###其他命令

1. list fratures: 设备特性、硬件之类的性能
2. list libraries: 当前设置支持的libs
3. list users: 系统上所有的users。如：UserInfo{0:Primary:3}那么USER_ID为0
4. list permissions: 查看权限列表