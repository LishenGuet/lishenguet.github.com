---
layout: post
title: "Preference详解"
description: "对Android的学习，SharedPreference全面学习"
category: 技术
tags: [Android,学习,技术]
---
{% include JB/setup %}

很多程序都有个性化设置，这个地方如果用Preference来设置会更好，比如说要不要用wlan还是其他网络连接，存放自己的用户名与密码等等。在Android系统源码中，绝大多数应用程序的UI布局采用了Preference的布局结构，而不是我们平时在模拟器中构建应用程序时使用的View布局结构。**Preference的优点在于布局界面的可控性和高效率以及可存储值的简洁性**(每个PreferenPreferencece存储在相对应下的SharedPreference文件夹下)。 

##Preference的控件

###单一控件

1. preference是文本框
2. CheckPreference是单选框
3. EditTextPreferece输入文本框
4. ListPreference列表框
5. RingtonePreference是铃声

###组合控件

1. PreferenceCategory用于组合一组Preference，使布局有层次感
2. PreferenceScreen是所有Preference元素的根节点

###控件属性说明

每一个Preference都有一个对应的xml文件，在xml里面还有一些Attributes属性需要说明：

1. android:key 每一个控件独一无二的“ID”,唯一标示此Preference
2. android:defaultValue 默认值
3. android:enabled 该控件是可用状态
4. android:title 每个控件显示的大标题
5. android:summary 每个控件显示的小标题
6. android:persistent 控件对应的值是否写入sharedPreference文件中，如果是true,则表示写
7. android:dependency 表示一个控件的可用状态依赖另一个控件
8. android:disableDependentState 与7相反，有你没我，有我没你。

###举例
  
    <PreferenceCategory android:title="无线和网络设置"></PreferenceCategory> 

    <CheckBoxPreference android:key="apply_wifi"  
        android:title="Wi-Fi" android:summary="打开Wi-Fi">  
    </CheckBoxPreference> 

    <Preference android:key="wifi_setting" android:title="Wi-Fi设置"  
        android:summary="设置和管理无线接入点" android:dependency="apply_wifi">  
        <!-- 点击时 自定义一个默认跳转Intent  action指定隐式Intent -->  
        <!-- action指定隐式Intent -->  
        <intent android:action="com.example.testandlearning.seemAction" />    
    </Preference>  

    <EditTextPreference android:key="number_edit"  
        android:title="输入电话号码" android:defaultValue="123">  
    </EditTextPreference> 

    <ListPreference android:key="depart_value"  
        android:title="部门设置" android:dialogTitle="选择部门" android:entries="@array/department"  
        android:entryValues="@array/department_value">  
    </ListPreference>  

    <RingtonePreference android:key="ring_key"  
        android:title="铃声" android:ringtoneType="all" android:showDefault="true"  
        android:showSilent="true">  
    </RingtonePreference>  

###对于EditPreference

getEditText()  返回的是我们在该控件中输入的文本框值

getText()     返回的是我们之前sharedPreferen文件保存的值

###对于ListPreference

1. android:dialogTitle：弹出控件对话框时显示的标题
2. android:entries：类型为array，控件欲显示的文本
3. android:entryValues：类型为array，与文本相对应的key-value键值对，value保存至sharedPreference文件

上面例子中的array即是有一个xml文件与之对应

        <resources>  
            <string-array name="department">  
                <item>IT</item>  
                <item>Commerce</item>  
                <item>HR</item>  
            </string-array>  
            <string-array name="department_value">  
                <item>001</item>  
                <item>002</item>  
                <item>003</item>  
            </string-array>  
        </resources> 

- CharSequence[]  getEntries（）返回的是控件显示文本的一个”key”数组，对应于属性android:entries
- CharSequence[]  getEntryValues（）返回的一个”value”数组，对应于属性android: entryValues
- CharSequence    getEntry（）: 返回当前选择文本
- String          getValue（）:返回当前选中文本选中的value 

###对于RingtonePreference

- android:ringtoneType：响铃的铃声类型，主要有：ringtone(音乐)、notification(通知)、alarm(闹铃)、all(所有可用声 音类型)。
- android:showDefault：默认铃声，可以使用系统(布尔值---true,false)的或者自定义的铃声
- android:showSilent：指定铃声是否为静音。指定铃声包括系统默认铃声或者自定义的铃声

