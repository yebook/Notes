## Kotlin 对Fragment扩展方法



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

