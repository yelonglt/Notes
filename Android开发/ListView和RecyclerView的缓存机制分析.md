#ListView缓存机制分析
详细信息可以阅读AbsListView.RecycleBin类

1. RecycleBin里面维护两个数组。mActiveViews数组里面记录的是当前屏幕展示的视图；mScrapViews记录的是移除屏幕的视图，mScrapViews是一个二维数组，getViewTypeCount决定数组的长度，getItemViewType决定相同类型的View放到那个坐标下。初始化方法为RecycleBin.setViewTypeCount()
2. View复用的核心方法是AbsListView.obtainView()。
   * 根据position获取scrapView。
   * 如果scrapView不为null，我们就可以利用convertView进行复用操作。
   * 如果mAdapter.getView(position, scrapView, this)返回的View和我们从缓存池中拿出的View不同，则把它重新存进去。
   * 当scrapView为nul，mAdapter.getView(position, null, this)，传递convertView为null，适配器会新建一个View。

3. ListView的缓存机制利用了生产者和消费者的原理，View滑出屏幕时把它缓存起来，当下一个View滑入时，如果能在缓存池中找到，则把它取出来(从缓存池中remove掉了)复用。

