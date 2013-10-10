---
layout: post
title: "Activity生命周期与栈管理解析"
description: "对Android的学习，Activity的全面了解与栈管理方式"
category: 技术
tags: [Android,学习,技术]
---
{% include JB/setup %}

#Android复习系列1：Activity相关知识

当然谈到四大组件，必不可少的就是Activity，它是应用的脸面，要是丑陋又复杂，那结果可想而知。

##Activity有四个状态

运行、暂停、停止、重启。而Activity是采用栈的形式来管理，当新建的Activity完成就会放置到栈顶，进入运行状态，前一个Activity会自动向栈底移动，知道新的Activity退出，否则不会出现在前台。

1. Activity在栈顶，即屏幕最前面，处于**运行活动状态**；
2. Activity失去焦点，依旧可见，被其他非全屏的对话框遮住，称为**暂停状态**。暂停的Activity依然是活动的，但是系统内存严重不足的时候，可能会被系统结束
3. Activtiy完全不可见，称之为**停止状态**，这时依然保持了它本身的状态及成员信息，不过由于用户不可见，当系统其它地方需要内存时，经常被结束
4. Activtiy由不可见到可见状态，称之为**重启状态**，这时Activity必须迅速恢复它以前的状态。

##Activity生命周期的各个方法

1. onCreate()当Activity首次创建时调用，这里通常的工作是创建视图， 绑定数据到列表等。这个方法还有一个 Bundle 参数，如果这个 Activity之前由冻结的状态，这个状态将包含在里面。之后，通常会接着调用onStart()方法。
2. onRestart()Activity 已经被停止，在其被重新开始之前调用。接下来回调用 onStart()方法。
3. onStart()当 Activity 变到用户可见时调用，接下来如果Activity变成不可见的话，将会调用 onStop()， 否则将调用onResume() 。
4. onResume()当Activity开始能和用户交互时调用，此时的 Activity位于栈顶， 接下来通常会调用onPause() 。
5. onPause()当系统准备开始一个新的Activity或者重置一个已有的 Activity时调用。通常需要在这里进行保存数据、停止动画以及其它占用CPU 资源的活动等。这个方法完成之前，下一个Activity不会继续，所以这个方法的必须较快的完成。接下来如果Activity又回到栈顶将调用onResume()，如果Activity 变的不可见，将调用onStop() 。
6. onStop()当Activity不可见时调用，如果Activity变的可见，将会调用 onRestart()，如果Activity将销毁，调用onDestroy() 。
7. onDestroy()这是Activity被销毁之前最后一次调用，可能是调用了 Activity 的finish()方法，或者系统要回收资源，这两者可以通过isFinishing()方法进行区别。

