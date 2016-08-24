#Java单例的写法
####Double Checked locking 模式
```
private static SettingsDbHelper sInst = null;
public static SettingsDbHelper getInstance(Context context) {
    if (sInst == null) {                              // 1
        synchronized (SettingsDbHelper.class) {       // 2
            SettingsDbHelper inst = sInst;            // 3
            if (inst == null) {                       // 4
                inst = newSettingsDbHelper(context);  // 5
                sInst = inst;                         // 6
            }
        }
    }
    return sInst;                                     // 7
}
```
####加入volatile关键字，上面的单例性能提升25%
```
public static SettingsDbHelper getInstance(Context context) {
    SettingsDbHelper inst = sInst;  //在这里创建临时变量
    if (inst == null) {
        synchronized (SettingsDbHelper.class) {
            inst = sInst;
            if (inst == null) {
                inst = newSettingsDbHelper(context);
                sInst = inst;
            }
        }
    }
    return inst;  //注意这里返回的是临时变量

}
```
####静态内部类实现，是线程安全的使用起来也是很简洁
```
//静态内部类只有在加载内部类的时候才初始化
private static class UserManagerHolder {
    public static UserManager instance = new UserManager();
}

public UserManager() {
}

public static UserManager getInstance() {
    return UserManagerHolder.instance;
}
```


