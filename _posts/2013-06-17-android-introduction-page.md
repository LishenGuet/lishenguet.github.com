---
layout: post
title: "Android模仿微信引导页"
description: "引导页对用户体验越发重要,告知用户软件的更新功能,帮助用户更流畅的使用软件"
category: 技术
tags: [技术, 学习, Android]
---
{% include JB/setup %}

以微信为例，我们在首次打开微信的时候会有一系列很好的引导页，以帮助读者更好的了解微信，能够更顺畅的使用。所以一个好的引导页对用户体验是非常重要的。前一阵子学习了相关的代码，并做了一些笔记，这里分享给大家。同时也作为我的温故而知新的过程。在学习的过程中主要参考了Android的SDK与一些博客。

首先引导页的布局文件里面我们用到了ViewGroup，它是View的子类，ViewGroup的直接已知子类有`AbsoluteLayout, AdapterView, FrameLayout, LinearLayout, RelativeLayout, SlidingDrawer `，并且它还可以是很多layout的组合。我们用一个RelativeLayout，里面包含了三个Layout，一个是自己定义的用来显示多个图片；一个是下面的小点，来指示当前是第几个图片；最后一个是一个淡入淡出的效果来从引导页进入主界面。

##自己定义的多图片

    <edu.lishen.guangxitravel.MyScrollLayout 
        xmlns:android="http://schemas.android.com/apk/res/android"
        android:id="@+id/scrollLayout"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:visibility="visible">
        <RelativeLayout android:background="@drawable/w01"
            android:layout_width="match_parent"
            android:layout_height="match_parent">
            <TextView android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_alignParentBottom="true"
                android:layout_centerHorizontal="true"
                android:layout_marginBottom="89dp"
                android:text="微信，不只是个聊天工具"
                android:textColor="@color/TextColor"
                android:textSize="18sp"/>
        </RelativeLayout>
        <RelativeLayout android:background="@drawable/w02"
            android:layout_width="match_parent"
            android:layout_height="match_parent">
            <TextView android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_alignParentBottom="true"
                android:layout_centerHorizontal="true"
                android:layout_marginBottom="96dp"
                android:text="第一次，你可以使用透明背景的动画表情，来表达你此刻的心情"
                android:gravity="center_horizontal"
                android:textColor="@color/TextColor"
                android:textSize="18sp"/>
        </RelativeLayout>
        <RelativeLayout android:background="@drawable/w03"
            android:layout_width="match_parent"
            android:layout_height="match_parent">
        </RelativeLayout>
        <RelativeLayout android:background="@drawable/w04"
            android:layout_width="match_parent"
            android:layout_height="match_parent">
        </RelativeLayout>
        <RelativeLayout android:background="@drawable/w05"
            android:layout_width="match_parent"
            android:layout_height="match_parent">
        </RelativeLayout>
        <RelativeLayout android:background="@drawable/w06"
            android:layout_width="match_parent"
            android:layout_height="match_parent">
        </RelativeLayout>
        <RelativeLayout android:background="@drawable/w07"
            android:layout_width="match_parent"
            android:layout_height="match_parent">
        </RelativeLayout>  
        <RelativeLayout android:background="@drawable/w08"
            android:layout_width="match_parent"
            android:layout_height="match_parent">
        </RelativeLayout> 
        <RelativeLayout android:background="@drawable/w01"
            android:layout_width="match_parent"
            android:layout_height="match_parent">
            <Button
              android:id="@+id/startBtn"
              android:layout_width="wrap_content"
              android:layout_height="wrap_content"
              android:layout_alignParentBottom="true"
              android:layout_centerHorizontal="true"
              android:layout_marginBottom="98dp"
              android:text="开始我的微信生活"
              android:textSize="18sp"
              android:textColor="@color/TextColor"
              android:background="@drawable/button_bg"
              android:layout_marginLeft="8dp"
              android:layout_marginRight="8dp"
              android:layout_gravity="center_vertical"
              />
        </RelativeLayout>      
    </edu.lishen.guangxitravel.MyScrollLayout>