![alt text](http://beginor.github.io/assets/post-images/activity_lifecycle.png "Title")

##Activity的四种LanchMode模式

LaunchMode在多个Activity跳转的过程中扮演着重要的角色，它可以决定是否生成新的Activity实例，是否重用已存在的Activity实例，是否和其他Activity实例公用一个task里。这里简单介绍一下task的概念，task是一个具有栈结构的对象，一个task可以管理多个Activity，启动一个应用，也就创建一个与之对应的task。

Activity一共有以下四种LaunchMode:

1. standard
2. singleTop
3. singleTask
4. singleInstance

standard:每次都需要启动一个新的Activity,尽管之前的Activity还有。

singleTop:如果有对应的Activity实例正位于栈顶，则重复利用，不生成新的实例。

singleTask:如果该Task里面有对应的Activity实例，则此Activity实例之上的其他Activity实例统统出栈，使此Activity实例成为栈顶对象，显示到幕前。

singleInstance:这种启动模式比较特殊，因为它会启用一个新的栈结构，将Acitvity放置于这个新的栈结构中，并保证不再有其他Activity实例进入。例如A栈有一个实例a1生成了一个Instance的实例，则B栈也有一个实例b，这个时候B栈Activity在启动一个Activtiy，则A栈增加了一个a2，现在按退出则把A的之前的a1退出，然后a2退出，然后退到b。

singleInstance:另一种更方便的解释，在当前栈点了返回，则优先在当前栈退完所有的Activity，接着再退最近活跃的那个栈。

具体的例子与图解可以参照这篇文章[Activvity四种启动模式](http://blog.csdn.net/liuhe688/article/details/6754323 "Activvity四种启动模式").

##Activity的Task相关

每一个应用都有一个Task,Task之间是相互独立的，在Home键回到主屏之后，Activity就会转移到幕后，而新的Activity就会到幕前。当前按后退键时就是针对当前的Activity。

1. Activity的affinity
2. Intent几种常见的flags
3. activity与task相关属性

###affinity

task对于activity就是它的身份证，拥有相同的affinity的多个activity理论同属于一个task,task的affinity取决于根Activity的affinity的值。affinity的应用场合有哪些？1.根据affinity重新为Activity选择宿主task（与allowTaskReparenting属性配合工作）；2.启动一个Activity过程中Intent使用了`FLAG_ACTIVITY_NEW_TASK`标记，根据affinity查找或创建一个新的具有对应affinity的task。我们会在后面进行详细讲解。

默认情况下，一个应用内的所有Activity都具有相同的affinity，都是从Application（参考application的taskAffinity属性）继承而来，而Application默认的affinity是manifest中的包名，我们可以为application设置taskAffinity属性值，这样可以应用到application下的所有activity，也可以单独为某个Activity设置taskAffinity。例如：在系统自带的Browser中，package为com.android.browser，但是application却自定义一个taskAffinity属性值：

        <application   android:name="Browser"  
               android:label="@string/application_name"  
               android:icon="@drawable/ic_launcher_browser"  
               android:backupAgent=".BrowserBackupAgent"  
               android:taskAffinity="android.task.browser" >

如果想替换一个通知、快捷键或者其他能从外部启动的应用程序的内部活动，开发者需要在想替换的活动里明确地设置任务亲和力（Task Affinity）。例如，如果想替换联系人详细信息浏览界面（用户可以直接操作或者通过快捷方式调用），开发者需要设置任务亲和力（Task Affinity）为"com.android.contacts"。 上面就是想替换浏览器。

###Intent几种常见的flags

1. `FLAG_ACTIVITY_NEW_TASK` 当Intent对象包含这个标记时，系统会寻找或创建一个新的task来放置目标Activity，寻找时依据目标Activity的taskAffinity属性进行匹配，如果找到一个task的taskAffinity与之相同，就将目标Activity压入此task中，如果查找无果，则创建一个新的task，并将该task的taskAffinity设置为目标Activity的taskActivity，将目标Activity放置于此task。注意，如果同一个应用中Activity的taskAffinity都使用默认值或都设置相同值时，应用内的Activity之间的跳转使用这个标记是没有意义的，因为当前应用task就是目标Activity最好的宿主。

例如两个应用A与B。每个应用都有两个Activity,分别为Aa,Ab和Ba,Bb。现在我希望在Ba上直接启动Ab，如果没有`FLAG_ACTIVITY_NEW_TASK`，那么在B的这个task里面就会有一个Ab进入到栈顶。这个时候退屏，再分别进入B与A会发现，B显示的是Ab,A显示的是Aa。所以对A没有任何影响。如果有NEW_TASK,那么系统就会根据A的affintiy新建一个task，这样Ba与Ab就不在一个Task,退屏打开A和B的时候会发现，A的直接就在Ab上，而B的像刚打开一样。

2. `FLAG_ACTIVITY_CLEAR_TOP` 当Intent对象包含这个标记时，如果在栈中发现存在Activity实例，则清空这个实例之上的Activity，使其处于栈顶。这个SecondActivity既可以在onNewIntent()中接收到传来的Intent，也可以把自己销毁之后重新启动来接受这个Intent。在使用默认的“standard”启动模式下，如果没有在Intent使用到`FLAG_ACTIVITY_SINGLE_TOP`标记，那么它将关闭后重建，如果使用了这个`FLAG_ACTIVITY_SINGLE_TOP`标记，则会使用已存在的实例；对于其他启动模式，无需再使用`FLAG_ACTIVITY_SINGLE_TOP`，它都将使用已存在的实例，Intent会被传递到这个实例的onNewIntent()中。

不同之处在于：一个是用原来的，一个是销毁了重建，代码不同在于，本身该Activity是standard模式：

        Intent intent = new Intent(this, SecondActivity.class);  
        intent.setFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP);  
        startActivity(intent); 

        Intent intent = new Intent(this, SecondActivity.class);  
        intent.setFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP | Intent.FLAG_ACTIVITY_SINGLE_TOP);  
        startActivity(intent); 

