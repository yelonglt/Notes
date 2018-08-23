#ListView缓存机制分析
详细信息可以阅读AbsListView.RecycleBin类

1. RecycleBin里面维护两个数组。mActiveViews数组里面记录的是当前屏幕展示的视图；mScrapViews记录的是移除屏幕的视图，mScrapViews是一个二维数组，getViewTypeCount决定数组的长度，getItemViewType决定相同类型的View放到那个坐标下。初始化方法为RecycleBin.setViewTypeCount()
2. View复用的核心方法是AbsListView.obtainView()。
   * 根据position获取scrapView。
   * 如果scrapView不为null，我们就可以利用convertView进行复用操作。
   * 如果mAdapter.getView(position, scrapView, this)返回的View和我们从缓存池中拿出的View不同，则把它重新存进去。
   * 当scrapView为nul，mAdapter.getView(position, null, this)，传递convertView为null，适配器会新建一个View。

3. ListView的缓存机制利用了生产者和消费者的原理，View滑出屏幕时把它缓存起来，当下一个View滑入时，如果能在缓存池中找到，则把它取出来(从缓存池中remove掉了)复用。

#RecycleView缓存机制分析
详细信息可以阅读RecyclerView.Recycler类。RecyclerView拥有三级缓存（算上mAdapter.createViewHolder的话就是四级了）。分析一下它的主要成员变量的用处。

1. mAttachedScrap--未与RecyclerView分离的ViewHolder列表（即一级缓存）。不需要回调createView和bindView，用于屏幕内ItemView快速重用。
2. mChangedScrap--RecyclerView中需要改变的ViewHolder列表（即一级缓存）。
3. mCachedViews--RecyclerView的ViewHolder缓存列表（即一级缓存）。不需要回调createView和bindView，默认上限为2，即缓存屏幕外2个ItemView。可以调用mRecyclerView.setItemViewCacheSize(5)设置大小。
4. mViewCacheExtension--用户设置的RecyclerView的ViewHolder缓存列表扩展（即二级缓存）。不直接使用，需要用户定制，默认不实现。
5. mRecyclerPool--RecyclerView的ViewHolder缓存池（即三级缓存），在有限的mCachedViews中如果存不下ViewHolder时，就会把ViewHolder存入mRecyclerPool中。不需要回调createView，但是需要回调bindView。默认上限是5，技术上可实现所有的RecyclerViewPool共用同一个。

###获取缓存
分析tryGetViewHolderForPositionByDeadline()方法

1. 从mChangedScrap中获取ViewHolder，未成功执行步骤2
2. 从mAttachedScrap中获取ViewHolder，未成功执步骤3
3. 从mCachedViews中获取ViewHolder，未成功执步骤4
4. 如果mViewCacheExtension不为空，从mViewCacheExtension中获取ViewHolder，未成功执步骤5
5. 从mRecyclerPool中获取ViewHolder，未成功执步骤6
6. mAdapter.createViewHolder()创建一个指定类型的ViewHolder。判断是否bound，未bound或holder需要更新或无效，执行mAdapter.bindViewHolder()。此方法内部会调用onBinderViewHolder()方法。

###简析RecyclerViewPool
RecyclerViewPool是由SparseArray实现，其中包含了多个ViewType对应的ArrayList集合。获取时通过ViewType得到对应的ArrayList集合，之后返回一个ViewHolder，并在集合中删除这个ViewHolder。添加也是类似，发现没有ViewType对应的ArrayList集合时，将进行创建，并且在比较大小不超过默认大小5时添加。

性能优化建议，譬如一个页面使用RecyclerView嵌套时，在外层RecyclerView的适配其中新建一个视图池new RecyclerView.RecycledViewPool()，然后在外层适配器的OnCreateViewHolder方法中给你子RecyclerView设置视图池。