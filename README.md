# react-native-aliyun-push

[![npm](https://img.shields.io/npm/v/@hengkx/react-native-aliyun-push.svg?style=flat-square)](https://www.npmjs.com/package/@hengkx/react-native-aliyun-push)

[阿里云移动推送](https://www.aliyun.com/product/cps?spm=5176.2020520107.0.0.fgXGFp)react-native 封装组件

## 前提

使用本组件前提是注册过阿里云移动推送服务，注册过 app 并取得了 appKey 及 appSecret, 如果要使用 ios 版还要向苹果公司申请证书并配置好阿里云上的设置。
这里不详细描述，请参考[阿里云移动推送文档](https://help.aliyun.com/document_detail/30054.html)

## 安装

ReactNative 0.60.x 及以后

```
npm i https://github.com/mangdin/react-native-aliyun-push
```

<details>
  <summary>android配置</summary>

1. 在 Project 根目录下 build.gradle 文件中配置 maven 库 URL:

```
allprojects {
    repositories {
        mavenLocal()
        jcenter()
        maven {
            // All of React Native (JS, Obj-C sources, Android binaries) is installed from npm
            url "$rootDir/../node_modules/react-native/android"
        }
        // 下面是添加的代码
        maven {
            url "http://maven.aliyun.com/nexus/content/repositories/releases/"
        }
        // 配置HMS Core SDK的Maven仓地址。
        maven {
            url 'https://developer.huawei.com/repo/'
        }
        // 添加结束
    }
}
```

2. 确保 settings.gradle 中被添加如下代码：

```
include ':react-native-aliyun-push'
project(':react-native-aliyun-push').projectDir = new File(rootProject.projectDir, '../node_modules/react-native-aliyun-push/android')
```

3. 确保 app/build.gradle 中被添加如下代码：

```
dependencies {
    //下面是被添加的代码
    compile project(':react-native-aliyun-push')
    //添加结束
}
```

4. 确保 MainApplication.java 中被添加如下代码

```
// 下面是被添加的代码

import android.app.NotificationChannel;
import android.app.NotificationManager;
import android.content.Context;
import android.graphics.Color;
import android.os.Build;

import org.wonday.aliyun.push.AliyunPushPackage;

import com.alibaba.sdk.android.push.CloudPushService;
import com.alibaba.sdk.android.push.CommonCallback;
import com.alibaba.sdk.android.push.noonesdk.PushServiceFactory;
import com.alibaba.sdk.android.push.huawei.HuaWeiRegister;
import com.alibaba.sdk.android.push.register.MiPushRegister;
import com.alibaba.sdk.android.push.register.GcmRegister;
// 添加结束
...
    @Override
    protected List<ReactPackage> getPackages() {
      return Arrays.<ReactPackage>asList(
          new MainReactPackage(),
            //下面是被添加的代码
            new AliyunPushPackage()
            //添加结束
      );
    }
  };

  @Override
  public void onCreate() {
    super.onCreate();

    //下面是添加的代码
    this.initCloudChannel(this);
    //添加结束
  }

  // 下面是添加的代码
  /**
   * 初始化阿里云推送通道
   * @param applicationContext
   */
  private void initCloudChannel(final Context applicationContext) {

    // 创建notificaiton channel
    this.createNotificationChannel();
    PushServiceFactory.init(applicationContext);
    CloudPushService pushService = PushServiceFactory.getCloudPushService();
    pushService.setNotificationSmallIcon(R.mipmap.ic_launcher_s);//设置通知栏小图标， 需要自行添加
    //pushService.register(applicationContext, "阿里云appKey", "阿里云appSecret", new CommonCallback() {
    pushService.register(applicationContext, new CommonCallback() {
      @Override
      public void onSuccess(String responnse) {
        // success
      }
      @Override
      public void onFailed(String code, String message) {
        // failed
      }
    });

    // 关于第三方推送通道的设置，请仔细阅读阿里云文档
    // https://help.aliyun.com/document_detail/30067.html?spm=a2c4g.11186623.6.589.598b7fa8vf9qWF

    // 注册方法会自动判断是否支持小米系统推送，如不支持会跳过注册。
    MiPushRegister.register(applicationContext, "小米AppID", "小米AppKey");

    // 注册方法会自动判断是否支持华为系统推送，如不支持会跳过注册。
    HuaWeiRegister.register(this);

    // 接入FCM/GCM初始化推送
    GcmRegister.register(applicationContext, "send_id", "application_id");

    // OPPO通道注册
    OppoRegister.register(applicationContext, appKey, appSecret); // appKey/appSecret在OPPO通道开发者平台获取

    // 魅族通道注册
    MeizuRegister.register(applicationContext, "appId", "appkey"); // appId/appkey在魅族开发者平台获取

    // VIVO通道注册
    VivoRegister.register(applicationContext);

  }


  private void createNotificationChannel() {
      if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
          NotificationManager mNotificationManager = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
          // 通知渠道的id
          String id = "1";
          // 用户可以看到的通知渠道的名字.
          CharSequence name = "notification channel";
          // 用户可以看到的通知渠道的描述
          String description = "notification description";
          int importance = NotificationManager.IMPORTANCE_HIGH;
          NotificationChannel mChannel = new NotificationChannel(id, name, importance);
          // 配置通知渠道的属性
          mChannel.setDescription(description);
          // 设置通知出现时的闪灯（如果 android 设备支持的话）
          mChannel.enableLights(true);
          mChannel.setLightColor(Color.RED);
          // 设置通知出现时的震动（如果 android 设备支持的话）
          mChannel.enableVibration(true);
          mChannel.setVibrationPattern(new long[]{100, 200, 300, 400, 500, 400, 300, 200, 400});
          //最后在notificationmanager中创建该通知渠道
          mNotificationManager.createNotificationChannel(mChannel);
      }
  }
  // 添加结束

```

5. 确保 androidmanifest.xm 中被添加如下代码：

```
<meta-data
                android:name="com.vivo.push.api_key"
                android:value="xxx" />
        <meta-data
                android:name="com.vivo.push.app_id"
                android:value="xxx" />
        <meta-data
            android:name="com.huawei.hms.client.appid"
            android:value="appid=xxx" />
        <meta-data android:name="com.alibaba.app.appkey" android:value="xxx"/> <!-- 阿里云推送- appKey -->
        <meta-data android:name="com.alibaba.app.appsecret" android:value="xxx"/> <!-- 阿里云推送的appSecret -->
    
```
	
	

### 注意: 如果你使用多个阿里云 SDK, 遇到 alicloud-android-utdid 冲突，

请参考 [[这里]](https://github.com/wonday/react-native-aliyun-push/issues/113)

</details>

<details>
  <summary>ios配置</summary>

1. 执行`react-native link react-native-aliyun-push`或手工添加 node_modules/react-native-aliyun-push/ios/RCTAliyunPush.xcodeproj 到 xcode 项目工程

2. 点击项目根节点，在 targets app 的`Build Settings`中找到`Framework search path`, 添加`$(PROJECT_DIR)/../node_modules/react-native-aliyun-push/ios/libs`

3. 添加阿里云移动推送 SDK

拖拽 node_modules/react-native-aliyun-push/ios/libs 下列目录到 xcode 工程的`frameworks`目录下，选择`create folder references`。

- AlicloudUtils.framework
- CloudPushSDK.framework
- UTDID.framework
- UTMini.framework

4. 点击项目根节点，在 targets app 的 BuildPhase 的 Link Binary With Libraries 中添加公共包依赖

- libz.tbd
- libresolv.tbd
- libsqlite3.tbd
- CoreTelephony.framework
- SystemConfiguration.framework
- UserNotifications.framework

同时确保 targets app 的 BuildPhase 的 Link Binary With Libraries 包含

- AlicloudUtils.framework
- CloudPushSDK.framework
- UTDID.framework
- UTMini.framework

5. 修改 AppDelegate.m 添加如下代码

```
#import "AliyunPushManager.h"
```

```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{

...

  // 下面是添加的代码
  [[AliyunPushManager sharedInstance] setParams:@"阿里云appKey"
                                      appSecret:@"阿里云appSecret"
                                   lauchOptions:launchOptions
              createNotificationCategoryHandler:^{
                //create customize notification category here
  }];
  // 添加结束

  return YES;
}

```

```
// 下面是添加的代码

// APNs注册成功回调，将返回的deviceToken上传到CloudPush服务器
- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken
{
  [[AliyunPushManager sharedInstance] application:application didRegisterForRemoteNotificationsWithDeviceToken:deviceToken];
}


// APNs注册失败回调
- (void)application:(UIApplication *)application didFailToRegisterForRemoteNotificationsWithError:(NSError *)error
{
  [[AliyunPushManager sharedInstance] application:application didFailToRegisterForRemoteNotificationsWithError:error];
}

// 打开／删除通知回调
- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo fetchCompletionHandler:(void (^)(UIBackgroundFetchResult result))completionHandler
{
  [[AliyunPushManager sharedInstance] application:application didReceiveRemoteNotification:userInfo fetchCompletionHandler:completionHandler];
}


// 请求注册设定后，回调
- (void)application:(UIApplication *)application didRegisterUserNotificationSettings:(UIUserNotificationSettings *)notificationSettings
{
  [[AliyunPushManager sharedInstance] application:application didRegisterUserNotificationSettings:notificationSettings];
}
// 添加结束
```

</details>

<details>
  <summary>使用示例</summary>

引入模块

```
import AliyunPush from 'react-native-aliyun-push';
```

监听推送事件

```
componentDidMount() {
    //监听推送事件
    AliyunPush.addListener(this.handleAliyunPushMessage);
}

componentWillUnmount() {
    //移除监听
    AliyunPush.removeListener(this.handleAliyunPushMessage);

    //也可以用移除全部监听
    //AliyunPush.removeAllListeners()
}

handleAliyunPushMessage = (e) => {
	console.log("Message Received. " + JSON.stringify(e));


    //e结构说明:
    //e.type: "notification":通知 或者 "message":消息
    //e.title: 推送通知/消息标题
    //e.body: 推送通知/消息具体内容
    //e.actionIdentifier: "opened":用户点击了通知, "removed"用户删除了通知, 其他非空值:用户点击了自定义action（仅限ios）
    //e.extras: 用户附加的{key:value}的对象

};

```

</details>

<details>
  <summary>阿里云SDK接口封装</summary>

详细参数说明请参考阿里云移动推送 SDK [[android版]](https://help.aliyun.com/document_detail/30066.html?spm=5176.doc30064.6.643.Mu5vP0) [[ios版]](https://help.aliyun.com/document_detail/42668.html?spm=5176.doc30066.6.649.VmzJfM)

**获取 deviceId**

示例:

```
AliyunPush.getDeviceId()
    .then((deviceId)=>{
        //console.log("deviceId:"+deviceId);
    })
    .catch((error)=>{
        console.log("getDeviceId() failed");
    });
```

**绑定账号**

参数：

- account 待绑定账号

示例:

```
AliyunPush.bindAccount(account)
    .then((data)=>{
        console.log("bindAccount success");
        console.log(JSON.stringify(data));
    })
    .catch((error)=>{
        console.log("bindAccount error");
        console.log(JSON.stringify(error));
    });
```

**解绑定账号**

示例:

```
AliyunPush.unbindAccount()
    .then((result)=>{
        console.log("unbindAccount success");
        console.log(JSON.stringify(result));
    })
    .catch((error)=>{
        console.log("bindAccount error");
        console.log(JSON.stringify(error));
    });
```

**绑定标签**

参数：

- target 目标类型，1：本设备；2：本设备绑定账号；3：别名
- tags 标签（数组输入）
- alias 别名（仅当 target = 3 时生效）

示例:

```
AliyunPush.bindTag(1,["testtag1","testtag2"],"")
    .then((result)=>{
        console.log("bindTag success");
        console.log(JSON.stringify(result));
    })
    .catch((error)=>{
        console.log("bindTag error");
        console.log(JSON.stringify(error));
    });
```

**解绑定标签**

参数:

- target 目标类型，1：本设备；2：本设备绑定账号；3：别名
- tags 标签（数组输入）
- alias 别名（仅当 target = 3 时生效）

示例:

```
AliyunPush.unbindTag(1,["testTag1"],"")
    .then((result)=>{
        console.log("unbindTag succcess");
        console.log(JSON.stringify(result));
    })
    .catch((error)=>{
        console.log("unbindTag error");
        console.log(JSON.stringify(error));
    });
```

**查询当前 Tag 列表**

参数:

- target 目标类型，1：本设备

示例:

```
AliyunPush.listTags(1)
    .then((result)=>{
        console.log("listTags success");
        console.log(JSON.stringify(result));
    })
    .catch((error)=>{
        console.log("listTags error");
        console.log(JSON.stringify(error));
    });
```

**添加别名**

参数:

- alias 要添加的别名

示例:

```
AliyunPush.addAlias("testAlias")
    .then((result)=>{
        console.log("addAlias success");
        console.log(JSON.stringify(result));
    })
    .catch((error)=>{
        console.log("addAlias error");
        console.log(JSON.stringify(error));
    });
```

**删除别名**

参数:

- alias 要移除的别名

示例:

```
AliyunPush.removeAlias("testAlias")
    .then((result)=>{
        console.log("removeAlias success");
        console.log(JSON.stringify(result));
    })
    .catch((error)=>{
        console.log("removeAlias error");
        console.log(JSON.stringify(error));
    });
```

**查询别名列表**

示例:

```
AliyunPush.listAliases()
    .then((result)=>{
        console.log("listAliases success");
        console.log(JSON.stringify(result));
    })
    .catch((error)=>{
        console.log("listAliases error");
        console.log(JSON.stringify(error));
    });
```

**设置桌面图标角标数字** (ios 支持，android 支持绝大部分手机)

参数:

- num 角标数字，如果要清除请设置 0

示例:

```
AliyunPush.setApplicationIconBadgeNumber(5);
```

**获取桌面图标角标数字** (ios 支持，android 支持绝大部分手机)

示例:

```
AliyunPush.getApplicationIconBadgeNumber((num)=>{
    console.log("ApplicationIconBadgeNumber:" + num);
});
```

**同步角标数到阿里云服务端** (仅 ios 支持)

参数:

- num 角标数字

示例:

```
AliyunPush.syncBadgeNum(5);
```

**获取用户是否开启通知设定** (ios 10.0+支持)

示例:

```
AliyunPush.getAuthorizationStatus((result)=>{
    console.log("AuthorizationStatus:" + result);
});
```

**获取初始消息**

app 在未启动时收到通知后，点击通知启动 app,
如果在向 JS 发消息时，JS 没准备好或者没注册 listener，则先临时保存该消息，
并提供 getInitalMessage 方法可以获取，在 app 的 JS 逻辑完成后可以继续处理该消息

示例:

```
async componentDidMount() {
    //监听推送事件
    AliyunPush.addListener(this.handleAliyunPushMessage);
    const msg = await AliyunPush.getInitialMessage();
    if(msg){
        this.handleAliyunPushMessage(msg);
    }
}

componentWillUnmount() {
    AliyunPush.removeListener(this.handleAliyunPushMessage);
}
handleAliyunPushMessage = (e) => {
    .....
}
```

</details>
