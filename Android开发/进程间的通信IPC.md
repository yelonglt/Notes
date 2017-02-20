##IPC通信的步骤
####服务端
1. 定义一个远程Service，在远程Service里面实现要做的功能
2. 定义一个通信规则（aidl文件），类似接口的定义，但是前面不能有任何修饰符，如Public
3. 在远程Service内部实现一个内部类，该内部类要集成aidl文件编译后产生的Stub类
4. 在onBind方法中将该内部类的对象返回

####客户端
1. 以隐式意图的方式绑定远程服务
2. 将aidl的文件包从服务端拷贝到客户端并编译生成通信接口
3. 实现ServiceConnection，在onServiceConnected方法将远程服务返回的Service对象转换为aidl定义的通信接口（不能强制类型转换），只能通过Stub.asInterface(service)的方式转换
4. 通过该对象去调用远程服务的功能

####示例代码
######服务器代码
创建通信规则文件，以.aidl结尾的文件

```
interface ServerConnectRegulation {
	void sayHello();
}
```
创建Service

```
public class ServerService extends Service {

	@Override
	public IBinder onBind(Intent intent) {
		MyBinder binder = new MyBinder();
		System.out.println(binder);
		return binder;
	}

	private void sayHello() {
		System.out.println("hello, every body!!!");
	}

	private class MyBinder extends Stub {

		@Override
		public void sayHello() throws RemoteException {
			ServerService.this.sayHello();
		}

	}
}
```
在AndroidManifest.xml文件里面注册服务

```
<service android:name=".services.ServerService" >
            <intent-filter>
                <action android:name="com.anjoyo.action.IPCSERVER" />
            </intent-filter>
</service>
```

######客户端代码
首先拷贝.aidl文件到客户端对应的包下，然后调用服务端的实现

```
public class IPCClientActivity extends Activity implements ServiceConnection {
	/** Called when the activity is first created. */
	@Override
	public void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.main);

		Intent intent = new Intent("com.anjoyo.action.IPCSERVER");
		bindService(intent, this, Context.BIND_AUTO_CREATE);
	}

	@Override
	public void onServiceConnected(ComponentName name, IBinder service) {
		System.out.println(service);
		ServerConnectRegulation con = Stub.asInterface(service);
		try {
			con.sayHello();
		} catch (RemoteException e) {
			e.printStackTrace();
		}
	}

	@Override
	public void onServiceDisconnected(ComponentName name) {

	}
}
```
