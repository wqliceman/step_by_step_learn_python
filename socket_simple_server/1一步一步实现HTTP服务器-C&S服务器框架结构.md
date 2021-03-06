## 概述

继上一篇[XmlRPC简介及使用](https://mp.weixin.qq.com/s/CAnV2bELbB7HhH6UbLsI9A )对PRC做了个简单介绍，其实RPC就是网络编程的一个实际应用，不过上次从上层讲解，可能还不能很好的理解一个简单**Client/Server** (客户端/服务器,后面简称**C/S)**网络通信框架，本次就使用Pyhthon来说明一个完整的Socket服务器通信的框架，同时**本文先从TCP进行讲解**(具体TCP和UDP系列后面进行细说)。

本文主要从以下几个方面进行阐述：

- 服务器和客户端通信简单介绍
- Socket函数简介
- 服务端代码实现
- 客户端代码实现

通过以上4个步骤，对网络编程有个简单理解，方便后文层层深入作铺垫

## Socket简介

**Socket（套接字）**是计算机网络数据结构，在任何类型的通信开始之前，网络应用程序都必须创建套接字。就类似于网线两端的水晶头，如果没得这个东西无法建立连接。Socket是在应用层和传输层之间的一个抽象层，它把TCP/IP层复杂的操作抽象为几个简单的接口供应用层调用已实现进程在网络中通信。 Socket起源于UNIX，在Unix一切皆文件哲学的思想下，Socket是一种"打开—读/写—关闭"模式的实现，服务器和客户端各自维护一个"文件"（如果客户端和服务器在同一台机器上，进行进程间通信，可以是一个文件句柄；如果在网络上这个就相对复杂些），在建立连接打开后，可以向自己文件写入内容供对方读取或者读取对方内容，通讯结束时关闭文件。 

### 套接字编程API

#### 通用套接字方法

- **socket(family=AF_INET, type=SOCK_STREAM, proto=0)** #创建套接字

- **recv(buffersize, flags=None)**  #接收TCP消息

- **send(data, flags=None)** #发送TCP消息

#### 服务端套接字方法

- **bind(address)** #将地址(主机名、端口号)绑定到套接字上

- **listen(backlog=None)** #监听端口

- **accept()** #阻塞等待客户端的连接(被动接收客户端连接)

#### 客户端套接字方法

- **connect(address)** #连接服务器指定端口(主动发起TCP连接)

### Socket通信路程图

![socket通信流程](http://ww1.sinaimg.cn/large/665db722gy1frq9gi39mdj20mr0m075f.jpg)



## 服务端

既然是C/S模型，那么首先得先说说Server端，一般服务器的框架如下：

1. 创建服务器socket套接字
2. 套接字与地址绑定（此处地址为**ip:port**）
3. 监听端口
4. 服务器进入循环阶段(一般都是死循环，因为要提供持续服务)
5. 开始接受客户端的连接
6. 此时就开始进行消息的接收和发送
7. 关闭客户端socket套接字
8. 关闭服务器socket套接字(次数一般都不会执行，因为是死循环嘛，除非捕获异常)

基于上面几个步骤，此处使用Python代码进行实现时间戳服务器

```python
# TimestampeServer.py
import time
import socket

COMMAND = 'timetampe'

def start_server(host, port):
    #1. 创建socket句柄
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as tcp_server:
        if int(port) >0: #端口必须大于零
            addr = (host, port)
            tcp_server.bind(addr) #2. 绑定端口，参数为ip:port元组
            tcp_server.listen(10) #3. 开始监听端口，
            while True: #4.
                print('*'*10 + 'now waiting for client connect' + '*'*10)
                tcp_client, client_addr = tcp_server.accept() #5. 等待客户端连接
                print('new client : %s' % client_addr[0] + ':' + str(client_addr[1]))
                recv_data = tcp_client.recv(1024) #6.1 从客户端获取命令
                recv_data = str(recv_data, encoding='utf-8')
                if recv_data == COMMAND: #是否匹配命令
                    current_timestamp = time.ctime() #获取当前时间
                    tcp_client.send(bytes(current_timestamp, 'utf-8')) #6.2 返回当前时间给客户端
                tcp_client.close() #7. 关闭socket句柄
    #8. 此处使用with语句，超出作用于自动释放

if __name__ == '__main__':
    HOST = '127.0.0.1'
    PORT = 8099
    start_server(HOST, PORT)
```

通过再命令行运行 *python timestampeServer.py*程序即可

```bash
C:\ProgramData\Anaconda3\python.exe .\TimestampeServer.py
**********now waiting for client connect**********
```



## 客户端

对于客户端稍微简单点：

1. 创建客户端socket套接字
2. 尝试连接服务器
3. 发送/接收数据
4. 关闭客户端套接字

```python
# Client.py
import socket

COMMAND = 'timetampe'   #通过该命令获取服务器时间戳

def main(server, port):
    #1. 创建socket句柄
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as tcp_client:
        addr = (server, port)
        tcp_client.connect(addr) #2. 建立连接，此处需要一个ip:port的元组
        tcp_client.send(bytes(COMMAND, 'utf-8')) #3.1 发送命令, send需要byte数组参数
        current_timestampe = tcp_client.recv(1024) #3.2 接收服务器的响应
        if current_timestampe:
            print('current timestampe: %s' % str(current_timestampe, encoding='utf-8'))
        else:
            print('get timestampe failed')
     #4. 此处使用with语句，超出作用于自动释放

if __name__ == '__main__':
    SERVER = '127.0.0.1'
    PORT = 8099
    main(SERVER, PORT)
```

通过再命令行运行 *python Client.py*程序即可

```bash
C:\ProgramData\Anaconda3\python.exe .\Client.py
current timestampe: Sun May 27 21:43:58 2018
```

欢迎关注交流共同进步
![奔跑阿甘](http://ww1.sinaimg.cn/large/665db722gy1frf76owwqjj2076076q3e.jpg)