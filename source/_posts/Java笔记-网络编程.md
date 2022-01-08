---
title: Java笔记--网络编程
sidebar:
  - toc
date: 2020-04-17 17:26:40
tags: [Java]
categories: [Java基础]
---

## Java笔记--网络编程

一、 网络編程中有两个主要的问题:

1. 如何准确地定位网络上一台或多台主机：定位主机上的特定的应用
2. 到主机后如何可靠高效地进行数据传输

二、 网络编程的两个要素

1. 对应问题一：IP和端口号
2. 对应问题二：提供网络通信协议：TCP/IP参考模型
   + 应用层
   + 传输层
   + 网路层
   + 物理+数据链路层

### 通信要素一：IP和端口号

1. IP：唯一的表示Internet上的计算机（通信实体）
2. 在Java中使用InetAdress类代表IP
3. IP分类：IPv4和IPv6；万维网和局域网
4. 域名
5. 本地回路地址：127.0.0.1  localhost
6. 如何实例化InetAdress：两个方法：
   1. getByName(String host)
   2. getLocalHost()
7. 获取域名，获取主机地址
   1. getHostName()
   2. getHostAdress()

```java
public void test() {

    try {
        InetAddress inet1 = InetAddress.getByName("192.168.1.103");
        InetAddress inet1_1 = InetAddress.getByName("tplogin.cn");
        InetAddress inet2 = InetAddress.getByName("www.baidu.com");
        InetAddress inet3 = InetAddress.getByName("localhost");
        InetAddress inet3_1 = InetAddress.getLocalHost();

        System.out.println(inet1);
        System.out.println(inet1_1);
        System.out.println(inet2);
        System.out.println(inet3);
        System.out.println(inet3_1);

        System.out.println(inet2.getHostName());
        System.out.println(inet2.getHostAddress());
    } catch (UnknownHostException e) {
        e.printStackTrace();
    }

}
```

