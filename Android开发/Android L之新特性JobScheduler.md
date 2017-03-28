JobScheduler是Android Lollipop新增的特性，用于定义满足某些条件下执行的任务。例如可用于当设备连接到充电器后，调度程序将唤醒那些需要处理器工作的程序，让他们进行工作，或者在设备连接WiFi网络的时候上传下载照片，更新内容等。JobScheduler的优势相当巨大，可以帮助手机节省电量。使用场景：后台数据上传、推送应用等。特性有

1. 支持在一个任务上组合多个条件。
2. 内置条件：设备待机，设备充电和连接网络。
3. 支持持续的job，这意味着设备重启后，之前被中断的job可以继续执行。
4. 支持设置job的最后执行期限。
5. 根据你的配置，你可以设置job在后台运行还是在主线程中运行。

###使用
1. 创建一份JobService
2. 获取任务调度服务，创建一个JobInfo
3. 注册服务

```
public class JobSchedulerService extends JobService {

    private static final String TAG = JobSchedulerService.class.getSimpleName();

    public static final String MESSENGER_INTENT_KEY
            = BuildConfig.APPLICATION_ID + ".MESSENGER_INTENT_KEY";

    //Handler信使
    private Messenger mMessenger;

    @Override
    public void onCreate() {
        super.onCreate();
        Log.i(TAG, "Service onCreate");
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        mMessenger = intent.getParcelableExtra(MESSENGER_INTENT_KEY);
        return START_NOT_STICKY;
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        Log.i(TAG, "Service onDestroy");
    }

    @Override
    public boolean onStartJob(final JobParameters params) {
        Log.i(TAG, "on stop job: " + params.getJobId());

        Handler handler = new Handler();
        handler.postDelayed(new Runnable() {
            @Override
            public void run() {
                sendMessage(MESSAGE_CODE, params.getJobId());
                //任务执行之后手动完成
                jobFinished(params, false);
            }
        },3000);
        return true;
    }

    @Override
    public boolean onStopJob(JobParameters params) {
        Log.i(TAG, "on stop job: " + params.getJobId());
        return false;
    }

    private void sendMessage(int messageID, @Nullable Object params) {
        if (mMessenger == null) {
            Log.i(TAG, "There's no callback to send a message to.");
            return;
        }
        Message message = Message.obtain();
        message.what = messageID;
        message.obj = params;
        try {
            mMessenger.send(message);
        } catch (RemoteException e) {
            e.printStackTrace();
            Log.e(TAG, "Error passing service object back to activity.");
        }
    }
}
```

```
public void createJob(View v) {
        mJobScheduler = (JobScheduler) getSystemService(JOB_SCHEDULER_SERVICE);
        //创建定时任务
        JobInfo.Builder builder = new JobInfo.Builder(mJobId++,
                new ComponentName(this, JobSchedulerService.class));
        // 这个方法告诉系统这个任务每个三秒运行一次
        //builder.setPeriodic(3000);
        // 这个函数能让你设置任务的延迟执行时间(单位是毫秒)
        builder.setMinimumLatency(3000);
        // 这个方法设置任务最晚的延迟时间,到了时间其他条件还没有满足,任务也会被启动
        builder.setOverrideDeadline(3000);
        // 这个方法告诉系统只有在满足指定的网络条件时这个任务才会被执行
        builder.setRequiredNetworkType(JobInfo.NETWORK_TYPE_ANY);
        // 这个方法告诉系统当你的设备重启之后任务是否还要继续执行
        builder.setPersisted(true);
        // 这个方法告诉系统只有当设备在充电时这个任务才会被执行
        builder.setRequiresCharging(false);
        // 这个方法告诉系统只有当用户没有在使用该设备且有一段时间没有使用时才会启动该任务。
        builder.setRequiresDeviceIdle(false);
        // 设置退避、重试策略
        builder.setBackoffCriteria(3000, JobInfo.BACKOFF_POLICY_LINEAR);

        if (mJobScheduler.schedule(builder.build()) == JobScheduler.RESULT_FAILURE) {
            mJobScheduler.cancel(mJobId);
            //mJobScheduler.cancelAll();
        }
    }

```

```
public void finishJob(View v) {
        List<JobInfo> allPendingJobs = mJobScheduler.getAllPendingJobs();
        if (allPendingJobs.size() > 0) {
            // Finish the last one
            int jobId = allPendingJobs.get(0).getId();
            mJobScheduler.cancel(jobId);
            Toast.makeText(MainActivity.this, "cancel jodId == " + jobId, Toast.LENGTH_SHORT).show();
        } else {
            Toast.makeText(MainActivity.this, "no job can cancel", Toast.LENGTH_SHORT).show();
        }
    }
```
```
<service android:name=".JobSchedulerService"
                 android:permission="android.permission.BIND_JOB_SERVICE"/>
```

[参考地址](http://gityuan.com/2017/03/10/job_scheduler_service/)