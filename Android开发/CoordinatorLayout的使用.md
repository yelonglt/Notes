#CoordinatorLayout
***
###CoordinatorLayout的作用
1. 作为顶层布局
2. 调度和协调子布局
3. CoordinatorLayout作为“super-powered FrameLayout”通过调度和协调子布局的形式实现触摸影响布局的形式产生动画效果。CoordinatorLayout通过设置子View的Behavior来实现。

###CoordinatorLayout功能为什么如此强大
1. 神奇之处在于Behavior对象，CoordinatorLayout自己并不控制View，所有的控制权都在Behavior。
2. 系统的Behavior虽然不少，但是更多的需要我们自定义。

###FloatingActionButton
1. 设置位置的属性,有个锚点的概念，通过锚点实现
2. layout_anchor属性是在什么布局上
3. layout_anchorGravity属性设置在依赖布局的位置

```
<android.support.design.widget.FloatingActionButton
        android:id="@+id/fab"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="@dimen/fab_margin"
        app:layout_anchor="@id/app_bar"
        app:layout_anchorGravity="bottom|end"
        app:srcCompat="@android:drawable/ic_dialog_email"/>
        
```

###AppBarLayout嵌套TabLayout
####layout_scrollFlags的属性说明
1. scroll属性，所有想滚动出屏幕的view都需要设置这个flag， 没有设置这个flag的view将被固定在屏幕顶部。例如，TabLayout没有设置这个值，将会停留在屏幕顶部。
2. enterAlways，设置这个flag时，向下的滚动都会导致该view变为可见，启用快速“返回模式”。
3. enterAlwaysCollapsed，当你的视图已经设置minHeight属性又使用此标志时，你的视图只能已最小高度进入，只有当滚动视图到达顶部时才扩大到完整高度。
4. exitUntilCollapsed，滚动退出屏幕，最后折叠在顶端。

####使用说明，为了使得ToolBar有滑动效果，必须要做到这三点
1. CoordinatorLayout作为布局的父布局容器。
2. 给需要滑动的组件设置 app:layout_scrollFlags=”scroll|enterAlways” 属性。
3. 给滑动的组件设置app:layout_behavior属性。譬如给ViewPager设置 app:layout_behavior="@string/appbar_scrolling_view_behavior" 

```
<android.support.design.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/main_content"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <android.support.design.widget.AppBarLayout
        android:id="@+id/appbar"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar">

        <android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            android:background="?attr/colorPrimary"
            app:popupTheme="@style/ThemeOverlay.AppCompat.Light"
            app:layout_scrollFlags="scroll|enterAlways" />

        <android.support.design.widget.TabLayout
            android:id="@+id/tabs"
            android:layout_width="match_parent"
            android:layout_height="wrap_content" />

    </android.support.design.widget.AppBarLayout>

    <android.support.v4.view.ViewPager
        android:id="@+id/viewpager"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior" />

    <android.support.design.widget.FloatingActionButton
        android:id="@+id/fab"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="end|bottom"
        android:layout_margin="@dimen/fab_margin"
        android:src="@drawable/ic_done" />

</android.support.design.widget.CoordinatorLayout>

```
###AppBarLayout嵌套CollapsingToolbarLayout

####CollapsingToolbarLayout使用的地方
1. 这种效果在详情页面用的较多，展示个性化内容，图像有强烈的吸引力。这个效果重点使用了CollapsingToolbarLayout 。
2. CollapsingToolbarLayout可实现Toolbar的折叠效果。
3. CollapsingToolbarLayout的子视图类似与LinearLayout垂直方向排放。

####CollapsingToolbarLayout的属性
1. title,ToolBar的标题，当CollapsingToolbarLayout全屏没有折叠时，title显示的是大字体，在折叠的过程中，title不断变小到一定大小的效果。你可以调用setTitle(CharSequence)方法设置title。
2. contentScrim,ToolBar被折叠到顶部固定时候的背景，你可以调用setContentScrim(Drawable)方法改变背景或者 在属性中使用 app:contentScrim=”?attr/colorPrimary”来改变背景。
3. statusBarScrim,状态栏的背景，调用方法setStatusBarScrim(Drawable)。
4. layout_collapseParallaxMultiplier，CollapsingToolbarLayout滑动时，子视图的视觉差。值得范围0-1，值越大视察越大。
5. layout_collapseMode，子视图的折叠模式，在子视图设置，有两种“pin”：固定模式，在折叠的时候最后固定在顶端；“parallax”：视差模式，在折叠的时候会有个视差折叠的效果。

```
<android.support.design.widget.CoordinatorLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true"
    tools:context="com.dmall.md.MainActivity">

    <android.support.design.widget.AppBarLayout
        android:id="@+id/app_bar"
        android:layout_width="match_parent"
        android:layout_height="@dimen/app_bar_height"
        android:fitsSystemWindows="true"
        android:theme="@style/AppTheme.AppBarOverlay">

        <android.support.design.widget.CollapsingToolbarLayout
            android:id="@+id/toolbar_layout"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:fitsSystemWindows="true"
            app:contentScrim="?attr/colorPrimary"
            app:layout_collapseParallaxMultiplier="0.6"
            app:layout_scrollFlags="scroll|exitUntilCollapsed"
            app:statusBarScrim="@color/colorAccent"
            app:title="HelloWorld">

            <ImageView
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:contentDescription="@string/app_name"
                android:fitsSystemWindows="true"
                android:minHeight="60dp"
                android:scaleType="center"
                android:src="@mipmap/ic_launcher"
                app:layout_collapseMode="pin"
                app:layout_scrollFlags="scroll|enterAlwaysCollapsed"/>

            <android.support.v7.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                app:layout_collapseMode="pin"
                app:popupTheme="@style/AppTheme.PopupOverlay"/>

        </android.support.design.widget.CollapsingToolbarLayout>
    </android.support.design.widget.AppBarLayout>

    <android.support.v4.widget.NestedScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior"
        tools:context="com.dmall.md.MainActivity"
        tools:showIn="@layout/activity_main">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_margin="@dimen/text_margin"
            android:text="@string/large_text"/>

    </android.support.v4.widget.NestedScrollView>

    <android.support.design.widget.FloatingActionButton
        android:id="@+id/fab"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="@dimen/fab_margin"
        app:layout_anchor="@id/app_bar"
        app:layout_anchorGravity="bottom|end"
        app:srcCompat="@android:drawable/ic_dialog_email"/>

</android.support.design.widget.CoordinatorLayout>

```
 


