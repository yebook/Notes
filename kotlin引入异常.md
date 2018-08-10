### Kotlin引入第三方包时出现：Program type already present: kotlin.Deprecated



Kotlin项目中引入一个第三方的aar包，同步没问题，编译时出现此异常

```
Program type already present: kotlin.Deprecated
Message{kind=ERROR, text=Program type already present: kotlin.Deprecated, sources=[Unknown source file], tool name=Optional.of(D8)}
```



#### 解决方案一(亲测有效，推荐使用)：

在`app.gradle`加入以下代码

```
dependencies{
	...
    configurations {
            all*.exclude group: 'org.jetbrains.kotlin'
     }
 }
```



#### 解决方案二：

在 project下的`gradle.properties`中加入

```
android.enableD8=false
```

> 注意：使用此方法会有如下提示，在高版本中已被弃用
>
> The option 'android.enableD8' is deprecated and should not be used anymore.
> Use 'android.enableD8=true' to remove this warning.
> It will be removed in AGP version 3.3.



