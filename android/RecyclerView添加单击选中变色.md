## RecyclerView添加单击选中变色(Kotlin)

```kotlin
class DemoAdapter(data: List<String>) : BaseQuickAdapter<String, BaseViewHolder>(R.layout.item_demo, data) {
    //定义当前选中的positoin
    var mSelectedIndex: Int = 0
    //定义点击回调函数
    var itemClickListener: ((String) -> Unit)? = null
    
    init {
        setOnItemClickListener { adapter, view, position ->
            //点击选中
            mSelectedIndex = position
            var item = data.get(position)
            itemClickListener?.let { item?.let { it(item) } }
            //如数据过多，可只刷新需要改变的item
            //notifyItemChanged(mSelectedIndex)
            //notifyItemChanged(oldSelectedIndex)                    
            notifyDataSetChanged()
        }
    }

    override fun convert(helper: BaseViewHolder?, item: SettingsBean?) {
        helper?.let {
            it.itemView.setBackgroundResource(if(it.layoutPosition == mSelectedIndex) R.color.select_gray else R.color.normal_gray)
        }
    }
}
```