##Preference的跳转

对于上面的这一行代码：

        <Preference android:key="wifi_setting" android:title="Wi-Fi设置"android:summary="设置和管理无线接入点"android:dependency="apply_wifi">   
            <!-- action指定隐式Intent -->  
            <intent android:action="com.example.testandlearning.seemAction" />    
        </Preference>

可以在对某一个控件点击的时候进行跳转到目标Intent，这有以下方法，首先要明确Intent有两种方式，隐示与显示（最后解释）。

1. 通过在xml配置文件中说明
2. 通过onPreferenceTreeClick()创建新的intent显示的进行跳转

这两种就牵涉到了Preference控件的事件监听了，这两种方法相互也是有影响的，之间的顺序是因程序不同而发生变化。

##Preference事件

**boolean onPreferenceTreeClick**如果返回true，则不执行默认动作或返回上层调用链，即是不再去执行xml里面的intent了。如果返回false,执行默认动作，返回上层调用链。

**boolean onPreferenceClickListener**在点击的时候触发，可以做相应的操作。

**boolean onPreferenceChangeListener**当控件的值发生变化时触发，true表示将新值写入preference文件，false表示不写入新值。

**触发规则**先调用onPreferenceClick（）方法，如果该方法返回true，则不再调用onPreferenceTreeClick方法；如果onPreferenceClick方法返回false，则继续调用onPreferenceTreeClick方法。onPreferenceChange的方法独立与其他两种方法的运行。也就是说，它总是会运行。

注意，点击控件后，是先回调onPreferenceChange（）方法，即先确定是否保存值。在调用Click这些方法。

##SharedPreference的用法
    
        SharedPreferences prefs;
        prefs = PreferenceManager.getDefaultSharedPreferences(this);
        //将当前的上下文作为this来获取实例
        prefs.registerOnSharedPreferenceChangeListener(this);
        //数据更改时，有一个机制来通知，相当于按钮注册一个触发事件，
        //相应的要完成最底下的onSharedPreferenceChanged方法
        String username,password,apiRoot;
        username=prefs.getString("username", "");
        password=prefs.getString("password", "");
        apiRoot=prefs.getString("apiRoot", "");
        boolean wifiChecked=prefs.getBoolean("wifi", false);  
        //如果第2个参数不存在时,备用默认值,这里的默认值是空
        @Override
        public void onSharedPreferenceChanged(SharedPreferences arg0, String arg1) {
            twitter=null;
            //每次改变，都会重新生成，然后去拿prefs里面的文件   
        }

##Intent的显示与隐式

###显示

显示的Intent不用讲，即是在一个应用之间，不同的Activity进行切换时用到的Intent机制。我们要用Intent确定启动具体哪一个Activity。

        //显示方式声明Intent，直接启动SecondActivity  
        Intent it=new Intent(MainActivity.this,EgActivity.class);  
        //启动Activity  
        startActivity(it); 

###隐式

比如说我在某个App例如新浪微博中点击拍照，会跳转到照相机的界面。但是当我新装了camer360或之类的第三方照相app的时候在微博中点击照相就会先弹出一个Dialog来让我选择是使用默认camer还是camer360.这个地方就要用到隐式Intent。如`Intent intent = new Intent("com.example.testandlearning.seemAction");`

这里我们用action来实现。

在mainfest.xml文件里，要使用这样的activity声明

        <activity android:name=".CActivity"  
                android:label="@string/title_activity_main" >  
            <intent-filter>  
                <action android:name="com.example.testandlearning.seemAction"></action>    
                <category android:name="android.intent.category.DEFAULT"></category>   
            </intent-filter>  
        </activity> 

即是这个intent-filter起着作用。 

我们在MainActivity中实现一个Button，点击发送:`new Intent("com.example.testandlearning.seemAction");`此时就会弹出一个Dialog让我们自动选择是使用BActivity还是使用CActiviy了，就如前文我提到camer360的那个例子。

在xml文件里的控件中就像上面一样就应该写成这样`<intent android:action="com.example.testandlearning.seemAction" />`




