---
layout: post
title: "Android中网络连接"
description: "对Android网络连接学习,Socket与HTTP两种方式详解"
category: 技术
tags: [Android,学习,技术]
---
{% include JB/setup %}

当前的手机因为有了网络连接与非常多的网络应用而流行起来,也许现在1G的流量都不太够用了。所以Android网络这块一直希望能够学习一下。**网络编程当前主流的是两大块Socket编程与HTTP连接。还有一个TCP/IP的连接，总是与他们脱不了关系。**

学习的主要内容：

1. TCP/IP 的基本知识，网络的基础
2. Socket与HTTP的不同
3. Socket详解
4. HTTP详解

##TCP/IP的基础知识

网络编程中两个主要的问题,一个是如何准确的定位网络上一台或多台主机,另一个就是找到主机后如何可靠高效的进行数据传输。IP层主要负责网络主机定位，IP地址唯一确定Internet上一台主机。传输层提供面向应用的可靠或者非可靠的数据传输机制，这也是网络编程的主要对象，一般不需要关心IP层如何处理数据的。

###两类传输协议:TCP、UDP

TCP:Transfer Control Protocal。是一种可靠的面向连接的传输协议。通过协议得到一个顺序的无差错的数据流。发送方和接收方的成对Socket之间必须建立连接，服务器端有守护进程等待连接，客户端申请连接。需要有三次握手或者四次握手。

UDP:User Datagram Protocol。无连接协议，每个数据都是完整的，包括完整的源与目的地址，它在网络上以任何可能的路径传往目的地，但最后不保证是否按时正确的到达。

一个是可靠的面向连接的,有连接时间;另一个是不可靠的,无需简历连接,传输有大小限制的。因为TCP需要检验正确性等，效率较低，类似于拨打电话，建立专门的虚拟连接；UDP性能较好，适用于分散系统的应用程序，例如视频会议等。

##Socket与HTTP不同

创建Socket连接时，可以指定使用的传输协议，它支持TCP与UDP。通常情况下我们是说Socket指TCP连接。Socket一旦建立，双发即可开始相互发送数据，直到双方连接断开。而实际应用中我们要穿过中间层，包括路由器，网关，防火墙等，大部分防火墙默认会关闭长时间处于非活跃状态的连接而导致Socket连接断开，因此需要轮询告诉网络，该连接处于活跃状态。

HTTP使用的是请求-响应的方式，不仅在请求时需要先建立连接，而且需要客户端发出请求后，服务器端才能回复。很多情况需要服务器主动推送数据给浏览器或者客户端，在HTTP连接时，客户端定时向服务器发送连接请求，不仅可以保持在线，同时也是在“询问”服务器是否有新的数据，如果有就将数据传给客户端。

##Socket的详解

套接字是通信的基石,支持TCP/IP协议的基本操作单元，是通信过程中端点的抽象表示，包含了五种信息：**连接使用的协议，本地主机的IP地址，本地进程的协议端口，远地主机的IP地址，远地进程的协议端口。**

应用层通信时，TCP可能会遇到多个应用程序进程提供并发服务的问题。多个连接或多个应用程序可能需要同一个TCP协议端口传输数据。为了区别不同，操作系统为应用程序与TCP协议交互提供了套接字（Socket）接口。通过接口区分不同的服务，实现数据传输的并发服务。

建立Socket连接至少需要一对套接字，其中一个运行于客户端，称为ClientSocket，另一个运行与服务器端称为ServerSocket。

套接字之间的连接过程分为三个步骤：服务器监听，客户端请求，连接确认。一个基本过程是这样的:**Server 端Listen(监听)某个端口是否有连接请求，Client端向 Server端发出Connect(连接)请求，Server端向Client端发回 Accept（接受）消息。一个连接就建立起来了。Server端和Client端都可以通过Send，Write等方法与对方通信。**

一个功能齐全的Socket，包含以下基本结构，工作过程包含以下四个基本的步骤：

1. 创建Socket
2. 打开连接到Socket的输入/输出流
3. 按照一定的协议对Socket进行读/写操作
4. 关闭Socket。

###具体实现

java在包java.net中提供了两个类Socket和ServerSocket，分别用来表示双向连接的客户端和服务端。这是两个封装得非常好的类，使用很方便。其构造方法如下：

