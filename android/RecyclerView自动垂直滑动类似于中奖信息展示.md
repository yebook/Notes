## RecyclerView实现垂直自动无限滚动，类似于中奖信息，跑马灯

[java版源码](https://download.csdn.net/download/ye1016649801/10739631)



> 实现RecyclerView中的效果有两种：
>
> 一种为以item为单体，每隔多少秒进行滚动一次
>
> 一种为整体流形式进行缓慢滚动



![效果图](http://pd8746ife.bkt.clouddn.com/autoscrollrecyclerview.gif)



#### 实现无限滚动

> 这里实现无限滚动的方式为在adpater中设置itemCount为 Integer.MAX_VALUE
>
> 注意：此处基于BaseQuickAdapter的库进行的，也可直接使用原生

```kotlin
class MainAdatper(data: List<String>) : BaseQuickAdapter<String, BaseViewHolder>(R.layout.item_txt, data) {

    override fun convert(helper: BaseViewHolder?, item: String?) {
        helper?.setText(R.id.mTv, item)
    }


    override fun getItem(position: Int): String? {
        val newPosition = position % data.size
        return getData().get(newPosition)
    }

    override fun getItemViewType(position: Int): Int {
        var count = getHeaderLayoutCount() + getData().size
        //刚开始进入包含该类的activity时,count为0。就会出现0%0的情况，这会抛出异常，所以我们要在下面做一下判断
        if (count <= 0) {
            count = 1
        }
        var newPosition = position % count;
        return super.getItemViewType(newPosition);
    }

    override fun getItemCount(): Int {
        return Integer.MAX_VALUE
    }

}
```







#### item为单位，每隔n秒滚动一个item

> 这边主要是使用LinearLayoutManager中的**startSmoothScroll()** 方法实现，然后再配合自定义layoutManager 控制速度达到目的效果



```kotlin

mRv.layoutManager = LinearLayoutManager(this)
mRv.addItemDecoration(DividerItemDecoration(this, DividerItemDecoration.VERTICAL))
mRv.adapter = MainAdatper(data)

mSmoothScroll = object : LinearSmoothScroller(this) {
    override fun getVerticalSnapPreference(): Int {
        return LinearSmoothScroller.SNAP_TO_START
    }

    override fun calculateSpeedPerPixel(displayMetrics: DisplayMetrics?): Float {
        // 移动一英寸需要花费3ms
        return 3f / (displayMetrics?.density ?: 1f)
    }
}



==============================================================
fun startAuto() {
    if (mAutoTask != null && (mAutoTask?.isDisposed ?: true))
        mAutoTask?.dispose()
	
    //此处的4为当前显示的最后一个item
    mAutoTask = Observable.interval(1, 2, TimeUnit.SECONDS).subscribe {
        //定位到指定项如果该项可以置顶就将其置顶显示
        mSmoothScroll.targetPosition = it.toInt()
        (mRv.layoutManager as LinearLayoutManager).startSmoothScroll(mSmoothScroll)
    }
}

fun stopAuto() {
    if (mAutoTask != null && (mAutoTask?.isDisposed ?: true)) {
        mAutoTask?.dispose()
        mAutoTask = null
    }
}


override fun onStart() {
    super.onStart()
    //开始滚动
    startAuto()
}

override fun onStop() {
    super.onStop()
    //停止滚动
    stopAuto()
}

```



#### 流式滚动

> 流式滚动是使用recyclerView中的 **smoothScrollBy(x, y)** 方法来实现的
>
> 此处是通过自定义recyclerView来进行实现，也可直接在activity中进行实现。



```kotlin
class AutoScrollRecyclerView(mContext: Context, attrs: AttributeSet?) : RecyclerView(mContext, attrs) {

    private var mAutoTask: Disposable? = null

    //禁止手动滑动
    override fun onTouchEvent(e: MotionEvent): Boolean {
        return true
    }

    fun start() {
        if (mAutoTask != null && !mAutoTask!!.isDisposed) {
            mAutoTask!!.dispose()
        }
        mAutoTask = Observable.interval(1000, 100, TimeUnit.MILLISECONDS).observeOn(AndroidSchedulers.mainThread()).subscribe { smoothScrollBy(0, 20) }
    }

    fun stop() {
        if (mAutoTask != null && !mAutoTask!!.isDisposed) {
            mAutoTask!!.dispose()
            mAutoTask = null
        }
    }

}

==================================================
//在activity中调用开始滚动与停止滚动
override fun onStart() {
    super.onStart()
    mRvTwo.start()
}

override fun onStop() {
    super.onStop()
    mRvTwo.stop()
}
    
```



### 源码

[kotlin源码](https://github.com/yebook/AutoScrollRecyclerView)

[java源码](https://download.csdn.net/download/ye1016649801/10739631)



### 感谢

* https://stackoverflow.com/questions/31235183/recyclerview-how-to-smooth-scroll-to-top-of-item-on-a-certain-position
* https://www.jianshu.com/p/3acc395ae933