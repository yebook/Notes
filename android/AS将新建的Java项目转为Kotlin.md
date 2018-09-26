

## AS将新建的Module的Java项目转为Kotlin项目

#### 选中Module，将Java文件转为Kotlin

![1535511274111](F:\Notes\1535511274111.png)



#### 将项目转为Kotlin

* ##### 方案一：使用AS工具来进行转换

![1535511732296](F:\Notes\1535511732296.png)

![1535511814457](F:\Notes\1535511814457.png)

* ##### 方案二：手动添加相关代码

  > 打开module的`build.gradle`文件添加以下代码

  ```groovy
  apply plugin: 'com.android.library'
  apply plugin: 'kotlin-android'	//添加此行
  ...
  dependencies {
      ...
      //添加此行
      implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version"	
  }
  ```
