##RxJava的使用

####RxJava使用三部曲
RxJava是基于观察者模式的

1. 初始化被观察者（Observable、Flowable（支持背压））。生产事件，相关函数如：create、just等。
2. 初始化观察者（Observer、Subscriber（支持背压））。消费事件Observer、Consumer（简化订阅）
3. 建立订阅关系

####线程调度

1. 线程选项
   * Schedulers.io() 代表io操作，通常用于网络或读写文件等io密集的操作
   * Schedulers.computation() 代表CPU计算密集型的操作，例如需要大量计算的操作
   * Schedulers.newThread() 代表一个常规的新线程
   * AndroidSchedulers.mainthread() 代表Android主线程 
2. 生成事件所在线程subScribeOn()
3. 消费事件所在线程observeOn()
4. 注意点
   * 多次指定生产者的线程只有第一次指定有效。
   * 多次指定订阅者接收接收线程是可以的，也就是说每次调用一次observeOn()下游的线程就会切换一次

####操作符

1. map操作符。可以将一个Observable对象通过某种关系转换为另一个Observable对象
2. concat操作符。把多个Observable组合在一起按顺序执行。使用场景如注册成功之后登录、网络请求数据先读取缓存然后读取网络数据
3. flatMap操作符。把一个Observable对象转换成多个Observable，然后把产生的多个Observable放到一个单独的Observable里。但是这样没办法保证事件的顺序
4. concatMap操作符。功能和flatMap类似，但是可以保证事件的顺序
5. zip操作符。用于合并事件。使用场景获取多个接口数据共同更新UI
6. distinct操作符。去重操作符
7. filter操作符。过滤操作符，参数Predicate类
8. buffer操作符。类似拼接操作符
9. timer操作符。定时任务操作符
10. interval操作符。定时任务操作符，带有延时功能
11. skip操作符。跳过n个事件之后在接收处理
12. take操作符。最多接收n个事件之后就不在处理。