* Socket(InetAddress address, int port);
*　Socket(InetAddress address, int port, boolean stream);
*　Socket(String host, int prot);
*　Socket(String host, int prot, boolean stream);
*　Socket(SocketImpl impl)
*　Socket(String host, int port, InetAddress localAddr, int localPort)
*　Socket(InetAddress address, int port, InetAddress localAddr, int localPort)
*　ServerSocket(int port);
*　ServerSocket(int port, int backlog);
*　ServerSocket(int port, int backlog, InetAddress bindAddr)

客户端

        try{
            Socket client = new Socket("202.193.74.128",4701);
            //向本机的4700端口发出客户请求
            BufferedReader sin=new BufferedReader(new InputStreamReader(System.in));
            //由系统标准输入设备构造BufferedReader对象
            PrintWriter os=new PrintWriter(client.getOutputStream());
            //由Socket对象得到输出流，并构造PrintWriter对象
            BufferedReader is=new BufferedReader(new InputStreamReader(client.getInputStream()));
            //由Socket对象得到输入流，并构造相应的BufferedReader对象
            String readline;
            readline = sin.readLine(); //从系统标准输入读入一字符串
            while(!readline.equals("bye")){
                //若从标准输入读入的字符串为 "bye"则停止循环
                os.println(readline);
                //将从系统标准输入读入的字符串输出到Server
                os.flush();
                //刷新输出流，使Server马上收到该字符串
                System.out.println("Client:"+readline);
                //在系统标准输出上打印读入的字符串
                System.out.println("Server:"+is.readLine());
                //从Server读入一字符串，并打印到标准输出上
                readline=sin.readLine(); //从系统标准输入读入一字符串
            }
            os.close();
            is.close();
            client.close();
            
        }catch(Exception e){
            e.printStackTrace();
        }

服务器端

        try{
            ServerSocket server = null;
            try{
                server = new ServerSocket(4700);
                //创建一个ServerSocket在端口4700监听客户请求
            }catch(Exception e){
                System.out.println("can not listen to:"+e);
            }
            Socket socket = null;
            try{
                socket = server.accept();
                //使用accept()阻塞等待客户请求，有客户
                //请求到来则产生一个Socket对象，并继续执行
            }catch(Exception e) {
                System.out.println("Error."+e);
                //出错，打印出错信息
            }
            String line;
            BufferedReader is = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            //由Socket对象得到输入流，并构造相应的BufferedReader对象
            PrintWriter os=new PrintWriter(socket.getOutputStream());
            //由Socket对象得到输出流，并构造PrintWriter对象
            BufferedReader sin=new BufferedReader(new InputStreamReader(System.in));
            //由系统标准输入设备构造BufferedReader对象
            System.out.println("Client:"+is.readLine());
            //在标准输出上打印从客户端读入的字符串
            line = sin.readLine();
            while(!line.equals("bye")){
                //若从标准输入读入的字符串为 "bye"则停止循环
                os.println(line);
                //将从系统标准输入读入的字符串输出到Server
                os.flush();
                //刷新输出流，使Server马上收到该字符串
                System.out.println("Server:"+line);
                //在系统标准输出上打印读入的字符串
                System.out.println("Clinet:"+is.readLine());
                //从Server读入一字符串，并打印到标准输出上
                line=sin.readLine(); //从系统标准输入读入一字符串
            }
            os.close();
            is.close();
            socket.close();
            server.close();
            
        }catch(Exception e){
            e.printStackTrace();
        }

