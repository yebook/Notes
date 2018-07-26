## dagger2 Component之间的依赖

> Component之间的依赖方式有二种：
>
> 一、通过Dependencies进行依赖
>
> 二、通过@Subcomponent进行依赖



### Dependencies依赖

```kotlin
//测试数据类
data class User(val name:String, val age: Int)

//父级
@Module
class TestParentModule(private val user: User) {
    @Provides
    fun provideTestParentValue(): User {
        return this.user
    }
}

@Component(modules = [(TestParentModule::class)])
interface TestParentComponent {
    fun user(): User
}

//子级
@Module
class TestChildModule {
    @Provides
    fun provideTestChildService(service: TestChildServiceImpl): TestChildService {
        return service
    }
}

//使用dependencies进行依赖
@Component(dependencies = [(TestParentComponent::class)], modules = [(TestChildModule::class)])
interface TestChildComponent {
    fun inject(activity: SubActivity)
}


interface TestChildService {
    fun getInfo(): String
}

class TestChildServiceImpl @Inject constructor() : TestChildService {
    override fun getInfo(): String {
        return "This is Test Child Info"
    }
}


//注解声明
@Inject
lateinit var mUser: User

@Inject
lateinit var mTestChildService: TestChildService

//编译后进行初始化
 fun initJection() {
        val parentComponent = DaggerTestParentComponent.builder().testParentModule(TestParentModule(User("张三", 18))).build()
        DaggerTestChildComponent.builder().testParentComponent(parentComponent).testChildModule(TestChildModule()).build().inject(this)
}

//注入成功
toast("mUser: ${mUser.toString()}\n child: ${mTestChildService.getInfo()}")

```



### @Subcomponent依赖

> 1."子级Component"使用@Subcomponent进行标注，不再需要dependencies;
>
> 2."父级Component"需要把"子级Component"追加到当前Component中，同时传入“子级Component”需要的Module;

```kotlin
//父级
@Component(modules = [(ParentModule::class)])
interface ParentComponent {
    fun addSub(module: ChildModule): ChildComponent
}

@Module
class ParentModule {
    @Provides
    fun provideParentService(service: ParentServiceImpl): ParentService {
        return service
    }
}

interface ParentService {
    fun getParentInfo():String
}
class ParentnServiceImpl @Inject constructor(): ParentService {
    override fun getParentInfo():String {
        return "Parent info"
    }
}


//子级
@Subcomponent(modules = [(ChildModule::class)])
interface ChildComponent {
    fun inject(activity: MainActivity)
}

@Module
class ChildModule {
    @Provides
    fun provideChildService(service: ChildServiceImpl): ChildService {
        return service
    }
}

interface ChildService {
    fun getChildInfo(): String
}

class ChildServiceImpl @Inject constructor(): ChildService {
    override fun getChildInfo(): String {
        return "Child Info"
    }
}

//注解声明
@Inject
lateinit var mParentService: ParentService
@Inject
lateinit var mChildService: ChildService

//编译后初始化
fun initInjection() {
    DaggerParentComponent.builder().parentModule(ParentMoule()).build().addSub(ChildModule()).inject(this)
}

//使用
toast(mParentService.getParentInfo())
toast(mChildService.getChildInfo())

```



### 使用场景与差别

> 使用@Subcomponent，我们需要“父级”知道“子级”的存在（因为需要添加追加方法），如果它们分别在不同的模块（Android多模块化），而且“父级”处于被依赖层（子模块引用父模块），@Subcomponent就达不到我们的目的。所以，他们之间还是有一些差别，具体如下：
>
> - dependencies适合于全局通用性依赖；@Subcomponent同类业务通性继承；
> - dependencies更单纯的属于依赖性质；@Subcomponent更适合”继承“性质；
> - dependencies需要”父级“提供服务能力；@Subcomponent不需要”父级“提供服务，但是需要追加”子级“方法；
> - ”Dagger2注册“方式有所区别，具体可查看对应代码；





