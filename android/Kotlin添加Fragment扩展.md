## Kotlin 简化Fragment使用的扩展方法

> 为了更方便的使用Frgment，使用扩展方法对其进行扩展，来简化其使用方式

* 添加扩展函数

  ```kotlin
  inline fun FragmentManager.inTransaction(func: FragmentTransaction.() -> FragmentTransaction) = beginTransaction().func().commit()
  
  fun AppCompatActivity.addFragment(fragment: Fragment, frameId: Int) = supportFragmentManager.inTransaction { add(frameId, fragment) }
  
  fun AppCompatActivity.replaceFragment(fragment: Fragment, frameId: Int) = supportFragmentManager.inTransaction{replace(frameId, fragment)}
  
  ```

* activity中进行调用

  ```kotlin
  addFragment(mFragment1, R.id.mFlContent)
  replaceFragment(mFragment2, R.id.mFlContent)
  ```

