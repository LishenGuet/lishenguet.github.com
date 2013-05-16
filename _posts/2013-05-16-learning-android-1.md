---
layout: post
title: "learning Android 1"
description: "学习android系列1"
category: Android
tags: [Activity,AsyncTask,TextWatcher,AndroidApiForWeibo]
---
{% include JB/setup %}

## 开始Android

最近在看Android应用程序开发，环境搭建是去年10月份的事情了，但是因为Paper，Blog，给本科生上Access数据库等一系列琐碎的事情，直到今年4月份从南宁回来才认真的看了几天。期间的学习思路是这样的：界面> Activity > Prefs > 服务 > Database > 列表适配器 > 广播Broadcast Receiver > 内容提供其ContentProvider > 系统服务。如果学习的话大家可以参考Learning Android这本书，可以很快的上手。整个书是围绕一个Twitter的应用程序Yamba来讲解，针对每一章都在增加功能，重构代码。但这本书还是要求有一定的编程和Java基础，能够写个Android的HelloWord等程序的基础。等到学完之后不仅对Android的开发入了门，对代码重构这件事理解深入。

## 题外话

- **我的博客从旧版到新版的改变，就耗了很长时间，有很多都是不规范、重构困难、注释不全导致了很大的工作量。** 

- **对于学习来说，写了代码，但可能还有许多不理解的地方，写技术类的博客也是一种学习，姑且叫做知识梳理吧。**

- **对于博客来说还是内容至上，鉴于师弟也说一天一博，希望自己尽量坚持。**

## 系列1主要内容

- 新浪微博AndroidAPI,涉及认证
- Android的界面布局
- Activity(OnClickListener,TextWatcher和AsyncTask)

**新浪微博AndroidAPI**


**工程大致结构**

1. 应用程序运行，先加载配置文件manifest文件，将application里面声明的activity，server，receiver等都加载进去，并且会找到程序的入口Activity，这里不多做解释，只是为了保持完整性。 
2. 程序中要引入Twitter的jar包，并且需要连网，所以要在manifest配置文件里面声明程序的连网权限`<uses-permission android:name="android.permission.INTERNET"/>`
3. 每个程序都会有string.xml来存放常用的字符串，例如我们声明的项目名称，APP名称，这样避免写死在项目里，易于修改；每个程序都会有layout目录，放一些相对应的程序中的界面，每一个界面对应一个布局文件，布局文件中有声明id的会自动生成在R文件中；如果我们用到图片就放在drawable目录里，音频就放在raw目录里，程序一般都是有菜单的，菜单布局文件一般放在menu里。
4. 剩下的就是代码了，有java基础的一看就懂。

**界面布局文件**

有多种布局方式：线性的（默认是横向Horizontal）、相对布局（利用id放置），个人比较熟悉这两种。

`android:layout_weight="1"`：表示布局的权重，默认值是0，设定高度如果是`match_parent`，那么加载界面的时候该控件下面的空间都会被挤出屏幕之外，设置为1时，会在与其他空间和谐相处的时候占据最大空间。

`android:layout_gravity="right"` 表示控件在布局中所在的水平位置和垂直位置，针对该控件。

`android:gravity="center"` 表示控件的内容在控件中的位置，针对控件内容。


**Activity(生命周期，线程机制，事件监听)**

- **生命周期**

这个事情对于任意一个Activity或者是Server等都是必要的，要学会在onCreate里面做初始化工作，在onStart启动后，要在onResume里面做实际操作，onPause是指可能其它的Activity放它前面，但是该Activity并没有消失，可以恢复到onResume，也可能由于资源问题被废了到onStop，它指的是activity已经没有了，恢复就要onCreate里面重新创建，还有一个onDestroy状态是指将Activity销毁。熟话说：字不如表，表不如图，来吧：

![Alt 生命周期](http://hi.csdn.net/attachment/201007/28/0_12803210018q71.gif)

- **线程机制**

Android默认是在单线程下运行，这意味着上一个操作没有执行完，下一个操作不会执行。这个单线程也是UI线程，即用户在界面上操作都在这执行，除了渲染界面还有相应用户事件等。这样有一个问题，比如我们连网操作可能因为环境问题导致是一个费时的操作，这种费时会引发android的假死，所以我们需要引入多线程，让一些长时间运行的操作放在独立线程。这里主要学习了Thread()、Handler()和AsyncTask()两种方法。这里就拿网络连接举例：

Thread()+Handler()：

对于上面的这种方法有存在缺陷的：

+ 线程的开销比较大，每个任务都建立一个线程会造成很大开销
+ 线程无法管理，匿名的线程创建启动后不受控制，发送多个请求则会启动非常多线程
+ 还要引入Handler这样代码比较臃肿

AsyncTask()的特点是在主线程之外运行，并且回调方法在主线程中执行，避免了Handler的麻烦，也解决了匿名线程存在的问题。

- **事件监听**