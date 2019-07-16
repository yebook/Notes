## 集成腾讯TBS显示Office文档与播放视频

> [官网接入文档](https://x5.tencent.com/tbs/guide/sdkInit.html)



### 1.导入lib包

	> x5暂时不提供64位so文件，为了保证64位手机能正常加载x5内核请参照如下链接修改相关配置https://x5.tencent.com/tbs/technical.html#/detail/sdk/1/34cf1488-7dc2-41ca-a77f-0014112bcab7

```
> 1.将官网下的jar包导入到项目中
> 2.在main目录下新建jniLibs包,并将liblbs.so放入(当然也可以直接放在libs包下)
> 3.在build.gradle文件下加入ndk
	android{
		//....
		defaultConfig{
			ndk {
            	abiFilters "armeabi", "armeabi-v7a",  'x86'
        	}
		}
	}
> 4.将so库放在lib包下的还需加入sourceSets（选加）
	android {
		//...
		sourceSets {
    		main {
        		jniLibs.srcDirs = ['libs']
    		}
		}
	}
```



### 2.添加相关权限

```xml
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.READ_PHONE_STATE" />
```



### 3.初始化x5内核

```java
public void initX5() {
    QbSdk.PreInitCallback cb = new QbSdk.PreInitCallback() {
        @Override
        public void onViewInitFinished(boolean arg0) {
            //x5內核初始化完成的回调，为true表示x5内核加载成功，否则表示x5内核加载失败，会自动切换到系统内核。
            LogUtil.d(" onViewInitFinished is " + arg0, "");
        }

        @Override
        public void onCoreInitFinished() {
            LogUtil.d(" onViewInitFinished is onCoreInitFinished", "");
        }
    };
    //x5内核初始化接口
    QbSdk.initX5Environment(getApplicationContext(),  cb);
}
```

> **注意：**集成中大部分问题都出在无法加载内核这一块
> 检查自己手机或设备是否有内核源，[官网检查方法](https://x5.tencent.com/tbs/technical.html#/detail/sdk/1/71884f91-badf-4314-a5b7-6cfa6aed81cc),也可以使用微信访问这个地址: [debugtbs.qq.com]查看，安装内核，而后重装应用或者重启手机



### 4.打开本地Office文件(目前仅支持本地文件，不支持在线)

```kotlin
//手动将TbsReaderView添加到你的布局中，为什么不直接写到xml中是因为它的构造函数只有一个，直接写到xml会报错
private val mTbsReaderView by lazy { TbsReaderView(this, this) }

override fun onCreate(savedInstanceState: Bundle?) {
	//...
	mFlContent.addView(mTbsReaderView, RelativeLayout.LayoutParams(RelativeLayout.LayoutParams.MATCH_PARENT, RelativeLayout.LayoutParams.MATCH_PARENT))
    
}
override fun onCallBackAction(p0: Int?, p1: Any?, p2: Any?) {
}

override fun onDestroy() {
        mTbsReaderView.onStop()
        super.onDestroy()
}

//打开本地Office文件
fun openFile() {
    //本地文件地址    
    var filePath = intent.getStringExtra("path")
    
    var bundle = Bundle()
    bundle.putString("filePath", filePath)
    //两个参数都是必需的，tempPath传入一个存在的目录就好
    bundle.putString("tempPath", cacheDir.path)
	
    //获取后缀，得到文件类型
    val suffix = filePath.substring(filePath.lastIndexOf(".") + 1)
    var result = mTbsReaderView.preOpen(suffix, false)
    //准备打开成功,打开文件
    if (result) {
        mTbsReaderView.openFile(bundle);
    }
}
```



### 5.播放视频（可播放本地与在线视频）

```xml
//在manifest中声明视频播放类，此为为sdk提供
<activity
    android:name="com.tencent.smtt.sdk.VideoActivity"
    android:configChanges="orientation|screenSize|keyboardHidden"
    android:exported="false"
    android:launchMode="singleTask"
    android:alwaysRetainTaskState="true">

    <intent-filter>
        <action android:name="com.tencent.smtt.tbs.video.PLAY" />
        <category android:name="android.intent.category.DEFAULT" />
    </intent-filter>
</activity>
```

```kotlin
//判断当前是否可用
if (TbsVideo.canUseTbsPlayer(applicationContext)) {
    //播放视频
    TbsVideo.openVideo(applicationContext, path)
} else {
    toast("播放不可用")
}
```