---
layout: post
title: "Android中的多线程异步操作"
description: "对Android的学习，多线程Handler与AsyncTask的具体实现详解"
category: 技术
tags: [Android,学习,技术]
---
{% include JB/setup %}

近期在做Android客户端配合Web端Struts2+Hinbernate+Spring+Json获取实时数据的实验，一个简单的实验，但运行之后报了个没见过的错。`Caused by: android.os.NetworkOnMainThreadException`，谷老师了一下，原因在于Android4.0之后在主线程里面执行Http请求就会报这个错，这一定是怕网络连接过慢而出现Activity假死才这样做的。本来就想认真复习一下这方面的知识，那就开始吧！

上面这个问题，有一个硬方法，但仅推荐测试的时候使用，在`Activity文件的setContentView(R.layout.activity_main)`下面加上如下代码：

        if (android.os.Build.VERSION.SDK_INT > 9) {
            StrictMode.ThreadPolicy policy = new StrictMode.ThreadPolicy.Builder().permitAll().build();
            StrictMode.setThreadPolicy(policy);
        }

切入正题！

***

当进程需要做运算量较大的操作和IO操作，或者主进程UI进程需要连接网络等情况下，我们就要学会异步处理。Handler与AsyncTask都是做多线程异步处理时用到的，两者各有千秋，一一介绍。主要参考[该博客](http://www.fengwenxuan.com/android/2224.html "description with a title").

##两者的共同点

都是可以解决主线程UI异步更新，防止主线程UI的force close问题。都可以实现异步任务机制。能够使得应用保持稳定流畅的运行，并且带来较快的响应。

##两者的异同点

AsynTask是Android的轻量级异步类，可以直接继承AsynTask，在类中实现异步操作，并提供接口反馈当前操作的进度，实现UI进度的更细，最后反馈执行的结果给UI线程。**优点：简单，快捷，过程可控。缺点：多个异步操作并且需要变更UI时比较复杂。**

![alt text](http://www.fengwenxuan.com/wp-content/uploads/2012/03/AsyncTask.png "Title")

Handler异步实现涉及到Handler、Looper、Message和Thread四个对象，实现异步的流程。实现的流程是主线程启动Thread(子线程)，子线程运行并生成Message,Looper获取Message并传递给Handler,Handler获取Looper中的Message,并进行UI变更。**优点：结构清晰，功能定义明确，多个后台任务时简单清晰。缺点：单个后台异步处理，代码相对过多，结果相对复杂。**

![alt text](http://www.fengwenxuan.com/wp-content/uploads/2012/03/handler_thread_looper_message.png "Title")

##AsynTask详解

###AsyncTask的五个状态，即五个阶段。

1. 准备运行：onPreExecute(),该回调函数在任务被执行之后立即由UI线程调用。这个步骤通常用来建立任务，在用户接口（UI）上显示进度条。
2. 正在后台运行：doInBackground(Params...),该回调函数由后台线程在onPreExecute()方法执行结束后立即调用。通常在这里执行耗时的后台计算。计算的结果必须由该函数返回，并被传递到onPostExecute()中。在该函数内也可以使用publishProgress(Progress...)来发布一个或多个进度单位(unitsof progress)。这些值将会在onProgressUpdate(Progress...)中被发布到UI线程。
3. 进度更新：onProgressUpdate(Progress...),该函数由UI线程在publishProgress(Progress...)方法调用完后被调用。一般用于动态地显示一个进度条。
4. 完成后台任务：onPostExecute(Result),当后台计算结束后调用。后台计算的结果会被作为参数传递给这一函数。
5. 取消任务：onCancelled ()，在调用AsyncTask的cancel()方法时调用

###AsyncTask的三个模板参数

1. Params，传递给后台任务的参数类型。
2. Progress，后台计算执行过程中，进步单位（progress　units）的类型。（就是后台程序已经执行了百分之几了。）
3. Result， 后台执行返回的结果的类型。

###模拟一个进度条更新的后台加载图片更新主线程的例子。

        public class AsyncTaskActivity extends Activity {              
            private ImageView mImageView;  
            private Button mButton;  
            private ProgressBar mProgressBar;  
              
            @Override  
            public void onCreate(Bundle savedInstanceState) {  
                super.onCreate(savedInstanceState);  
                setContentView(R.layout.main);   
                mImageView= (ImageView) findViewById(R.id.imageView);  
                mButton = (Button) findViewById(R.id.button);  
                mProgressBar = (ProgressBar) findViewById(R.id.progressBar);  
                mButton.setOnClickListener(new OnClickListener() {  
                      
                    @Override  
                    public void onClick(View v) {  
                        GetCSDNLogoTask task = new GetCSDNLogoTask();  
                        task.execute("http://csdnimg.cn/www/images/csdnindex_logo.gif");  
                    }  
                });  
            }  
              
            class GetCSDNLogoTask extends AsyncTask<String,Integer,Bitmap> {//继承AsyncTask  
          
                @Override  
                protected Bitmap doInBackground(String... params) {//处理后台执行的任务，在后台线程执行  
                    publishProgress(0);//将会调用onProgressUpdate(Integer... progress)方法  
                    HttpClient hc = new DefaultHttpClient();  
                    publishProgress(30);  
                    HttpGet hg = new HttpGet(params[0]);//获取csdn的logo  
                    final Bitmap bm;  
                    try {  
                        HttpResponse hr = hc.execute(hg);  
                        bm = BitmapFactory.decodeStream(hr.getEntity().getContent());  
                    } catch (Exception e) {  
                          
                        return null;  
                    }  
                    publishProgress(100);  
                    //mImageView.setImageBitmap(result); 不能在后台线程操作ui  
                    return bm;  
                }  
                  
                protected void onProgressUpdate(Integer... progress) {//在调用publishProgress之后被调用，在ui线程执行  
                    mProgressBar.setProgress(progress[0]);//更新进度条的进度  
                 }  
          
                 protected void onPostExecute(Bitmap result) {//后台任务执行完之后被调用，在ui线程执行  
                     if(result != null) {  
                         Toast.makeText(AsyncTaskActivity.this, "成功获取图片", Toast.LENGTH_LONG).show();  
                         mImageView.setImageBitmap(result);  
                     }else {  
                         Toast.makeText(AsyncTaskActivity.this, "获取图片失败", Toast.LENGTH_LONG).show();  
                     }  
                 }  
                   
                 protected void onPreExecute () {//在 doInBackground(Params...)之前被调用，在ui线程执行  
                     mImageView.setImageBitmap(null);  
                     mProgressBar.setProgress(0);//进度条复位  
                 }      
                 protected void onCancelled () {//在ui线程执行  
                     mProgressBar.setProgress(0);//进度条复位  
                 }      
            }   
        }  

AsyncTask为我们抽象出一个后台任务的五种状态，对应了五个回调接口，我们只需要根据不同的需求实现这五个接口（doInBackground是必须要实现的），就能完成一些简单的后台任务。使用AsyncTask的方式使编写后台进程和UI进程交互的代码变得更为简洁，使用起来更加方便。

当然AsyncTask是有缺陷的，具体还没涉及到就先mark一下别人的[博客](http://blog.csdn.net/mylzc/article/details/6784415 "description with a title").

##Handler+Thread详解

禁忌:不要在非UI线程中对UI进行操作，这样一定会抛出类似错误CalledFromWrongThreadException:only the original thread that created a view hierarchy can touch its views。

        public class ThreadHandlerActivity extends Activity {    
            private static final int MSG_SUCCESS = 0; //获取图片成功的标识  
            private static final int MSG_FAILURE = 1; //获取图片失败的标识   
            private ImageView mImageView;  
            private Button mButton;     
            private Thread mThread;  
              
            private Handler mHandler = new Handler() {  
                public void handleMessage (Message msg) {//此方法在ui线程运行  
                    switch(msg.what) {  
                    case MSG_SUCCESS:  
                        mImageView.setImageBitmap((Bitmap) msg.obj); //imageview显示从网络获取到的logo  
                        Toast.makeText(getApplication(),  getApplication().getString(R.string.get_pic_success), Toast.LENGTH_LONG).show();  
                        break;  
          
                    case MSG_FAILURE:  
                        Toast.makeText(getApplication(), getApplication().getString(R.string.get_pic_failure), Toast.LENGTH_LONG).show();  
                        break;  
                    }  
                }  
            };  
              
            @Override  
            public void onCreate(Bundle savedInstanceState) {  
                super.onCreate(savedInstanceState);  
                setContentView(R.layout.main);  
                mImageView= (ImageView) findViewById(R.id.imageView);//显示图片的ImageView  
                mButton = (Button) findViewById(R.id.button);  
                mButton.setOnClickListener(new OnClickListener() {  
                      
                    @Override  
                    public void onClick(View v) {  
                        if(mThread == null) {  
                            mThread = new Thread(runnable);  
                            mThread.start();//线程启动  
                        }  
                        else {  
                            Toast.makeText(getApplication(), getApplication().getString(R.string.thread_started), Toast.LENGTH_LONG).show();  
                        }  
                    }  
                });  
            }  
              
            Runnable runnable = new Runnable() {  
                  
                @Override  
                public void run() {//run()在新的线程中运行  
                    HttpClient hc = new DefaultHttpClient();  
                    HttpGet hg = new HttpGet("http://csdnimg.cn/www/images/csdnindex_logo.gif");//获取csdn的logo  
                    final Bitmap bm;  
                    try {  
                        HttpResponse hr = hc.execute(hg);  
                        bm = BitmapFactory.decodeStream(hr.getEntity().getContent());  
                    } catch (Exception e) {  
                        mHandler.obtainMessage(MSG_FAILURE).sendToTarget();//获取图片失败  
                        return;  
                    }  
                    mHandler.obtainMessage(MSG_SUCCESS,bm).sendToTarget();
                    //获取图片成功，向ui线程发送MSG_SUCCESS标识和bitmap对象    
                }  
            };  
              
        }  

还有一种简便方法即是调用View的post方法来更新ui
         
        mImageView.post(new Runnable() {//另外一种更简洁的发送消息给ui线程的方法。     
            @Override  
            public void run() {//run()方法会在ui线程执行  
                mImageView.setImageBitmap(bm);  
            }  
        });

具体的消息处理机制有:

        public void run() {  
            //建立消息循环的步骤  
            Looper.prepare();//1、初始化Looper  
            mHandler = new Handler(){//2、绑定handler到CustomThread实例的Looper对象  
                public void handleMessage (Message msg) {//3、定义处理消息的方法  
                    switch(msg.what) {  
                    case MSG_HELLO:  
                        Log.d("Test", "CustomThread receive msg:" + (String) msg.obj);  
                    }  
                }  
            };  
            Looper.loop();//4、启动消息循环  
        }  

1. 初始化Looper
2. 绑定handler到CustomThread实例的Looper对象
3. 定义处理消息的方法
4. 启动消息循环

##线程复习

线程的有用之处:

1. 使 UI 响应更快
2. 利用多处理器系统
3. 简化建模
4. 执行异步或后台处理

###线程的两种创建方式:

参考[博客](http://www.cnblogs.com/rollenholt/archive/2011/08/28/2156357.html "description with a title").

Java中要想实现多线程，有两种手段，一种是继续Thread类，另外一种是实现Runable接口。

1. class EG extends Thread ，当然里面要有run()方法。EG eg = new EG(); 调用该线程的时候需要使用线程对象的start()方法，**注意一定不是run()方法。**并且start()方法执行过后，线程已经结束不能重复调用，否则抛出异常。
2. class EG implements Runnable，里面同样要有run()方法。EG eg = new EG();然后Thread th = new Thread(eg);th.start();

第一种方式使用无参构造函数创建线程，则当线程开始工作时，它将调用自己的run()方法。第二种方式使用带参数的构造函数创建线程，因为你要告诉这个新线程使用你的run()方法，而不是它自己的。如上例，可以把一个目标赋给多个线程，这意味着几个执行线程将运行完全相同的作业。两种方法中,实现Implement Runnable接口更容易实现数据资源的共享。所以尽可能实现Runnable接口来完成线程的创建。

在java中，每次程序运行至少启动2个线程。一个是main线程，一个是垃圾收集线程。因为每当使用java命令执行一个类的时候，实际上都会启动一个JVM，每一个JVM实际上就是在操作系统中启动了一个进程。

###判断线程是否启动

`Example eg = new Example();Thread th = new Thread(eg);th.isAlive();`

###线程的转让与强制执行

`Thread.yield()与Thread.join()`yield是放弃CPU资源（暂停当前正在执行的线程对象，并执行其他线程）。而join是将一个新的A线程插入一个B线程之后，当B线程执行结束之后就执行A线程，这个类似修改线程优先级。强制提升A线程优先级，让B线程执行完就直接执行A线程，而不是调度别的线程执行。采用该方法可以完成一些特殊的使用场景。比如某个步骤必须在上一个步骤完成之后才可以完成，这个时候你就可以采用join，把一个操作join到其前置依赖的操作之后。

###线程的休眠

`Thread.sleep(2000);`休眠2秒

###线程的中断

`Thread.interrupt();`

###线程的优先级

`Thread.setPriority(8);`主线程的优先级是5,对于线程是谁先得到CPU谁先执行，优先级越高则会抢占资源执行。Thread类具有三个定义线程优先级范围的静态最终 常量：`Thread.MIN_PRIORITY （为1） Thread.NORM_PRIORITY （为5） Thread.MAX_PRIORITY （为10）`

###线程的同步与死锁

有时可能我们对同一个资源进行操作时,不同的线程会在同一时间内操作,这样就使得实际操作两次,而资源只是用一次的情况发生,这显然是有问题的。这里就要了解一下同步的技术，即是在同一时间段只有一个线程运行。同步有两种方法:同步代码块或者同步方法。

对于同步代码块:

        class hello implements Runnable {
            public void run() {
                for(int i=0;i<10;++i){
                    synchronized (this) {
                        if(count>0){
                            try{
                                Thread.sleep(1000);
                            }catch(InterruptedException e){
                                e.printStackTrace();
                            }
                            System.out.println(count--);
                        }
                    }
                }
            }
        }

对于同步方法:

        class hello implements Runnable {
            public void run() {
                for (int i = 0; i < 10; ++i) {
                    sale();
                }
            }
 
            public synchronized void sale() {
                if (count > 0) {
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(count--);
                }
            }
        }

过多的同步可能导致死锁,这里不具体介绍,参见上面学习的博客地址。