##指示点，指示当前第几个图片

    <LinearLayout 
        android:orientation="horizontal"
        android:id="@+id/scrollFlagLayout"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="24.0dip"
        android:layout_alignParentBottom="true"
        android:layout_centerHorizontal="true"
        android:visibility="visible">
        <ImageView android:clickable="true" android:padding="5.0dip" 
            android:layout_width="wrap_content" android:layout_height="wrap_content"
            android:layout_gravity="center_vertical" android:src="@drawable/page_indicator_bg"/> 
        <ImageView android:clickable="true" android:padding="5.0dip" 
            android:layout_width="wrap_content" android:layout_height="wrap_content"
            android:layout_gravity="center_vertical" android:src="@drawable/page_indicator_bg"/> 
        <ImageView android:clickable="true" android:padding="5.0dip" 
            android:layout_width="wrap_content" android:layout_height="wrap_content"
            android:layout_gravity="center_vertical" android:src="@drawable/page_indicator_bg"/> 
        <ImageView android:clickable="true" android:padding="5.0dip" 
            android:layout_width="wrap_content" android:layout_height="wrap_content"
            android:layout_gravity="center_vertical" android:src="@drawable/page_indicator_bg"/> 
        <ImageView android:clickable="true" android:padding="5.0dip" 
            android:layout_width="wrap_content" android:layout_height="wrap_content"
            android:layout_gravity="center_vertical" android:src="@drawable/page_indicator_bg"/> 
        <ImageView android:clickable="true" android:padding="5.0dip" 
            android:layout_width="wrap_content" android:layout_height="wrap_content"
            android:layout_gravity="center_vertical" android:src="@drawable/page_indicator_bg"/>
        <ImageView android:clickable="true" android:padding="5.0dip" 
            android:layout_width="wrap_content" android:layout_height="wrap_content"
            android:layout_gravity="center_vertical" android:src="@drawable/page_indicator_bg"/> 
        <ImageView android:clickable="true" android:padding="5.0dip" 
            android:layout_width="wrap_content" android:layout_height="wrap_content"
            android:layout_gravity="center_vertical" android:src="@drawable/page_indicator_bg"/> 
        <ImageView android:clickable="true" android:padding="5.0dip" 
            android:layout_width="wrap_content" android:layout_height="wrap_content"
            android:layout_gravity="center_vertical" android:src="@drawable/page_indicator_bg"/>       
    </LinearLayout>

这里用到了`android:src="@drawable/page_indicator_bg"`对于Drawable Resources支持很多种形式，例如`Bitmap File，Nine-Patch File，Layer List，State List等`可以参考android的API
这里我们的`@drawable/page_indicator_bg`文件如下

    <?xml version="1.0" encoding="utf-8"?>
    <selector
        xmlns:android="http://schemas.android.com/apk/res/android">
        <item android:state_enabled="true"  android:drawable="@drawable/page_indicator_unfocused" />
        <item android:state_enabled="false" android:drawable="@drawable/page_indicator_focused" />
    </selector>

##由引导页到主界面的淡入淡出效果

    <LinearLayout android:id="@+id/animLayout"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:visibility="gone">
        <LinearLayout
            android:id="@+id/leftLayout" 
            android:layout_width="wrap_content"
            android:layout_height="match_parent">
        <ImageView android:layout_height="wrap_content"
            android:layout_width="wrap_content"
            android:src="@drawable/whatsnew_left"/>
        <ImageView android:layout_height="wrap_content"
            android:layout_width="wrap_content"
            android:src="@drawable/whatsnew_left_m"/>
        </LinearLayout>
        <LinearLayout 
            android:id="@+id/rightLayout" 
            android:layout_width="match_parent"
            android:layout_height="match_parent">
        <ImageView android:layout_height="wrap_content"
            android:layout_width="wrap_content"
            android:src="@drawable/whatsnew_right_m"/>
        <ImageView android:layout_height="wrap_content"
            android:layout_width="wrap_content"
            android:src="@drawable/whatsnew_right"/>
        </LinearLayout>
    </LinearLayout>

这里visibility里有`VISIBLE：设置控件可见，INVISIBLE：设置控件不可见，GONE：设置控件隐藏`对于INVISIBLE和GONE的主要区别是：当控件visibility属性为INVISIBLE时，界面保留了view控件所占有的空间；而控件属性为GONE时，界面则不保留view控件所占有的空间。

##设置全屏模式

` android:theme="@android:style/Theme.NoTitleBar.Fullscreen"`如果只是想设置显示视图没有title而不隐藏状态栏的，可以使使用

    在AndroidManifest里面
    android:theme="@android:style/Theme.NoTitleBar"
    在onCreate函数里
    getWindow().setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN,
                WindowManager.LayoutParams.FLAG_FULLSCREEN);

##对于最后的淡入淡出效果:  

    Animation leftOutAnimation = AnimationUtils.loadAnimation
        (getApplicationContext(),  R.anim.translate_left);
    Animation rightOutAnimation = AnimationUtils.loadAnimation
        (getApplicationContext(),  R.anim.translate_right);
    leftLayout.setAnimation(leftOutAnimation);
    rightLayout.setAnimation(rightOutAnimation);
    leftOutAnimation.setAnimationListener(new AnimationListener(){
        @Override
        public void onAnimationEnd(Animation arg0) {
            leftLayout.setVisibility(View.GONE);
            rightLayout.setVisibility(View.GONE);
            Intent intent = new Intent(MainActivity.this,OtherActivity.class);
            MainActivity.this.startActivity(intent);
    一个Activity从内部主动关闭? 那么finish是最通常和正确的方法
    MainActivity.this.finish();
    Activity的切换动画指的是从一个activity跳转到另外一个activity时的动画。
    一部分是第一个activity退出时的动画；
    另外一部分时第二个activity进入时的动画；
    1.它必需紧挨着startActivity()或者finish()函数之后调用"
    2.它只在android2.0以及以上版本上适用  
        }

##对于MyScrollLayout

这是我们自己定义的一个继承ViewGroup的类，这里面我们主要学习速度跟踪VelocityTracker和滑动控制Scroller。

###因为继承ViewGroup，所以要重载onLayout和onMeasure函数

onMeasure方法负责测量将要放在CustomGridLayout内部的View的大小，onLayout方法负责分配尺寸及位置给将要放在CustomGridLayout内部的View。

    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        if(changed){
            int childLeft = 0;
            final int childCount = getChildCount();
            for(int i=0;i<childCount;i++){
                final View childView = getChildAt(i);
                if(childView.getVisibility() != View.GONE){
                    final int childWidth = childView.getMeasuredWidth();
                    childView.layout(childLeft, 0, childLeft+childWidth, childView.getMeasuredHeight());
                    childLeft += childWidth;
                }
            }
        }
    }
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec){
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        final int width = MeasureSpec.getSize(widthMeasureSpec);
        final int widthMode = MeasureSpec.getMode(widthMeasureSpec);
        final int count = getChildCount();
        for(int i=0;i<count;i++){
            getChildAt(i).measure(widthMeasureSpec, heightMeasureSpec);
        }
        scrollTo(curScreen*width,0);
    }

