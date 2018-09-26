## 环境配置

```gradle
  //gradle中加入
  apply plugin: 'kotlin-dapt'
  //引入依赖包和编译器  
  compile "com.google.dagger:dagger:2.14.1"
  kapt "com.google.dagger:dagger-compiler:2.14.1"
```
## @Inject构造方法注入

* 使用注解标注构造方法

    ```kotlin
    class MainPresenter @Inject constructor() {
        fun doSomething():String{
            return "This is result"
        }
    }
    ```

* 使用注解标注变量

  ```kotlin
   @Inject
   lateinit var mPresenter:MainPresenter
  ```

* 使用Component建立联系

    ```kotlin
    @Component
    interface MainComponent {
        fun inject(activity:MainActivity)
    }
    ```

* **编译, 编译，编译**之后会自动生成相应Dagger开头的Component文件

* 注入注册

    ```kotlin
    class MainActivity: AppCompatActivity() {
        @Inject 
        lateinit var mPresenter:MainPresenter
        
        override fun onCreate(savedInstanceState: Bundle?) { 
            super.onCreate(savedInstanceState) 
            setContentView(R.layout.activity_main) 
            //调用注册
            initInjection() 
            mClickBtn.setOnClickListener { 
                toast(mPresenter.doSomething()) 
            } 
        } 
        /* Dagger2注入注册 */ 
        private fun initInjection() { 
            DaggerMainComponent.builder().build().inject(this) 
        }
    }
    ```


## @Module工厂注入

* 1.创建需要实例的类

  ```kotlin
  interface MainService {
  	fun getMainInfo():String    
  }
  
  class MainServiceImpl:MainService {
      override fun getMainInfo():String {
          return "This is main info"
      }
  }
  
  //在MainActivity中声明
  @Inject
  lateinit var mMainService:MainService
  ```

* 2.创建实例化工厂及方法

  ```kotlin
  @Module
  class MainModule {
      @Provides
      fun provideMainService():MainService {
          return MainServiceImpl()
      }
  }
  ```

* 3.Component与Module关联

  ```kotlin
  @Component(modules = [(MainModule::class)])
  interface MainComponent {
      fun inject(activity:MainActivity)
  }
  ```

* 4.**编译 编译 编译** 自动生成Dagger开头的相应文件

* 5.注册

  ```kotlin
  class MainActivity: AppCompatActivity() {
      @Inject
  	lateinit var mMainService:MainService
      
      override fun onCreate(savedInstanceState: Bundle?) { 
          super.onCreate(savedInstanceState) 
          setContentView(R.layout.activity_main) 
          //调用注册
          initInjection() 
          mClickBtn.setOnClickListener { 
              toast(mMainService.getMainInfo()) 
          } 
      } 
      /* Dagger2使用工厂注入*/ 
      private fun initInjection() { 
          DaggerMainComponent.builder().mainModule(MainModule()).build().inject(this) 
      }
  }
  ```



## 递归注入（构造方法与工厂相结合）

* 将MainServiceImpl构造方法使用@Inject标注

  ```kotlin
  class MainServiceImpl @Inject constructor():MainService {
      override fun getMainInfo():String {
          return "This is main info"
      }
  }
  ```

* 工厂方法修改为：

  ```kotlin
  	@Provides
      fun provideMainService(service:MainServiceImpl):MainService {
          return service
      }
  ```
