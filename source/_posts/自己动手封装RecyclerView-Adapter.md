---
title: 自己动手封装RecyclerView.Adapter
date: 2019-05-18 11:09:06
tags: android
categories: android
comments: false
---
RecyclerView封装了ViewHolder的回收复用，也就是说RecyclerView标准化了ViewHolder，编写Adapter面向的是ViewHolder而不再是View了，复用的逻辑被封装了，写起来更加简单。直接省去了ListView中convertView.setTag(holder)和convertView.getTag()这些繁琐的步骤。
<!-- more -->
### 提供了一种插拔式的体验，高度的解耦，异常的灵活，针对一个Item的显示RecyclerView专门抽取出了相应的类，来控制Item的显示，使其的扩展性非常强。
### 设置布局管理器以控制Item的布局方式，横向、竖向以及瀑布流方式
### 可设置Item的间隔样式（可绘制），通过继承RecyclerView的ItemDecoration这个类，然后针对自己的业务需求去书写代码。
### 可以控制Item增删的动画，可以通过ItemAnimator这个类进行控制，当然针对增删的动画，RecyclerView有其自己默认的实现。

``` bash
/**
 * 自定义封装RecyclerView.Adapter
 */
abstract class RecyclerViewAdapter<M> : RecyclerView.Adapter<RecyclerViewHolder>() {

    companion object {
        /**
         * 加载中
         */
        private const val LOADING = 0
        /**
         * 加载失败
         */
        private const val LOAD_FAILURE = 1
        /**
         * 全部加载完成
         */
        private const val LOAD_COMPLETE = 2
    }

    abstract val models: List<M>
    private var isEnableLoadMore: Boolean = false
    private var mOnLoadMoreListener: OnLoadMoreListener? = null
    private var mOnItemClickListener: OnItemClickListener? = null
    private var mOnItemLongClickListener: OnItemLongClickListener? = null
    private var mLoadStatus: Int = LOADING

    fun setLoadMoreComplete(hasMore: Boolean) {
        this.isEnableLoadMore = hasMore
        this.mLoadStatus = if (hasMore) LOADING else LOAD_COMPLETE
        notifyDataSetChanged()
    }

    fun setLoadMoreFailure() {
        this.mLoadStatus = LOAD_FAILURE
        notifyItemChanged(itemCount - 1)
    }

    fun setOnLoadMoreListener(listener: OnLoadMoreListener) {
        this.mOnLoadMoreListener = listener
    }

    fun setOnItemClickListener(listener: OnItemClickListener) {
        this.mOnItemClickListener = listener
    }

    fun setOnItemLongClickListener(listener: OnItemLongClickListener) {
        this.mOnItemLongClickListener = listener
    }

    override fun getItemViewType(position: Int): Int {
        if (this.isEnableLoadMore && position == this.models.size) {
            return R.layout.rv_load_more
        }
        return getItemViewLayout(position)
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): RecyclerViewHolder {
        val inflater = LayoutInflater.from(parent.context)
        val view = inflater.inflate(viewType, parent, false)
        val holder = RecyclerViewHolder(view)
        holder.itemView.setOnClickListener {
            val position = holder.adapterPosition
            if (position != RecyclerView.NO_POSITION) {
                mOnItemClickListener?.onItemClick(it, position)
            }
        }
        holder.itemView.setOnLongClickListener {
            val position = holder.adapterPosition
            if (position != RecyclerView.NO_POSITION) {
                mOnItemLongClickListener?.onItemLongClick(it, position)
            }
            false
        }
        return holder
    }

    override fun getItemCount(): Int {
        return if (this.isEnableLoadMore) this.models.size + 1 else this.models.size
    }

    override fun onBindViewHolder(holder: RecyclerViewHolder, position: Int) {
        if (getItemViewType(position) == R.layout.rv_load_more) {
            bindLoadMoreViewHolder(holder)
        } else {
            convert(holder, this.models[position])
        }
    }

    private fun bindLoadMoreViewHolder(holder: RecyclerViewHolder) {
        holder.setText(R.id.load_status,
                when (this.mLoadStatus) {
                    LOAD_FAILURE -> R.string.load_failure
                    LOAD_COMPLETE -> R.string.load_complete
                    else -> R.string.loading
                })
                .setVisibility(R.id.load_progress,
                        if (this.mLoadStatus == LOADING) View.VISIBLE else View.GONE)
    }

    override fun onViewAttachedToWindow(holder: RecyclerViewHolder) {
        if (this.isEnableLoadMore && holder.adapterPosition == itemCount - 1) {
            this.mOnLoadMoreListener?.onLoadMore()
        }
    }

    /**
     * 返回的是当前item的布局id
     */
    protected abstract fun getItemViewLayout(position: Int): Int

    protected abstract fun convert(holder: RecyclerViewHolder, model: M)

    interface OnLoadMoreListener {
        fun onLoadMore()
    }

    interface OnItemClickListener {
        fun onItemClick(view: View, position: Int)
    }

    interface OnItemLongClickListener {
        fun onItemLongClick(view: View, position: Int): Boolean
    }

}
```

``` bash
open class RecyclerViewHolder(view: View) : RecyclerView.ViewHolder(view) {

    private val cache = SparseArray<View>()
    val context: Context by lazy { itemView.context }

    @Suppress("UNCHECKED_CAST")
    fun <V : View> getView(@IdRes id: Int): V {
        var view = cache.get(id)
        if (null == view) {
            view = itemView.findViewById(id)
            cache.put(id, view)
        }
        return view as V
    }

    fun setText(@IdRes id: Int, @StringRes text: Int): RecyclerViewHolder {
        getView<TextView>(id).setText(text)
        return this
    }

    fun setText(@IdRes id: Int, text: CharSequence): RecyclerViewHolder {
        getView<TextView>(id).text = text
        return this
    }

    fun setImageResource(@IdRes id: Int, @DrawableRes resId: Int): RecyclerViewHolder {
        getView<ImageView>(id).setImageResource(resId)
        return this
    }

    fun setImageBitmap(@IdRes id: Int, bitmap: Bitmap): RecyclerViewHolder {
        getView<ImageView>(id).setImageBitmap(bitmap)
        return this
    }

    fun setBackgroundColor(@IdRes id: Int, @ColorInt color: Int): RecyclerViewHolder {
        getView<View>(id).setBackgroundColor(color)
        return this
    }

    fun setBackgroundResource(@IdRes id: Int, @DrawableRes resId: Int): RecyclerViewHolder {
        getView<View>(id).setBackgroundResource(resId)
        return this
    }

    fun setVisibility(@IdRes id: Int, visibility: Int): RecyclerViewHolder {
        getView<View>(id).visibility = visibility
        return this
    }

    fun setEnable(@IdRes id: Int, enable: Boolean): RecyclerViewHolder {
        getView<View>(id).isEnabled = enable
        return this
    }
}
```