8. **端口号**：端口号标识正在计算机上运行运行的进程（程序）
   + 不用的进程有不同的端口号
   + 被规定为一个16位的整数0~65535。
   + 端口分类：
     + **公认端口**：0~1023。被预先定义的服务通信占用（如：HTTP占用端口80，FTP占用端口21，Telnet占用端口23）
     + **注册端口**：1024-49151。分配给用户进程或应用程序。（如：Tomcat占用端口8080，MySQL占用端口3360，Oracle占用端口1521等）
     + **动态**/**私有端口**：49152-65535
   + **端口号与IP地址的组合得出一个网络套接字：Socket**

### 通信要素二：网络通信协议

#### TCP/IP协议簇

+ **传输控制协议**-------TCP
+ **用户数据报协议**---UDP

+ **TCP协议**:
  ➢ 使用TCP协议前，须先建立**TCP**连接，形成传输数据通道
  ➢ 传输前，采用“**三次握手**”方式，点对点通信，**是可靠的**
  ➢ TCP协议进行通信的两个应用进程:客户端、服务端。
  ➢ 在连接中可**进行大数据量的传输**
  ➢ 传输完毕，需**释放已建立的连接，效率低**

+ **UDP协议**
  ➢ 将数据、源、目的封装成数据包，**不需要建立连接**
  ➢ 将每个数据包的大小限制在64K内
  ➢发送不管对方是否准备好，接收方收到也不确认，故是不可靠的
  ➢可以广播发送
  ➢发送数据结束时**无需释放资源，开销小，速度快**

### TCP的网络编程

##### 客户端发送文本到服务端

```java
@Test
//这是一个客户端的程序
public void client() {
    Socket socket = null;
    OutputStream os = null;
    try {
        InetAddress inet  = InetAddress.getByName("192.168.1.103");
        socket = new Socket(inet, 12356);

        os = socket.getOutputStream();
        os.write("我是WZL".getBytes());
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        if (os != null) {
            try {
                os.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        if (socket != null) {
            try {
                socket.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}

@Test
public void server() {

    ServerSocket ss = null;
    Socket sock = null;
    InputStream is = null;
    ByteArrayOutputStream baos = null;
    try {
        ss = new ServerSocket(12356);
        sock = ss.accept();

        is = sock.getInputStream();

        //不建议这样直接去使用，遇到UTF-8的中文会乱码
        //byte[] buffer = new byte[1024];
        //int len;
        //while ((len = is.read(buffer)) != -1){
        //    String str = new String(buffer);
        //    System.out.print(str);
        //}

        //使用ByteArrayOutputStream可以防止中文乱码问题
        baos = new ByteArrayOutputStream();
        byte[] buffer = new byte[5];
        int len;
        while ((len = is.read(buffer)) != -1){
            baos.write(buffer, 0, len);
        }

        System.out.println(baos.toString());

        System.out.println("收到了来自：" + sock.getInetAddress().getHostAddress() + "的消息");
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        if (baos != null) {
            try {
                baos.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        if (is != null) {
            try {
                is.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        if (sock != null) {
            try {
                sock.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        if (ss != null) {
            try {
                ss.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

##### 客户端发送文件到服务端

```java
@Test
public void client() {
    Socket sock = null;
    OutputStream os = null;
    FileInputStream fis = null;
    try {

        InetAddress inet = InetAddress.getByName("192.168.1.103");
        sock = new Socket(inet, 9090);

        os = sock.getOutputStream();
        fis = new FileInputStream(new File("IOTest/pic1.png"));

        //传输文件
        byte[] buffer = new byte[1024];
        int len;
        while ((len = fis.read(buffer)) != -1){
            os.write(buffer, 0, len);
        }


    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        if (fis != null) {
            try {
                fis.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        if (os != null) {
            try {
                os.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        if (sock != null) {
            try {
                sock.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}

@Test
//文件传输服务端
public void server() {
    ServerSocket ss = null;
    Socket sock = null;
    InputStream is = null;
    FileOutputStream fos = null;
    try {
        ss = new ServerSocket(9090);
        sock = ss.accept();
        is = sock.getInputStream();
        fos = new FileOutputStream(new File("~/IOTest/pic_trans.png"));

        byte[] buffer = new byte[1024];
        int len;
        while ((len = is.read(buffer)) != -1){
            fos.write(buffer, 0, len);
        }
        System.out.println("收到文件，来自：" + sock.getInetAddress().getHostAddress());

    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        if (fos != null) {
            try {
                fos.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        if (is != null) {
            try {
                is.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        if (sock != null) {
            try {
                sock.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        if (ss != null) {
            try {
                ss.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

###### 完成发送以后服务端返回值

尤其注意IO是阻塞式的，如果不停止流那么流不会停止的，需要`sock.shutdownOutput();`来停止阻塞

```java
@Test
public void client() {
    Socket sock = null;
    OutputStream os = null;
    FileInputStream fis = null;
    try {

        InetAddress inet = InetAddress.getByName("192.168.1.103");
        sock = new Socket(inet, 9090);

        os = sock.getOutputStream();
        fis = new FileInputStream(new File("IOTest/pic1.png"));

        //传输文件
        byte[] buffer = new byte[1024];
        int len;
        while ((len = fis.read(buffer)) != -1){
            os.write(buffer, 0, len);
        }
        //给出指令，停止输出文件，防止read()发生阻塞
        sock.shutdownOutput();

        //获得服务器反馈
        InputStream is = sock.getInputStream();
        ByteArrayOutputStream baos = new ByteArrayOutputStream();

        byte[] bufferr = new byte[5];
        int len1;
        while ((len1 = is.read(bufferr)) != -1){
            baos.write(bufferr, 0, len1);
        }
        System.out.println(baos.toString());

    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        if (fis != null) {
            try {
                fis.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        if (os != null) {
            try {
                os.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        if (sock != null) {
            try {
                sock.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}

@Test
//文件传输服务端
public void server() {
    ServerSocket ss = null;
    Socket sock = null;
    InputStream is = null;
    FileOutputStream fos = null;
    try {
        ss = new ServerSocket(9090);
        sock = ss.accept();
        is = sock.getInputStream();
        fos = new FileOutputStream(new File("IOTest/pic_trans.png"));

        //接收文件
        byte[] buffer = new byte[1024];
        int len;
        while ((len = is.read(buffer)) != -1){
            fos.write(buffer, 0, len);
        }
        System.out.println("收到文件，来自：" + sock.getInetAddress().getHostAddress());

        //服务器端给予客户端一个反馈
        OutputStream os = sock.getOutputStream();
        os.write("文件接收成功".getBytes());

    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        if (fos != null) {
            try {
                fos.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        if (is != null) {
            try {
                is.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        if (sock != null) {
            try {
                sock.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        if (ss != null) {
            try {
                ss.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

### UDP网络编程

```java
@Test
//发送端
public void sender() {
    DatagramSocket socket = null;
    try {
        InetAddress inet = InetAddress.getByName("192.168.1.103");
        socket = new DatagramSocket();

        String str = "我是UDP方式发送的这条信息";
        byte[] data = str.getBytes();
        DatagramPacket packet = new DatagramPacket(data, 0, data.length, inet, 9090);

        socket.send(packet);
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        if (socket != null)
            socket.close();
    }
}

@Test
//接收端
public void receiver() {
    DatagramSocket socket = null;
    try {
        socket = new DatagramSocket(9090);

        byte[] buffer = new byte[1024];
        DatagramPacket dp = new DatagramPacket(buffer, 0, buffer.length);

        socket.receive(dp);
        System.out.println(new String(dp.getData(), 0, dp.getData().length));
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        if (socket != null)
            socket.close();
    }
}
```
