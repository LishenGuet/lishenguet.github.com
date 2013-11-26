---
layout: post
title: "Android适配器Adapter"
description: "对Android适配器Adapter学习，多种Adapter详解"
category: 技术
tags: [Android,学习,技术]
---
{% include JB/setup %}

Adapter是一种适配器，主要连接后端数据与前端显示。当然在数据里面我们可以有数据库SQLite存储拿到的数据，也可以是其他数据。显示有ListView与GridView。这里主要学习Adapter,先看下它的接口类视图:

    |--Adapter
    |--|--ListAdapter
    |--|--|--BaseAdapter
    |--|--|--|--ArrayAdapter
    |--|--|--|--CursorAdapter
    |--|--|--|--|--ResourceCursorAdapter--SimpleCursorAdapter
    |--|--|--|--SimpleAdapter

还有一些SpinnerAdapter与WrapperListAdapter不常用，所以这里不多解释。

- BaseAdapter是一个抽象类，继承它需要实现较多方法，较高的灵活性
- ArrayAdapter支持泛型操作，最为简单，只能展示一行字
- SimpleAdapter有较好的扩充性，可以自定义出各种效果
- SimpleCursorAdapter适用简单的纯文字型ListView,需要Cursor的字段和UI的id对应起来，如果要求较为复杂的UI可以重写其他方法。

##ArrayAdapter

这个比较简单，即是一条条的将数据显示。View比较简单，只能显示一行文字。上面代码使用了`ArrayAdapter(Context context, int textViewResourceId, List<T> objects)`来装配数据，要装配这些数据就需要一个连接ListView视图对象和数组数据的适配器来两者的适配工作，ArrayAdapter的构造需要三个参数，依次为this,布局文件（注意这里的布局文件描述的是列表的每一行的布局，`android.R.layout.simple_list_item_1`是系统定义好的布局文件只显示一行文字，数据源(一个List集合)。同时用setAdapter（）完成适配的最后工作。直接看代码吧

    public List<String> getData(){
      List<String> data = new ArrayList<String>();
      for(int i =0;i<=5;i++){
        data.add("第"+i+"条内容");
      } 
      return data;
    }
    public void onCreate(Bundle savedInstanceState){
      super.onCreate(savedInstanceState);
      
      mListView = new ListView(this);
      ArrayAdapter adapter = new ArrayAdapter<String>(this,android.R.layout.simple_list_item_1,getData());
      mListView.setAdapter(adapter);
      //要在这个地方才把展示View放入，之前要对它进行赋值，不然会出现空指针
      setContentView(mListView);
    }

##SimpleAdapter

扩展性很好，可以放ImageView，Button,CheckBox等等。它可以显示GridView和ListView。对于SimpleAdapter的数据一般使用HashMap构成的List，List的每一节对应ListView或GridView的一行。HashMap的每个键值数据映射到布局文件中对应的id组件上。适配的时候SimpleAdapter对应的参数意义：

    //参数
    //context　　SimpleAdapter关联的View的运行环境
    //data　　一个HashMap组成的List。在列表中的每个条目对应列表中的一行,每一个map中应该包含所有在from参数中指定的键
    //resource 一个定义列表项的布局文件的资源ID。布局文件将至少应包含那些在to中定义了的ID
    //from  一个将被添加到Map映射上的键名
    //to　　　将绑定数据的视图的ID,跟from参数对应,这些应该全是控件

###GridView

要在GridView的Layout里面设置每行的列数

    <GridView android:id="@+id/gridview"
        android:layout_width="match_parent"
        android:layout_height="fill_parent"
        android:numColumns="3"> 
    </GridView>

每一个GridView里面可以放图片，按钮，文字等等，也可以组合在一起

    <ImageView android:layout_width="100dip"
        android:layout_height="80dip"
        android:id="@+id/ItemImage"
        android:layout_gravity="center_horizontal"/>
    <TextView android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/ItemText"
        android:layout_gravity="center"/>