####示例代码
```
public class RxApiUtil {

    private static final String TAG = RxApiUtil.class.getSimpleName();

    /**
     * 创建事件
     */
    public void operateCreate() {
        Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<String> e) throws Exception {
                // 通过ObservableEmitter类对象产生事件并通知观察者
                e.onNext("yelong");
                e.onNext("dial");
                e.onNext("ting");
                e.onComplete();
            }
        }).subscribe(new Observer<String>() {

            private Disposable mDisposable;

            @Override
            public void onSubscribe(@NonNull Disposable d) {
                Log.d(TAG, "事件.....onSubscribe");
                mDisposable = d;
            }

            @Override
            public void onNext(@NonNull String s) {
                Log.d(TAG, "事件....." + s);

                if (s.equals("dial")) {
                    // 切断事件的传递
                    mDisposable.dispose();
                }
            }

            @Override
            public void onError(@NonNull Throwable e) {
                Log.d(TAG, "事件.....onError");
            }

            @Override
            public void onComplete() {
                Log.d(TAG, "事件.....onComplete");
            }
        });
    }

    /**
     * map的作用是变换
     */
    public void operateMap() {
        Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {
                e.onNext(1);
                e.onNext(2);
                e.onNext(3);
            }
        }).map(new Function<Integer, String>() {
            @Override
            public String apply(@NonNull Integer integer) throws Exception {
                return "this is result " + integer;
            }
        }).subscribe(new Consumer<String>() {
            @Override
            public void accept(String s) throws Exception {

            }
        });
    }

    /**
     * 用于合并事件
     * 取发射器sObservable和发射器iObservable的第一个事件进行配对，一个事件只能被使用一次，
     * 组合的顺序严格按照事件发送的顺序来组合。最终接收器接收到的事件数量和事件最少的发射器事件数量一样。
     * 使用场景：一个页面多个接口返回，然后全部拿到后渲染UI
     */
    public void operateZip() {

        Observable<String> sObservable = Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<String> e) throws Exception {
                if (!e.isDisposed()) {
                    e.onNext("A");
                    e.onNext("B");
                    e.onNext("C");
                }
            }
        });

        Observable<Integer> iObservable = Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {
                if (!e.isDisposed()) {
                    e.onNext(1);
                    e.onNext(2);
                    e.onNext(3);
                }
            }
        });

        Observable.zip(sObservable, iObservable, new BiFunction<String, Integer, String>() {
            @Override
            public String apply(@NonNull String s, @NonNull Integer integer) throws Exception {
                return s + integer;
            }
        }).subscribe(new Consumer<String>() {
            @Override
            public void accept(String s) throws Exception {

            }
        });
    }

    /**
     * 把多个事件发射器组合在一起然后让顺序执行。可适用用户注册成功之后自动登录
     */
    public void operateConcat() {
        Observable.concat(Observable.just(1, 2, 3), Observable.just(4, 5)).subscribe(new Consumer<Integer>() {
            @Override
            public void accept(Integer integer) throws Exception {

            }
        });
    }

    /**
     * 把一个发射器通过某种方法转换成多个发射器，然后再把这些单一的发射器装进一个单一的发射器
     * 注意点：flatMap不能保证事件的顺序，如果需要保证事件的顺序，需要用到concatMap
     */
    public void operateFlatMap() {

        Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {
                e.onNext(1);
                e.onNext(2);
                e.onNext(3);
            }
        }).flatMap(new Function<Integer, ObservableSource<String>>() {
            @Override
            public ObservableSource<String> apply(@NonNull Integer integer) throws Exception {
                List<String> list = new ArrayList<String>();
                for (int i = 0; i < 3; i++) {
                    list.add("I am value:" + integer);
                }
                int delayTime = (int) (1 + Math.random() * 10);
                return Observable.fromIterable(list).delay(delayTime, TimeUnit.MILLISECONDS);
            }
        }).subscribeOn(Schedulers.newThread()).observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<String>() {
                    @Override
                    public void accept(String s) throws Exception {

                    }
                });
    }

    /**
     * 保证事件的顺序
     */
    public void operateConcatMap() {
        Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {
                e.onNext(1);
                e.onNext(2);
                e.onNext(3);
            }
        }).concatMap(new Function<Integer, ObservableSource<String>>() {
            @Override
            public ObservableSource<String> apply(@NonNull Integer integer) throws Exception {
                List<String> list = new ArrayList<String>();
                for (int i = 0; i < 3; i++) {
                    list.add("I am value:" + integer);
                }
                int delayTime = (int) (1 + Math.random() * 10);
                return Observable.fromIterable(list).delay(delayTime, TimeUnit.MILLISECONDS);
            }
        }).subscribeOn(Schedulers.newThread()).observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<String>() {
                    @Override
                    public void accept(String s) throws Exception {

                    }
                });
    }

    /**
     * 去重操作符
     */
    public void operateDistinct() {

        Observable.just(1, 1, 2, 3, 4, 4, 5, 6, 1, 2).distinct()
                .subscribe(new Consumer<Integer>() {
                    @Override
                    public void accept(Integer integer) throws Exception {

                    }
                });

    }

    /**
     * 过滤操作符
     */
    public void operateFilter() {
        Observable.just(1, 20, 30, -12, -10).filter(new Predicate<Integer>() {
            @Override
            public boolean test(@NonNull Integer integer) throws Exception {
                return integer > 0;
            }
        }).subscribe(new Consumer<Integer>() {
            @Override
            public void accept(Integer integer) throws Exception {

            }
        });
    }

    /**
     * 类似拼接操作符。看日志就可以很好的理解
     */
    public void operateBuffer() {
        Observable.just(1, 2, 3, 4, 5, 6).buffer(3, 2)
                .subscribe(new Consumer<List<Integer>>() {
                    @Override
                    public void accept(List<Integer> integers) throws Exception {

                    }
                });
    }

    /**
     * 定时任务操作符
     */
    public void operateTimer() {
        Observable.timer(10, TimeUnit.SECONDS).subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<Long>() {
                    @Override
                    public void accept(Long aLong) throws Exception {

                    }
                });
    }

    /**
     * 定时任务操作符，第一次执行延迟3秒间隔时间为2秒
     */
    public void operateInterval() {
        Observable.interval(3, 2, TimeUnit.SECONDS).subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<Long>() {
                    @Override
                    public void accept(Long aLong) throws Exception {

                    }
                });
    }

    /**
     * 在接收事件之前，做点其他额外的操作
     */
    public void doOnNext() {

        Observable.just(12, 20, 30, 40).doOnNext(new Consumer<Integer>() {
            @Override
            public void accept(Integer integer) throws Exception {
                Log.d(TAG, "事件.....接收事件之前 ->accept");
            }
        }).subscribe(new Consumer<Integer>() {
            @Override
            public void accept(Integer integer) throws Exception {
                Log.d(TAG, "事件.....消费事件 ->accept==" + integer);
            }
        });
    }

    /**
     * 跳过几个事件之后再接收处理
     */
    public void operateSkip() {

        Observable.just(1, 2, 3, 4, 5, 6).skip(2)
                .subscribe(new Consumer<Integer>() {
                    @Override
                    public void accept(Integer integer) throws Exception {

                    }
                });
    }

    /**
     * 最多接收发射器发送的conut个事件
     */
    public void operateTake() {

        Observable.just(1, 2, 3, 4, 5, 6).take(2)
                .subscribe(new Consumer<Integer>() {
                    @Override
                    public void accept(Integer integer) throws Exception {

                    }
                });
    }
}

```

>[GitHub RxJava链接](https://github.com/ReactiveX/RxJava)