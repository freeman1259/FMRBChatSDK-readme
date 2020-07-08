# FMRBChatSDK

**基于RBChat6.0提炼为sdk**

### 导包

floatmenu.aar、lfilepickerlibrary.aar、fmimsdk.aar放入项目libs目录下,
并添加以下依赖：
```java
    implementation files('libs/floatmenu.aar')
    implementation files('libs/lfilepickerlibrary.aar')
    implementation files('libs/fmimsdk.aar')
    implementation 'com.android.support:appcompat-v7:28.0.0'
    implementation 'com.google.code.gson:gson:2.8.5'
    implementation 'com.alibaba:fastjson:1.1.71.android'
    implementation 'com.android.support:cardview-v7:28.0.0'
    implementation 'com.yanzhenjie.permission:support:2.0.1'
    implementation 'com.squareup.okio:okio:1.17.4'
    implementation 'com.squareup.okhttp3:okhttp:3.12.3'
    implementation 'com.android.support:recyclerview-v7:28.0.0'
    implementation 'com.android.support.constraint:constraint-layout:1.1.3'
    implementation 'com.squareup.picasso:picasso:2.71828'
```

### 初始化
```java
  import cc.freeman.fmimsdk.IMClientManager;
  import cc.freeman.fmimsdk.IMConfig;
  public class MyApplication extends Application {
      public void onCreate() {
          super.onCreate();
	  IMConfig.HTTP_COMMON_CONTROLLER_URL = "http://101.200.179.162:7080/summerchat/";
          IMConfig.IM_SERVER_IP = "101.200.179.162";	  
          IMConfig.IM_SERVER_PORT = 9912;	  
          IMClientManager.init(this);
      }
  }
```

### 登录IM服务
在原app完成原登录业务后添加：
```java
  // 该变量应为登录用户的账号，目前测试期间尚未同步数据，临时使用固定账号测试
  String loginAccount = "400129";
  // 此处token由原app服务端返回,目前测试期间临时设置任意字符串
  String loginToken = "";
  // 登陆/连接全部完成后要做的事
  Observer afterLoginIMServerSucessObs = new Observer() {
	@Override
	public void update(Observable o, Object arg) {
		//MainTabActivity替换为你的登录完成后要进入的页面
		LoginRBChatHelper.afterLoginIMServerSucess(LoginActivity.this, MainTabActivity.class);
	}
  };

  // 提交用户登陆信息认证
  new LoginRBChatHelper.LoginRBChatAsyc(this
		, LoginRBChatHelper.constructLoginInfo(this, loginAccount, loginToken)
		, null, afterLoginIMServerSucessObs
  ).execute();
```
### 音视频观察者--添加到原app根页面中
```java
// MainTabActivity只是举例，使用中为你的项目中的主(根)页面
public class MainTabActivity
{
	/** 视频聊天请求观察者. */
	private Observer viodeoChatRequestObserver = new ObserverProvider.ViodeoChatRequestObserver(this);
	/** 实时语音聊天请求观察者. */
	private Observer realTimeVoiceChatRequestForNotIntChatUIObserver =
	  new ObserverProvider.RealTimeVoiceChatRequestForNotIntChatUIObserver(this);

	protected void onResume()
	{
		// 设置视频聊天请求观察者
		IMClientManager.sharedInstance
			.getTransDataListener().setViodeoChatRequestObserver(viodeoChatRequestObserver);
		
		// 设置实时语音聊天请求观察者
		IMClientManager.sharedInstance.getTransDataListener()
		 .setRealTimeVoiceChatRequestForNotIntChatUIObserver(realTimeVoiceChatRequestForNotIntChatUIObserver);
		
		super.onResume();
	}

	protected void onPause()
	{
		// 取消视频聊天请求观察者
		IMClientManager.sharedInstance
			.getTransDataListener().setViodeoChatRequestObserver(null);
		
		// 取消实时语音聊天请求观察者
		IMClientManager.sharedInstance
			.getTransDataListener().setRealTimeVoiceChatRequestForNotIntChatUIObserver(null);

		super.onPause();
	}
}
```

### 消息入口 ---- 添加到原app的“消息”界面中
在原app的消息列表页面布局文件中添加：
```java
    <cc.freeman.fmimsdk.alarm.AlarmsView
        android:id="@+id/rbchatAlarmsView"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
    </cc.freeman.fmimsdk.alarm.AlarmsView>
```
消息列表页面Activity:

```java
    msgAlarmView = findViewById(R.id.rbchatAlarmsView);
    @Override
    protected void onResume()
    {
        msgAlarmView.onResume();
        super.onResume();
    }

    @Override
    protected void onDestroy()
    {
        msgAlarmView.onDestroy();
        super.onDestroy();
    }
```
### 在退出app逻辑中调用退出IM服务方法
```java
    /**
     * 退出IM服务
     * <p>
     * @param context
     * @param exitJVM 该字段为rbchat原版方法中字段，用于退出整个app，
     *        作为sdk用于第三方app中该业务应由主app处理，所以该字段在本sdk中已设为失效
     * @param obsAfterLogoutIMServer 本观察者将在退出IM服务完成后，被调用，可根据主app业务需要传入或null
     */
    LoginRBChatHelper.doLogoutNoConfirm(final Context context, final boolean exitJVM
            , final Observer obsAfterLogoutIMServer)	    
```

### 替换消息列表页面中专属客服的默认图标
在原app主项目的drawable-xxhdpi目录中放入你的图标文件，并命名为：main_alarms_chat_message_icon.png，运行时即会覆盖sdk中的该图标。
该方法可在无需修改sdk的情况下完成替换。