做适配

    SimpleAdapter simple = new SimpleAdapter(this,list,R.layout.activity_gridviewitem,new String[] {"ItemImage","ItemText"},new int[]{R.id.ItemImage,R.id.ItemText});
    gridView.setAdapter(simple);

###ListView

View可以使用RelativeLayout做出类似微信的页面，带有用户名，用户头像，用户的最近状态，最近更新时间等。
    
    public void onCreate(Bundle savedInstanceState){
      super.onCreate(savedInstanceState);
      setContentView(R.layout.activity_listview1);
      
      ListView listView = (ListView)findViewById(R.id.listView);
      String[] strings={"img","title","info","time"};
      int[] ids = {R.id.img, R.id.title, R.id.info, R.id.time};
      
      ArrayList<HashMap<String,Object>> list = new ArrayList<HashMap<String,Object>>();
      for(int i =0;i<=5;i++){
        HashMap<String,Object> map = new HashMap<String,Object>();
        map.put("title", "人物"+i);
        map.put("time", "9月20日");
        map.put("info", "我通过了你的好友验证请求，现在我们可以开始对话啦");  
        map.put("img", R.drawable.png1);  
        list.add(map);  
      }
      
      SimpleAdapter simple = new SimpleAdapter(this,list,R.layout.activity_listview1item,strings,ids);
      listView.setAdapter(simple);
    }

##SimpleCursorAdapter

主要是与数据库查出的数据做同步，这种与数据库一起非常方便。这种方法类似上面所述的`SimpleAdapter，android.widget.SimpleCursorAdapter.SimpleCursorAdapter(Context context, int layout, Cursor c, String[] from, int[] to)`参数解释1.context 当前环境上下文。2.layout：要绑定的布局文件。3.c：从数据库返回的数据，存放在Cursor中。4.from：对应要绑定的数据库中的字段名。5.to：对应要绑定的控件ID。

这里要注意的是返回的Cursor这个类，因为都是封装好的，所以在SimpleCursorAdapter中传入的cursor必须要有`_id`这个字段，Cursor会根据这个字段进行逐一的数据绑定，如果数据库中没有`_id`，则利用SimpleCursorAdapter绑定就会报错。

    static final String[] FROM = { DbHelper.C_CREATED_AT,DbHelper.C_USER,
    DbHelper.C_TEXT };
    static final int[] TO = { R.id.textCreatedAt,R.id.textUser,R.id.textText };
    cursor=db.query(DbHelper.TABLE, null, null, null, null, null, 
        DbHelper.C_CREATED_AT+" DESC");
    //管理游标生命周期的办法，活动管理自己生命周期的同样方式管理游标
    startManagingCursor(cursor);
    SimpleCursorAdapter adapter=new SimpleCursorAdapter(this,R.layout.row,cursor,FROM,TO);
    adapter.setViewBinder(VIEW_BINDER);
    listTimeline.setAdapter(adapter);

这个地方为了把发表的时间修改为5分钟之前或者10分钟之前发表的这种形式，特别订制，修改了逻辑功能。bindview方法用于实现数据和view的绑定。

    //实现一个ViewBinder的实例，实现一个内部类
    //在每条数据元素与其对应的控件绑定时调用
    static final ViewBinder VIEW_BINDER =new ViewBinder(){
        @Override
        public boolean setViewValue(View view, Cursor cursor, int columnIndex) {
          if(view.getId() != R.id.textCreatedAt){
            //检查如果不是我们要的空间，就返回false;
            return false;
          }
          //如果是，先拿到他columnIndex的值
          long timeCreatedAt=cursor.getLong(columnIndex);
          CharSequence relTime= DateUtils.getRelativeTimeSpanString(view.getContext(),timeCreatedAt);
          ((TextView) view).setText(relTime);
          return true;      
        }   
    };

##BaseAdapter

BaseAdapter的灵活性相当高，可以深度定制。它实现了ListAdapter，SpinnerAdapter。能够为(Spinner,ListView,GridView)填充数据。一般我们建立自己的Adapter然后继承BaseAdapter,需要完成下面这四个方法

    public int getCount(); //为了得到一共几行
    public Object getItem(int position); 
    public long getItemId(int position);
    public View getView(int position, View convertView, ViewGroup parent);
    // position就是位置从0开始，convertView是Spinner,ListView中每一项要显示的view
    //通常return 的view也就是convertView
    //parent就是父窗体了，也就是Spinner,ListView,GridView了.

