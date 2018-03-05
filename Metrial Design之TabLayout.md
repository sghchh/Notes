# Material Design 之 TabLayout  
TabLayout的使用非常简单，和Toolbar很像，但是比后者还要简单。  
>Tab的特性：  

>* Tabs 应该在一行内，如果有必要，标签可以显示两行然后截断  
>* Tabs 不应该被嵌套，也就是说一个Tab的内容里不应该包含另一组Tabs  
>* Tabs 控制的显示内容的定位要一致。  
>* Tab 中当前可见内容要高亮显示  
>* Tabs 应该归类并且每组 tabs 中内容顺序相连  
>* 一组 tabs 至少包含 2 个 tab 并且不多于 6 个 tab  

## 使用   
像很多控件一样，TabLayout既可以通过静态使用，也可以通过动态使用；  
### 静态使用  
静态使用TabLayout很简单，在它的布局xml文件中添加TabItem子元素就可以了。  

	<?xml version="1.0" encoding="utf-8"?>
	<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="com.example.as.myapp.gank.GankActivity">
    <android.support.design.widget.TabLayout
        android:id="@+id/activity_gank_tablayout"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:tabIndicatorHeight="2dp"
        app:tabTextColor="@color/gankIndicatorColor"
        app:tabSelectedTextColor="@color/gankIndicatorColor"
        app:tabIndicatorColor="@color/gankIndicatorColor"
        app:tabBackground="@color/colorPrimary"
        app:tabMode="scrollable">
        <android.support.design.widget.TabItem
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="动态"/>
        <android.support.design.widget.TabItem
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="安卓"/>
        <android.support.design.widget.TabItem
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="iOS"/>
        <android.support.design.widget.TabItem
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="福利"/>
        <android.support.design.widget.TabItem
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="all"/>
        <android.support.design.widget.TabItem
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="前端"/>
        <android.support.design.widget.TabItem
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="视频"/>
        <android.support.design.widget.TabItem
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="拓展"/>

    </android.support.design.widget.TabLayout>
	</android.support.constraint.ConstraintLayout>  
上面的只是给每一个Tab设置了文字，也可以设置图片（通过icon这个属性设置），如果两个属性都添加了，那么就是图片与文字都有。  
### 动态使用  
动态使用添加Tab项的做法是通过TabLayout对象的addTab()方法添加，创建一个Tab则是通过TabLayout对象的newTab()方法创建，所以动态添加Tab的方法就是：  

	mTabLayout = (TabLayout) findViewById(R.id.tab_layout2);

        mTabLayout.addTab(mTabLayout.newTab());
          
如果要设置文字和图片的话就是：  

	 mTabLayout = (TabLayout) findViewById(R.id.tab_layout2);

        mTabLayout.addTab(mTabLayout.newTab().setText("个性推荐"));

### Tab选中监听  
Tab切换的时候，我们需要切换页面的内容，这时候就需要为它设置一个监听器TabLayout.OnTabSelectedListener，如下：  

	mTabLayout.addOnTabSelectedListener(new TabLayout.OnTabSelectedListener() {
            @Override
            public void onTabSelected(TabLayout.Tab tab) {
                Log.i(TAG,"onTabSelected:"+tab.getText());
            }

            @Override
            public void onTabUnselected(TabLayout.Tab tab) {

            }

            @Override
            public void onTabReselected(TabLayout.Tab tab) {

            }
        });


### TabLayout进阶属性  
上面讲到了TabLayout 最简单的使用，实际项目中，我们的需求更复杂一些，比如：需要改变Tabs 的指示器的高和颜色，需要改变Tab的宽高，Tab的颜色，固定的Tabs，可滚动的Tabs   

* app:tabBackground 设置Tabs的背景
* app:tabGravity 为Tabs设置Gravity,有两个常量值，GRAVITY_CENTER,GRAVITY_FILL（为center的时候Tab就居中显示；fill就充满）  
* app:tabIndicatorColor 设置指示器的颜色（默认情况下指示器的颜色为colorAccent）
* app:tabIndicatorHeight 设置指示器的高度，Material Design 规范建议是2dp
* app:tabMaxWidth 设置 Tab 的最大宽度
* app:tabMinWidth 设置 Tab 的最小宽度
* app:tabMode 设置Tabs的显示模式，有两个常量值，MODE_FIXED,MODE_SCROLLABLE（一般当Tab数目多的时候设置为Scrollable可滚动模式）。
* app:tabPadding 设置Tab padding
* app:tabPaddingTop/Bottom/Start/End  
* app:tabSelectedTextColor 设置Tab选中后，文字显示的颜色
* app:tabTextColor 设置Tab未选中，文字显示的颜色

