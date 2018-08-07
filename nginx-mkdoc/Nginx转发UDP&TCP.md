## 编译安装Nginx

从1.9.0开始，nginx就支持对TCP的转发，而到了1.9.13时，UDP转发也支持了。提供此功能的模块为ngx_stream_core。不过Nginx默认没有开启此模块，所以需要手动安装。
```
cd /usr/local/src
cd nginx-1.12.1
./configure --prefix=/usr/local/nginx --with-stream 
make && make install
```


## 配置Nginx
### TCP转发
目标：通过3000端口访问本机Mysql(其中mysql使用yum安装，默认配置文件)

**_/usr/local/nginx/conf/nginx.conf配置如下：_**

```
worker_processes  auto;

events {
    use epoll;
    worker_connections  1024;
}


stream {
    server {
        listen 3000;
        proxy_pass 127.0.0.1:3306;

    }
}
```
首先，先通过3306端口访问mysql：


接着，使用3000端口访问mysql：


然后，启动Nginx：
最后使用3000端口访问mysql：


### UDP转发
目标： 发送UDP数据到3000端口，3001端口可以接收

**_/usr/local/nginx/conf/nginx.conf配置如下：_**
```

worker_processes  auto;

events {
    use epoll;
    worker_connections  1024;
}


stream {
    server {
        listen 3000 udp;
        proxy_pass 127.0.0.1:3001;

    }
}
```
这里写一个**my_socket_server.py**侦听在3001端口，用于接收UDP数据：
```
# coding=utf-8

import socket

sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

sock.bind(('127.0.0.1', 3001))

print ('start server on [%s]:%s' % ('127.0.0.1', 3001))

while True:
    data, addr = sock.recvfrom(1024)
    print ('Received from %s:%s' % (addr,data))
    sock.sendto(b'Hello, %s!' % data, addr)
```

再写一个**my_socket_client.py**用于向Nginx侦听的3000端口发送数据：
```
# coding=utf-8

import socket

s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

while True:
    data = raw_input('Input msg: ')
    if len(data) == 0:
        continue
    s.sendto(data.encode(), ('127.0.0.1', 3000))
    print (s.recv(1024).decode('utf-8'))
```

同时运行两个脚本，在client端发送数据：

