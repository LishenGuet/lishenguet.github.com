---
layout: post
title: "学习Android的SQLite"
description: "详细讲解SQLite有关数据库的增删改查操作"
category: 技术
tags: [Android,学习,技术]
---
{% include JB/setup %}

在任何一种应用程序中都少不了数据存储，Web中不用多说。而在手机中，我们有时候需要管理我们的记事本，备忘录等等。当前使用的较多的都是关系型的数据库，大型web会用到Oracle，MySql。在移动设备中用到SQLite。当然还有MongoDB等数据库。

在Android中主要使用SQLite做数据存储。SQLite有以下优点：**轻量级、嵌入式、关系型数据库、支持SQL语言、开源，可移植性好、单文件系统，方便管理和维护、具有读写控制的安全性。**

##SQLite介绍

SQLite数据类型：NULL空值、INTEGER带符号整型、REAL浮点型、TEXT字符串文本、BLOB二进制对象。**它最大的特点就是弱类型，即在存储的时候可以将类型进行改变，有一种情况例外：在定义INTEGER PRIMARY KEY的字段只能存储64位整数，当向该字段保存其他类型时，将产生错误。**

数据库存储在 data/data/项目文件夹/databases/下

Android下需要自己创建数据库，然后创建表、索引、填充数据。Android平台下数据库相关的类有三种：

* SQLiteOpenHelper 抽象类：通过继承实现用户类，提供数据库打开、关闭等操作函数。
* SQLiteDatabase 数据库访问类：执行对数据库的增删改查操作。
* SQLiteCursor 查询结构操作类：用来访问查询结果中的记录。

##SQLiteOpenHelper

它是SQLiteDatabase的一个帮助类，用来管理数据库的创建和版本更新。一般继承之后主要实现onCreate和onUpgrade方法。主要有以下方法：

* SQLiteOpenHelper(Context context,String name,SQLiteDatabase.CursorFactory factory,int version)是一个构造方法，一般传递数据库名称。
* onCreate(SQLiteDatabase db)创建数据库时调用
* onUpgrade(SQLiteDatabase db,int oldVersion,int newVersion)版本更新时调用
* getReadableDatabases()创建或打开一个只读数据库
* getWritableDatabases()创建或打开一个读写数据库

###举例

        
        public class DatabaseHelper extends SQLiteOpenHelper {
 
        private static final String DB_NAME = "mydata.db"; //数据库名称
        private static final int version = 1; //数据库版本
     
        public DatabaseHelper(Context context) {
            super(context, DB_NAME, null, version);
        }
        //数据库第一次创建时会调用，一般在其中创建数据库表
        @Override
        public void onCreate(SQLiteDatabase db) {
            String sql = "create table user(id INTEGER PRIMARY KEY AUTOINCREMENT," +"name varchar(20), address TEXT)";          
            db.execSQL(sql);
        }
        //当数据库需要修改的时候，Android系统会主动的调用这个方法。
        @Override
        public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
            db.execSQL("drop table if exists " + TABLE);
            onCreate(db);
        }
        }

###实现创建一个数据库

        DatabaseHelper database = new DatabaseHelper(this);//这段代码放到Activity类中才用this

        //或者使用这种构造方法
        DatabaseHelper database = new DatabaseHelper(this, "user", null, 1);
        SQLiteDatabase db = null;
        db = database.getReadalbeDatabase();

##SQLiteDatabase

它主要是对数据进行操作的，一般有以下方法，具体可以查API

* （int）delete(String table,String whereClause,String[] whereArgs)删除数据行的便捷方法
* （long）insert(String table,String nullColumnHack,ContentValues values)添加数据行的便捷方法
* （int）update(String table,ContentValues values,String whereClause,String[] whereArgs)更新数据的便捷方法
* （void）execSQL(String sql)执行一个SQL语句，同样可以执行增加删除与更新操作
* （void）close)()关闭数据库
* （Cursor）query（String table,String[] columns,String selection,String[] selectionArgs,String groupBy,String having,String orderBy,String limit）查询指定表数据返回一个带游标的数据集，于防止SQL注入
* （Cursor）rawQuery(String sql,String[] selectionArgs)运行一个预置的SQL语句，返回带游标的数据集

###execSQL操作

        String sql = "insert into user(username,password) values ('Jack Johnson','iLovePopMuisc');//插入操作的SQL语句
        String sql = "delete from user where username='Jack Johnson'";//删除操作的SQL语句
        String sql = "update user set password = 'iHatePopMusic' where username='Jack Johnson'";//修改的SQL语句
        db.execSQL(sql);//执行SQL语句

###数据的添加

        public void insert() {
            //ContentValues对象，向其中插入键值对，键是数据库列名，值是插入到这一列的值
            ContentValues cv = new ContentValues();
            cv.put("name", "LiuMing");
            cv.put("address", "ShangHai");
            mDataBase.insert("user", null, cv);
        }

###数据的修改

        public int update() {
            ContentValues cv = new ContentValues();
            cv.put("name", "MaLi");
            //第三个参数where语句，相当于sql语句中where后面的语句，?占位符
            //第四个参数是占位符的值
            return mDataBase.update(USER_TABLE, cv, "id = ?", new String[]{"1"});
        }

###数据的删除

        public int delete() {
            //和udate()类似
            return mDataBase.delete(USER_TABLE, "id = ? and name = ?",new String[]{"1", "MaLi"});
        }

###数据的查询两种方法


        public Cursor rawQuery() {
            return mDataBase.rawQuery("select * from user where id = ? and name = ?",new String[]{"1", "LiuMing"});
        }
        //使用SELECT语句段构建查询，SELECT语句内容做为query()方法的参数
        public Cursor query() {
            String[] columns = {"name", "address"};
            //第二个参数： 要查询的列名
            //第三个参数：where语句
            //第三个参数：where语句中占位符的值
            //第四个参数：对查询结果进行分组
            //第五个参数：对分组结果进行限制
            //第六个参数：对查询结果进行排序
            return mDataBase.query(USER_TABLE, columns, "id = ?", new String[] {"1"},null, null, null);
        }


##Cursor游标

Cursor是结果集的游标，用于对结果集进行随机访问，提供了很多种方法：

- getCount()总记录条数
- isFirst()判断是否第一条记录
- isLast()判断是否最后一条记录
- moveToFirst()移动到第一条记录
- moveToLast()移动到最后一条记录
- move(int offset)移动到指定的记录
- moveToNext()移动到吓一条记录
- moveToPrevious()移动到上一条记录
- getColumnIndex(String columnName)获得指定列索引的int类型值

###举例

        Cursor c = db.query("user",null,null,null,null,null,null);//查询并获得游标
        if(c.moveToFirst()){//判断游标是否为空
            for(int i=0;i<c.getCount();i++){
                c.move(i);//移动到指定记录
                String username = c.getString(c.getColumnIndex("username");
                String password = c.getString(c.getColumnIndex("password"));
            }
        }
        c.close();

最后记得关闭数据库等操作。还有一些方法都可以在android的API中查到。这里不再详述！