自己定制一个Adapter继承BaseAdatper,在getView方法中可以简单显示一个TextView，可以显示一个Layout，也可以在Layout里事先定义好布局。这里我新建了一个名叫baseadapter_provider.xml，通过LayoutInflater直接调用。见下面的代码：

    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="horizontal" >    
        <ImageView  
            android:id="@+id/imageViewBaseAdapter"    
            android:layout_width="wrap_content"   
            android:layout_height="wrap_content"   
            android:src="@drawable/png1"  />  
        <TextView  
            android:id="@+id/textViewBaseAdapter"    
            android:layout_width="wrap_content"   
            android:layout_height="wrap_content"   
            android:text="BaseAdapter"  />  
    </LinearLayout>

    public class MyAdapter extends BaseAdapter {
      private Context _ctx = null;
      private String[] data = {"foo","bar","foobar","barfoo"};
      //在类中还加上了一个Context类型的变量_ctx，此变量将在构造时被赋值。放个简单的TextView，这里对getView做修改就可以了。
      public MyAdapter(Context context){
        _ctx = context;
      }     
      @Override
      public int getCount() {
        // TODO Auto-generated method stub
        return data.length;
      }
      @Override
      public Object getItem(int position) {
        // TODO Auto-generated method stub
        return data[position];
      }
      @Override
      public long getItemId(int position) {
        // TODO Auto-generated method stub
        return position;
      }
      @Override
      public View getView(int position, View convertView, ViewGroup parent) {
        // TODO Auto-generated method stub
        /*
         * 显示只有一个TextView
          TextView tv = null;
          if(null==convertView){
            tv=new TextView(_ctx);
            Log.v("日志", "Posl is"+position);
          }
          else{
            tv= (TextView)convertView;
          }
          tv.setText(data[position]);
          return tv;
        */
        /*
         * 给出一个LinearLayout
          LinearLayout lay = null;
          TextView tv1=new TextView(_ctx);
          TextView tv2=new TextView(_ctx);
          tv1.setText("* ");
          tv2.setText(data[position]);
          tv2.setGravity(Gravity.RIGHT);
          if(null==convertView){
            lay = new LinearLayout(_ctx);
            lay.addView(tv1);
            lay.addView(tv2);
          }
          else{
            lay = (LinearLayout)convertView;
            System.out.println("执行到这了");
          }
          return lay;
        */
        convertView = LayoutInflater.from(_ctx).inflate(R.layout.baseadapter_provider,null);
        TextView mTextView = (TextView)convertView.findViewById(R.id.textViewBaseAdapter);
        mTextView.setText("BaseAdapterDemo"+position);
        mTextView.setTextColor(Color.RED);
        return convertView;
      }
    }

    public void onCreate(Bundle savedInstanceState){
      super.onCreate(savedInstanceState);
      setContentView(R.layout.activity_mylistview);
      ListView lv= (ListView)findViewById(R.id.summary);
      MyAdapter myAdapter = new MyAdapter(this);
      lv.setAdapter(myAdapter);
    }

以上文章参照以下博客[博客1][1]、[博客2][2]、[博客3][3]、[博客4][4]、[博客5][5]、[博客6][6]

[1]: http://www.cnblogs.com/JDTang/archive/2012/12/11/2812423.html "博客1"
[2]: http://hubingforever.blog.163.com/blog/static/1710405792010431621954/ "博客2"
[3]: http://suchang1123.blog.163.com/blog/static/2001940512011103025510822/ "博客3"
[4]: http://disanji.net/2010/11/25/android-baseadapter-spinner-listview-gridview/ "博客4"
[5]: https://code.google.com/p/androidlearn/wiki/Adapter_custom "博客5"
[6]: http://blog.csdn.net/Android_Tutor/article/details/5513869 "博客6"







