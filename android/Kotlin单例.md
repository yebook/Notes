## Kotlin 单例



#### 方式一： object实现单例(饿汉式)

```kotlin
object SimpleSington {
  fun test() {}
}

//在Kotlin里调用
SimpleSington.test()

//在Java中调用
SimpleSington.INSTANCE.test();
```

真正的实现类似于这样：

```java
public final class SimpleSington {
   public static final SimpleSington INSTANCE;

   private SimpleSington() {
      INSTANCE = (SimpleSington)this;
   }

   static {
      new SimpleSington();
   }
}
```



> 注意：
>
> - 如果构造方法中存在过多的处理，会导致加载这个类时比较慢，可能引起性能问题。
> - 如果使用饿汉式的话，只进行了类的装载，并没有实质的调用，会造成资源的浪费。

#### 方式二： 懒汉式加载

```kotlin
class LazySingleton private constructor(){
    companion object {
        val instance: LazySingleton by lazy { LazySingleton() }
    }
}
```

- 显式声明构造方法为private
- companion object用来在class内部声明一个对象
- LazySingleton的实例instance 通过lazy来实现懒汉式加载
- lazy默认情况下是线程安全的，这就可以避免多个线程同时访问生成多个实例的问题



#### 选择方式

> 对于实例初始化花费时间较少，并且内存占用较低的话，应该使用object形式的饿汉式加载。否则使用懒汉式。 



> 参考：[Kotlin中的单例模式](https://droidyue.com/blog/2017/07/17/singleton-in-kotlin/)



