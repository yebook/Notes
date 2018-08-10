## Bugly热更新集成记录

> 官方文档：[Bugly Android热更新使用指南](https://bugly.qq.com/docs/user-guide/instruction-manual-android-hotfix/?v=20180709165613),  [补丁打包配置详解](https://bugly.qq.com/docs/user-guide/instruction-manual-android-hotfix-demo/)
>
> 官方Demo: [Bugly-Android-Demo](https://github.com/BuglyDevTeam/Bugly-Android-Demo)



### 应用配置

#### 1.添加插件依赖

```groovy
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        // tinkersupport插件, 其中lastest.release指拉取最新版本，也可以指定明确版本号，例如1.0.4
        classpath "com.tencent.bugly:tinker-support:1.1.2"
    }
}
```



#### 2.app module 添加SDK

```groovy
// 依赖插件脚本
apply from: 'tinker-support.gradle'

android {
    defaultConfig {
        ...
        // 开启multidex
        multiDexEnabled true    

        /*ndk {
            //设置支持的SO库架构
            abiFilters 'armeabi' //, 'x86', 'armeabi-v7a', 'x86_64', 'arm64-v8a'
        }*/
    }
    
    // 签名配置
    signingConfigs {
        release {
            try {
                storeFile file("G:/workspace/key/huiming.jks")
                storePassword "test123"
                keyAlias "huiming"
                keyPassword "test123"
            } catch (ex) {
                throw new InvalidUserDataException(ex.toString())
            }
        }

        debug {
            //storeFile file("G:/workspace/key/huiming.jks")
        }
    }
    
    buildTypes {
        release {
            minifyEnabled true
            signingConfig signingConfigs.release
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
        debug {
            debuggable true
            minifyEnabled false
            // signingConfig signingConfigs.debug
        }
    }
}
dependencies {
    compile "com.android.support:multidex:1.0.1" // 多dex配置
    //注释掉原有bugly的仓库
    //compile 'com.tencent.bugly:crashreport:latest.release'//其中latest.release指代最新版本号，也可以指定明确的版本号，例如1.3.4
    compile 'com.tencent.bugly:crashreport_upgrade:1.3.5'
    // 指定tinker依赖版本（注：应用升级1.3.5版本起，不再内置tinker）
    compile 'com.tencent.tinker:tinker-android-lib:1.9.6'
    //按需要添加，如有需要native支持
    //compile 'com.tencent.bugly:nativecrashreport:latest.release' //其中latest.release指代最新版本号，也可以指定明确的版本号，例如2.2.0
}
```



#### 3. app gradle同级目录下新建 `tinker-support.gradle` 文件

```groovy
apply plugin: 'com.tencent.bugly.tinker-support'

def bakPath = file("${buildDir}/bakApk/")

/**
 * 此处填写每次构建生成的基准包目录
 */
def baseApkDir = "app-0726-11-48-25"

/**
 * 对于插件各参数的详细解析请参考
 */
tinkerSupport {

    // 开启tinker-support插件，默认值true
    enable = true

    // 自动生成tinkerId, 你无须关注tinkerId，默认为false
    autoGenerateTinkerId = true

    // 指定归档目录，默认值当前module的子目录tinker
    autoBackupApkDir = "${bakPath}"

    // 是否启用覆盖tinkerPatch配置功能，默认值false
    // 开启后tinkerPatch配置不生效，即无需添加tinkerPatch
    overrideTinkerPatchConfiguration = true

    // 编译补丁包时，必需指定基线版本的apk，默认值为空
    // 如果为空，则表示不是进行补丁包的编译
    // @{link tinkerPatch.oldApk }
    baseApk = "${bakPath}/${baseApkDir}/app-release.apk"
//    baseApk =  "${bakPath}/${baseApkDir}/app-debug.apk"

    // 对应tinker插件applyMapping
    baseApkProguardMapping = "${bakPath}/${baseApkDir}/app-release-mapping.txt"
//    baseApkProguardMapping = "${bakPath}/${baseApkDir}/app-debug-mapping.txt"

    // 对应tinker插件applyResourceMapping
    baseApkResourceMapping = "${bakPath}/${baseApkDir}/app-release-R.txt"
//    baseApkResourceMapping = "${bakPath}/${baseApkDir}/app-debug-R.txt"

    // 构建基准包跟补丁包都要修改tinkerId，主要用于区分
//    tinkerId = "1.0.3-ccc"

    // 打多渠道补丁时指定目录
    // buildAllFlavorsDir = "${bakPath}/${baseApkDir}"

    // 是否使用加固模式，默认为false
    // isProtectedApp = true

    // 是否采用反射Application的方式集成，无须改造Application
    enableProxyApplication = true

    // 支持新增Activity
    supportHotplugComponent = true

}

/**
 * 一般来说,我们无需对下面的参数做任何的修改
 * 对于各参数的详细介绍请参考:
 * https://github.com/Tencent/tinker/wiki/Tinker-%E6%8E%A5%E5%85%A5%E6%8C%87%E5%8D%97
 */
tinkerPatch {
    tinkerEnable = true
    ignoreWarning = false
    useSign = false
    dex {
        dexMode = "jar"
        pattern = ["classes*.dex"]
        loader = []
    }
    lib {
        pattern = ["lib/*/*.so"]
    }

    res {
        pattern = ["res/*", "r/*", "assets/*", "resources.arsc", "AndroidManifest.xml"]
        ignoreChange = []
        largeModSize = 100
    }

    packageConfig {
    }
    sevenZip {
        zipArtifact = "com.tencent.mm:SevenZip:1.1.10"
//        path = "/usr/local/bin/7za"
    }
    buildConfig {
        keepDexApply = false
//      tinkerId = "base-2.0.1"
    }
}
```



#### 4.初始化SDK

```kotlin
class MyApp : Application() {

    override fun onCreate() {
        super.onCreate()
		
        initUpdate()
    }

    fun initUpdate() {
        setStrictMode()
        // 设置是否开启热更新能力，默认为true
        Beta.enableHotfix = true
        // 设置是否自动下载补丁
        Beta.canAutoDownloadPatch = true
        // 设置是否提示用户重启
        Beta.canNotifyUserRestart = true
        // 设置是否自动合成补丁
        Beta.canAutoPatch = true

        /**
         * 全量升级状态回调
         */
        Beta.upgradeStateListener = object : UpgradeStateListener {
            override fun onUpgradeFailed(b: Boolean) {

            }

            override fun onUpgradeSuccess(b: Boolean) {

            }

            override fun onUpgradeNoVersion(b: Boolean) {
                Toast.makeText(applicationContext, "最新版本", Toast.LENGTH_SHORT).show()
            }

            override fun onUpgrading(b: Boolean) {
                Toast.makeText(applicationContext, "onUpgrading", Toast.LENGTH_SHORT).show()
            }

            override fun onDownloadCompleted(b: Boolean) {

            }
        }

        /**
         * 补丁回调接口，可以监听补丁接收、下载、合成的回调
         */
        Beta.betaPatchListener = object : BetaPatchListener {
            override fun onPatchReceived(patchFileUrl: String) {
                Toast.makeText(applicationContext, patchFileUrl, Toast.LENGTH_SHORT).show()
            }

            override fun onDownloadReceived(savedLength: Long, totalLength: Long) {
                Toast.makeText(applicationContext, String.format(Locale.getDefault(),
                        "%s %d%%",
                        Beta.strNotificationDownloading,
                        (if (totalLength == 0L) 0 else savedLength * 100 / totalLength).toInt()), Toast.LENGTH_SHORT).show()
            }

            override fun onDownloadSuccess(patchFilePath: String) {
                Toast.makeText(applicationContext, patchFilePath, Toast.LENGTH_SHORT).show()
                //                Beta.applyDownloadedPatch();
            }

            override fun onDownloadFailure(msg: String) {
                Toast.makeText(applicationContext, msg, Toast.LENGTH_SHORT).show()
            }

            override fun onApplySuccess(msg: String) {
                Toast.makeText(applicationContext, msg, Toast.LENGTH_SHORT).show()
            }

            override fun onApplyFailure(msg: String) {
                Toast.makeText(applicationContext, msg, Toast.LENGTH_SHORT).show()
            }

            override fun onPatchRollback() {
                Toast.makeText(applicationContext, "onPatchRollback", Toast.LENGTH_SHORT).show()
            }
        }

        val start = System.currentTimeMillis()
        Bugly.setUserId(this, "falue")
        Bugly.setUserTag(this, 123456)
        Bugly.putUserData(this, "key1", "123")
        Bugly.setAppChannel(this, "bugly")


        // 这里实现SDK初始化，appId替换成你的在Bugly平台申请的appId,调试时将第三个参数设置为true
        Bugly.init(this, "986f303f96", true)
        val end = System.currentTimeMillis()
        Log.e("init time--->", (end - start).toString() + "ms")
    }

    override fun attachBaseContext(base: Context?) {
        super.attachBaseContext(base)
        // you must install multiDex whatever tinker is installed!
        MultiDex.install(base)

        // 安装tinker
        Beta.installTinker()
    }


    @TargetApi(9)
    protected fun setStrictMode() {
        StrictMode.setThreadPolicy(StrictMode.ThreadPolicy.Builder().permitAll().build())
        StrictMode.setVmPolicy(StrictMode.VmPolicy.Builder().detectAll().penaltyLog().build())
    }
}
```



#### 5.AndroidManifest.xml配置

* 权限配置

  ```xml
  <uses-permission android:name="android.permission.READ_PHONE_STATE" />
  <uses-permission android:name="android.permission.INTERNET" />
  <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
  <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
  <uses-permission android:name="android.permission.READ_LOGS" />
  <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
  ```



### 补丁打包

#### 1.编译基准包

> 注意：请确认`tinker-support.gradle`文件中是否有配置 autoGenerateTinkerId = true 自动生成tinkerId
>
> 如果没有，请手动修改tinkerId



打开AS右边的Gradle 执行`assembleRelease`编译生成基准包： 

![1532585447385](http://pd8746ife.bkt.clouddn.com/image/gradle_build.png)

这个会在build/outputs/bakApk路径下生成每次编译的基准包、混淆配置文件、资源Id文件，如下图所示： 

![](http://pd8746ife.bkt.clouddn.com/image/2.jpg) 



> 注意：app-release-mapping如果没有生成，请检查是否配置了签名

> 请先安装好并启动一次基准包apk
>
> 否则发布上传补丁包时会出现未找到包的情况



#### 2.生成升级补丁

* 修改 `tinker-support.gradle` 文件下基准包目录地址

  ```groovy
  /**
   * 此处填写每次构建生成的基准包目录
   */
  def baseApkDir = "app-0726-11-48-25"
  ```

* 使用`buildTinkerPatchRelase` 进行编译生成补丁包

  ![1532585447385](http://pd8746ife.bkt.clouddn.com/image/gradle_build.png)



* 编译完成后，将会在./build/outputs/patch/release目录下生成补丁包

  ![补丁包路径](http://pd8746ife.bkt.clouddn.com/image/%E8%A1%A5%E4%B8%81%E5%8C%85%E8%B7%AF%E5%BE%84.png)



* 将补丁包上传至bugly中下发补丁



### 问题记录

#### 1. 错误：Tinker.DefaultLoadReporter: patch loadReporter onLoadPatchListenerReceiveFail: patch receive fail: /data/user/0/xxxxxxxx/app_tmpPatch/tmpPatch.apk, code: -5

> 这个是因为这个手机在6.0的时候编译选项打开了JIT，可能会对patch产生影响，会对这部分手机关闭patch功能
>
> [GitHubIssues解释](https://github.com/Tencent/tinker/issues/625) 



#### 2.assembleRelease编译时出现错误 BUILD FAILED

```
Warning: com.qihancloud.opensdk.function.unit.MediaManager: can't find referenced class org.jetbrains.annotations.NotNull

Warning: okhttp3.internal.platform.ConscryptPlatform: can't find referenced class org.conscrypt.Conscrypt
Warning: okhttp3.internal.platform.ConscryptPlatform: can't find referenced class org.conscrypt.Conscrypt
Warning: okhttp3.internal.platform.ConscryptPlatform: can't find referenced class org.conscrypt.Conscrypt

...

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':app:transformClassesAndResourcesWithProguardForRelease'.
> Job failed, see logs for details

* Try:
Run with --stacktrace option to get the stack trace. Run with --info or --debug option to get more log output. Run with --scan to get full insights.

* Get more help at https://help.gradle.org

BUILD FAILED in 45s
```



----

**解决方案：在`proguard-rules.pro`中添加相对应的混淆**

```
-keep class com.qihancloud.opensdk.** {*;}
-dontwarn okio.**
-dontwarn retrofit2.Call
-dontnote retrofit2.Platform$IOS$MainThreadExecutor
#忽略所有警告
-ignorewarnings
```