3. `FLAG_ACTIVITY_SINGLE_TOP` 当task中存在目标Activity实例并且位于栈的顶端时，不再创建一个新的，直接利用这个实例。我们在上边的例子中也有讲到。

4. `FLAG_ACTIVITY_REORDER_TO_FRONT` 在多个activity中，`FLAG_ACTIVITY_SINGLE_TOP`会finish其他activity，而这个不会。这个会将已有的Activity重新放到栈顶。
但是重新回到栈顶可能传递的参数可能没有意义，因为intent没有更新。所以在接受的时候应该更新intent。例如：
        
        //需要重写这个方法，并且设置intent
        protected void onNewIntent(Intent intent) {
            super.onNewIntent(intent);
            setIntent(intent);
        }
        @Override
        protected void onResume() {
            super.onResume();
            int num = getIntent().getIntExtra("num", 0);
            Log.e("Num", "" + num);
            String s = getIntent().getStringExtra("hello" + num).toString();
            Log.e("A1", s);
        }

###activity与task相关属性

1. android:allowTaskReparenting 这个属性用来标记一个Activity实例在当前应用退居后台后，是否能从启动它的那个task移动到有共同affinity的task，“true”表示可以移动，“false”表示它必须呆在当前应用的task中，默认值为false。如果一个这个Activity的<activity>元素没有设定此属性，设定在<application>上的此属性会对此Activity起作用。例如在一个应用中要查看一个web页面，在启动系统浏览器Activity后，这个Activity实例和当前应用处于同一个task，当我们的应用退居后台之后用户再次从主选单中启动应用，此时这个Activity实例将会重新宿主到Browser应用的task内，在我们的应用中将不会再看到这个Activity实例，而如果此时启动Browser应用，就会发现，第一个界面就是我们刚才打开的web页面，证明了这个Activity实例确实是宿主到了Browser应用的task内。

2.android:alwaysRetainTaskState 这个属性用来标记应用的task是否保持原来的状态，“true”表示总是保持，“false”表示不能够保证，默认为“false”。此属性只对task的根Activity起作用，其他的Activity都会被忽略。默认情况下，如果一个应用在后台呆的太久例如30分钟，用户从主选单再次选择该应用时，系统就会对该应用的task进行清理，除了根Activity，其他Activity都会被清除出栈，但是如果在根Activity中设置了此属性之后，用户再次启动应用时，仍然可以看到上一次操作的界面。

这个属性对于一些应用非常有用，例如Browser应用程序，有很多状态，比如打开很多的tab，用户不想丢失这些状态，使用这个属性就极为恰当。

3. android:clearTaskOnLaunch 这个属性用来标记是否从task清除除根Activity之外的所有的Activity，“true”表示清除，“false”表示不清除，默认为“false”。同样，这个属性也只对根Activity起作用，其他的Activity都会被忽略。

如果设置了这个属性为“true”，每次用户从桌面回到这个应用时，都只会看到根Activity，task中的其他Activity都会被清除出栈。如果我们的应用中引用到了其他应用的Activity，这些Activity设置了allowTaskReparenting属性为“true”，则它们会被重新宿主到有共同affinity的task中。

4. android:finishOnTaskLaunch 这个属性和android:allowReparenting属性相似，不同之处在于allowReparenting属性是重新宿主到有共同affinity的task中，而finishOnTaskLaunch属性是销毁实例。如果这个属性和android:allowReparenting都设定为“true”，则这个属性胜出。