设置本View视图的最终大小，该功能的实现通过调用setMeasuredDimension()方法去设置实际的高对应属性:mMeasuredHeight和宽对应属性：mMeasureWidth。如果该View对象是个ViewGroup类型，需要重写该onMeasure()方法，对其子视图进行遍历的measure()过程。

##不同图片滑动

分两种情况:第一种是跟踪速度，达到目的图片。第二种是如果滑动超过一半，达到目的图片。所以对监听事件onTouchEvent

    public boolean onTouchEvent(MotionEvent event){
        final int action = event.getAction();
        final float x = event.getX();
        final float y = event.getY();
        switch(action){
        case MotionEvent.ACTION_DOWN:
            Log.d("ScrollLayout", "onTouchEvent  ACTION_DOWN");
            if(velocityTracker == null){
                velocityTracker = VelocityTracker.obtain();
                velocityTracker.addMovement(event);
            }
            if(!scroller.isFinished()){
                scroller.abortAnimation();
            }
            lastMotion = x;
            break;
        case MotionEvent.ACTION_MOVE:
            int deltaX = (int)(lastMotion-x);
            if(IsCanMove(deltaX)){
                if(velocityTracker != null){
                    velocityTracker.addMovement(event);
                }
                lastMotion = x;
                scrollBy(deltaX,0);
    //public void scrollBy (int x, int y)，将View的Content偏移(x,y)。x控制左右方向的偏移，y控制上下方向的偏移。
    //例如当x>0，y=0时，向右移动x像素，当x<0，y=0时，向左移动x像素，而View的大小和位置不发生改变。
    //public void scrollTo (int x, int y)，将View的Content的位置移动到(x,y)，而View的大小和位置不发生改变。
    //如果Content超出了View的范围，则超出的部分会被挡住。
            }
            break;
        case MotionEvent.ACTION_UP:
            int velocityX=0;
            if(velocityTracker != null){
                velocityTracker.addMovement(event);
                //设置units的值为1000，意思为一秒时间内运动了多少个像素
                velocityTracker.computeCurrentVelocity(1000);
                //这个速度有正有负 测试x轴的速度
                velocityX = (int) velocityTracker.getXVelocity();
            }
            if(velocityX > SNAP_VELOCITY && curScreen > 0){
                snapToScreen(curScreen - 1);
            }
            else if(velocityX < -SNAP_VELOCITY && curScreen <getChildCount()-1){
                snapToScreen(curScreen + 1);
            }
            else {       
                snapToDestination();       
            }
            if (velocityTracker != null) {       
                velocityTracker.recycle(); 
                //最后要回收
                velocityTracker = null;       
            } 
            break;
        }
        return true;
    }


使用的snapToDestination用来判断滑动是否超过一半
    
    public void snapToDestination(){
        final int screenWidth = getWidth();
        Log.d("ScrollLayout", String.valueOf(getScrollX()));
        final int destScreen = (getScrollX()+screenWidth/2)/screenWidth;
        snapToScreen(destScreen);
    }

使用snapToScreen跳转到相应的图片

    public void snapToScreen(int whichScreen){
        whichScreen = Math.max(0, Math.min(whichScreen, getChildCount()-1));
        if(getScrollX() != (whichScreen * getWidth())){
            final int delta = whichScreen*getWidth()-getScrollX();
            scroller.startScroll(getScrollX(), 0, delta, 0, Math.abs(delta)*2);
            curScreen = whichScreen;//设置当前屏幕
            invalidate();// Redraw the layout  重新构图   
            if (onViewChangeListener != null)
            {
                onViewChangeListener.OnViewChange(curScreen);
            }
        }
    }

startScroll函数功能开始一个动画控制，由(startX , startY)在duration时间内前进(dx,dy)个单位，到达坐标为(startX+dx , startY+dy)处。mScroller是一个封装位置和速度等信息的变量，startScroll（）
函数只是对它的一些成员变量做一些设置，这个设置的唯一效果就是导致mScroller.computeScrollOffset()返回true。

**核心代码就是这些**如果希望整个源程序的，请微博私信我@水手Answer