### Kotlin中MutableList 转换为ArrayList

```kotlin
mutableList: MutableList<String> = mutableListOf<String>()
arrayList = ArrayList(mutableList)
```



### 数据类实现Parcelable

##### 1.在gradle中添加

```groovy
android {
    ...
   
    //使用Kotlin实验特性    
    androidExtensions {
        experimental = true
    }
}
```



##### 2. 使用注解标记数据类并实现Parcelable接口

```kotlin
@SuppressLint("ParcelCreator")
@Parcelize
data class User(val name: String, val age: Int) : Parcelable
```



