##Activity和Fragment之间的通讯方式
1. Handler，这种方式很初级。缺点是耦合性极强，获取不到Activity的反馈
2. BoardcastReceiver，大材小用。广播适用于一对多且接收者不明确的情况下，而且广播的信息需要实现序列化接口，总之是一种耗性能的方式。
3. EventBus，事件总线可以用Action的传递。缺点是用的反射（对性能有影响），EventBus发送者和接收者分离严重，不好维护，且不可以获取Activity反馈，也不是十分理想
4. interface，这种方式总体来说还是不错。但是缺点，当项目大的时候，就需要定义很多的接口，容易混淆不好维护。
5. 观察者模式，这是最完美的方式。

###观察者模式的代码
```
/**
 * 接收通知者实现这个接口
 * Created by yelong on 2018/3/30.
 * mail:354734713@qq.com
 */
public interface Function {

    void notify(Bundle bundle);

}
```
```
/**
 * Created by yelong on 2018/3/30.
 * mail:354734713@qq.com
 */
public interface Observable<T> {

    int NOTIFY_SUCCESS = 1000;
    int NOTIFY_FAILURE = 2000;

    /**
     * 根据类名注册观察者
     */
    void registerObserver(T observer);

    /**
     * 根据名称注册观察者
     */
    void registerObserver(String name, T observer);

    /**
     * 根据名称反注册观察者
     */
    void removeObserver(String name);

    /**
     * 根据观察者反注册
     */
    void removeObserver(T observer);

    /**
     * 根据名称和观察者反注册
     */
    void removeObserver(String name, T observer);

    /**
     * 根据名称获取观察者
     */
    Set<T> getObserver(String name);

    /**
     * 清除观察者
     */
    void clear();

    /**
     * 通知观察者
     *
     * @param name   名称
     * @param bundle 参数
     * @return 返回值
     */

    int notify(String name, Bundle bundle);

}
```
```
/**
 * 所有的观察者管理类
 * Created by yelong on 2018/3/30.
 * mail:354734713@qq.com
 */
public class ObservableManager implements Observable<Function> {

    private HashMap<String, Set<Function>> mObserversMap;
    private final Object lockObj = new Object();

    private static class ObservableManagerHolder {
        private static ObservableManager instance = new ObservableManager();
    }

    private ObservableManager() {
        mObserversMap = new HashMap<>();
    }

    public static ObservableManager getInstance() {
        return ObservableManagerHolder.instance;
    }

    @Override
    public void registerObserver(Function observer) {
        synchronized (lockObj) {
            String name = observer.getClass().getSimpleName();
            Set<Function> observers;
            if (mObserversMap.containsKey(name)) {
                observers = mObserversMap.get(name);
            } else {
                observers = new HashSet<>();
                mObserversMap.put(name, observers);
            }
            observers.add(observer);
        }
    }

    @Override
    public void registerObserver(@NonNull String name, Function observer) {
        synchronized (lockObj) {
            Set<Function> observers;
            if (mObserversMap.containsKey(name)) {
                observers = mObserversMap.get(name);
            } else {
                observers = new HashSet<>();
                mObserversMap.put(name, observers);
            }
            observers.add(observer);
        }
    }


    @Override
    public void removeObserver(@NonNull String name) {
        synchronized (lockObj) {
            mObserversMap.remove(name);
        }
    }

    @Override
    public void removeObserver(@NonNull Function observer) {
        synchronized (lockObj) {
            for (String name : mObserversMap.keySet()) {
                Set<Function> observers = mObserversMap.get(name);
                observers.remove(observer);
            }
        }
    }

    @Override
    public void removeObserver(@NonNull String name, Function observer) {
        synchronized (lockObj) {
            if (mObserversMap.containsKey(name)) {
                Set<Function> observers = mObserversMap.get(name);
                observers.remove(observer);
            }
        }
    }

    @Override
    public Set<Function> getObserver(@NonNull String name) {
        Set<Function> observers = null;
        synchronized (lockObj) {
            if (mObserversMap.containsKey(name)) {
                observers = mObserversMap.get(name);
            }
        }
        return observers;
    }

    @Override
    public void clear() {
        synchronized (lockObj) {
            mObserversMap.clear();
        }
    }

    @Override
    public int notify(@NonNull String name, Bundle bundle) {
        synchronized (lockObj) {
            if (mObserversMap.containsKey(name)) {
                Set<Function> observers = mObserversMap.get(name);
                for (Function func : observers) {
                    func.notify(bundle);
                }
                return NOTIFY_SUCCESS;
            }
        }
        return NOTIFY_FAILURE;
    }

}
```

###注意点

1. 接收通知的类需要实现Function类
2. 记得注册和反注册

```
ObservableManager.getInstance().registerObserver(this);

ObservableManager.getInstance().removeObserver(this);
```