测试了一下，还有很多不完善的地方，还有许多需要改进，具体博客可以参考[地址](http://blog.csdn.net/Siobhan/article/details/4277380 "description with a title")

##HTTP详解

超文本传输协议,HTTP使用了可靠传输的TCP协议。这样就保证了我们所访问的资源万无一失，不会产生数据丢失或者损坏。一般是浏览器向Web服务器发送一个请求，Web服务器根据请求的资源提供数据。

一般通过URL统一资源定位符来定位具体的地址与内容。
1. 第一部分是协议
2. 第二部分是主机IP地址包括端口号，当然默认是80
3. 资源的具体地址，用“/”隔开。

###常见的HTTP方法

1. HEAD 向服务器要与Get请求相一致的响应，只不过响应体将不会被返回，一般为了获取包含在响应头中的元信息。
2. GET 向特定的资源发出请求。
3. POST 向指定资源提交数据进行处理请求（提交表单或者上传文件）。数据包含在请求提中，POST可能导致新的资源的建立或者修改
4. PUT 向指定资源上传新内容
5. DELETE 请求服务器删除Request-URL所标识的资源。

常用的web交互一般使用Get与Post，这两种方法的区别在于：

- Get的变量是在URL的？后面跟随；Post是单独在请求体中
- Get提交的数据有限制，最多1024自己；Post没有
- Get提交时会显示在地址栏；Post不会
- 在服务器端两者获取的方法也不同。
- Get方式更安全一些。

在请求之后会有一些常用的返回状态,常用的有:

- 200 (OK): 找到了该资源，并且一切正常。
- 401 (UNAUTHORIZED): 客户端无权访问该资源。这通常会使得浏览器要求用户输入用户名和密码，以登录到服务器。
- 403 (FORBIDDEN): 客户端未能获得授权。这通常是在401之后输入了不正确的用户名或密码。
- 404 (NOT FOUND): 在指定的位置不存在所申请的资源。
- 502 (Bad Gateway):作为网关或者代理工作的服务器尝试执行请求时，从上游服务器接收到无效的响应。

详尽的返回状态码可以参照[百科](http://baike.baidu.com/view/1790469.htm  "description with a title")

###请求报文

一个HTTP请求报文由请求行，请求头部和请求数据3个部分组成，HTTP有两类报文：请求报文和响应报文。顾名思义一个是客户端发送的请求时相应的HTTP报文，一个是Web服务器响应的HTTP报文。一般请求有Post与Get两种方法，上面已经详细说明了具体区别。这里主要看一下两种不同格式的报文：

这是一个**Post**请求 我们可以看到请求里面包括了url，code，connection，accept-language，userAgent，cookie还有**最后的数据**。Get请求时数据直接出现在url上。

        Request URL:http://lishen.sinaapp.com/admin
        Request Method:POST
        Status Code:200 OK

        Request Headers

        Accept:text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
        Accept-Encoding:gzip,deflate,sdch
        Accept-Language:zh-CN,zh;q=0.8,en-US;q=0.6,en;q=0.4
        Connection:keep-alive
        Content-Length:63
        Content-Type:application/x-www-form-urlencoded
        Cookie:lzstat_uv=30932486692382688113|3008730; pgv_pvi=1077707776; bdshare_firstime=1354712737008; UOR=event.youku.com,phmenu.sinaapp.com,; JSESSIONID="mhsfgb25606am9brquq8wwn6=/1/lishen"; saevisited=202.193.75.79.138189402810310216; Hm_lvt_1ec5b26562e28d79ec2fd80317c41165=1380079867,1380184987,1381158846,1381853287; Hm_lpvt_1ec5b26562e28d79ec2fd80317c41165=1381894029
        Host:lishen.sinaapp.com
        Origin:http://lishen.sinaapp.com
        Referer:http://lishen.sinaapp.com/login
        User-Agent:Mozilla/5.0 (Windows NT 6.2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/30.0.1599.69 Safari/537.36

        Form Dataview  URL encoded
        useremail:xxxx@xx.com
        password:xxxxxx
        loginRemember:on

###利用apache的Http包具体实现

Get方式：

        String url = "http://10.0.2.2/lishenBlog/json/commentAction!testAndroid"
        url+="?input=answer";
        HttpClient client=new DefaultHttpClient();
        HttpGet httpGet ;
        try {
           httpGet = new HttpGet(url);
           HttpResponse response=client.execute(httpGet);
           //判断请求是否成功
           if(response.getStatusLine().getStatusCode()==200){
               System.out.println("成功");
               Log.i(TAG, "成功");  
               HttpEntity  entity=response.getEntity();              
               if(entity!=null){
                   String resultJson = EntityUtils.toString(entity,"UTF-8");
                   System.out.println(resultJson);
                   String out="";
                   try{
                       JSONObject jsonObj=new JSONObject(resultJson);
                       out = jsonObj.getString("result"); 
                   }catch(Exception e){
                       e.printStackTrace();
                   }
                   new AlertDialog.Builder(this).setMessage(out).create().show();
               }
           }
           else{
               System.out.println(response.getStatusLine().getStatusCode()+"有问题"); 
           }
        }catch (ClientProtocolException e) {
           e.printStackTrace();
        }catch (IOException e) {
           e.printStackTrace();
        }

Post方式：

        HttpClient client=new DefaultHttpClient();
        HttpPost request; 
        try {
           request = new HttpPost(url); 

           //Post参数
           List<NameValuePair> params = new ArrayList<NameValuePair>();
           params.add(new BasicNameValuePair("input","answer"));
            //         可以继续添加其他参数
            //params.add(new BasicNameValuePair("input","answer"));  
            //         可以继续添加其他参数  

           //设置字符集
           HttpEntity httpEntity = new UrlEncodedFormEntity(params,HTTP.UTF_8); 
           
           request.setEntity(httpEntity);
           
           HttpResponse response=client.execute(request);
           //判断请求是否成功
           if(response.getStatusLine().getStatusCode()==200){
               System.out.println("成功");
               Log.i(TAG, "成功");  
               HttpEntity  entity=response.getEntity(); 
               if(entity!=null){
                   String resultJson = EntityUtils.toString(entity,"UTF-8");
                   System.out.println(resultJson);
                   String out="";
                   try{
                       JSONObject jsonObj=new JSONObject(resultJson);
                       out = jsonObj.getString("result"); 
                   }catch(Exception e){
                       e.printStackTrace();
                   }
                   new AlertDialog.Builder(this).setMessage(out).create().show();
               }
           }
           else{
               System.out.println(response.getStatusLine().getStatusCode()+"有问题"); 
           }
        }catch (ClientProtocolException e) {
           e.printStackTrace();
        }catch (IOException e) {
           e.printStackTrace();
        }       

总结：两者请求有所不同，一个直接用url装入参数请求，另一个Post需要在请求体中set进变量。共同点是两者都使用HttpClient去执行，并且返回给HttpResponse,返回包括了状态码与信息。

Java.net包中的HttpURLConnection类也有这些基础方式。

Get方式：

        String path="http://xxx.xx.com/login.jsp?id=hello&pwd=yes"
        //新建一个url对象
        URL url = new URL(path);
        //打开一个连接
        HttpURLConnection urlCon = (HttpURLConnection)url.openConncection();
        //设置超时时间
        urlCon.setConncetTimeout(5*1000);
        //开始连接
        urlCon.conncet();
        if(urlCon.getResponseCode()==HTTP_200){
            //获取返回数据、
            byte[] data = readStream(urlCon.getInputStream());
            Log.i(TAG_GET, new String(data, "UTF-8"));  
        }else{
             Log.i(TAG_GET, "Get方式请求失败");  
        }
        //关闭连接
        urlCon.disconnect();

Post方式

        String path="http://xxx.xx.com/login.jsp"

        //多了输入参数
        String params = "id="+URLEncoder.encode("hello","utf-8")+"&pwd="+URLEncoder.encode("yes","utf-8");
        byte[] postData = params.getBytes();

        //新建一个url对象
        URL url = new URL(path);
        //打开一个连接
        HttpURLConnection urlCon = (HttpURLConnection)url.openConncection();
        //设置超时时间
        urlCon.setConncetTimeout(5*1000);
        //post请求设置允许输出  
        urlConn.setDoOutput(true);  
        // Post请求不能使用缓存  
        urlConn.setUseCaches(false);  
        // 设置为Post请求  
        urlConn.setRequestMethod("POST");  
        urlConn.setInstanceFollowRedirects(true);  
        // 配置请求Content-Type  
        urlConn.setRequestProperty("Content-Type",  
                "application/x-www-form-urlencode");
        //开始连接
        urlCon.conncet();

        //发送带请求的参数
        DataOutputStream dos= new DataOutputStream(urlCon.getOutputStream());
        dos.write(postData);
        dos.flush();
        dos.close();

        if(urlCon.getResponseCode()==HTTP_200){
            //获取返回数据、
            byte[] data = readStream(urlCon.getInputStream());
            Log.i(TAG_GET, new String(data, "UTF-8"));  
        }else{
             Log.i(TAG_GET, "Get方式请求失败");  
        }
        //关闭连接
        urlCon.disconnect();

