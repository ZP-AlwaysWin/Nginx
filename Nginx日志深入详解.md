## Nginx日志深入详解

### 一、日志分类

Nginx日志主要分为两种：访问日志和错误日志。日志开关默认在Nginx配置文件（/etc/nginx/nginx.conf）中设置，两种日志都可以选择性关闭，默认都是打开的。

#### 1、访问日志

访问日志主要记录客户端访问Nginx的每一个请求，格式可以自定义。通过访问日志，你可以得到用户地域来源、跳转来源、使用终端、某个URL访问量等相关信息。Nginx中访问日志相关指令主要有两条：

##### （1）log_format

log_format用来设置日志格式，也就是日志文件中每条日志的格式，具体如下：
log_format name(格式名称) type(格式样式)
举例说明如下：

![](C:\Users\z15746\Desktop\Nginx日志\464291-20170522230449273-1819232912.png)
	                                    **图一、log_format默认格式**

log_format main '\$server_name \$remote_addr - \$remote_user [\$time_local] "$request" '
'\$status \$uptream_status \$body_bytes_sent "$http_referer" '
'"\$http_user_agent" "$http_x_forwarded_for" '
'\$ssl_protocol \$ssl_cipher \$upstream_addr \$request_time $upstream_response_time';

| $remote_addr          | 客户端的ip地址(代理服务器，显示代理服务ip)           |
| --------------------- | ---------------------------------------------------- |
| $remote_user          | 用于记录远程客户端的用户名称（一般为“-”）            |
| $time_local           | 用于记录访问时间和时区                               |
| $request              | 用于记录请求的url以及请求方法                        |
| $status               | 响应状态码，例如：200成功、404页面找不到等。         |
| $body_bytes_sent      | 给客户端发送的文件主体内容字节数                     |
| $http_user_agent      | 用户所使用的代理（一般为浏览器）                     |
| $http_x_forwarded_for | 可以记录客户端IP，通过代理服务器来记录客户端的ip地址 |
| $http_referer         | 可以记录用户是从哪个链接访问过来的                   |



**上面红色部分为Nginx默认指定的格式样式，每个样式的含义如下：**

1. $server_name：虚拟主机名称。
2. $remote_addr：远程客户端的IP地址。
3. -：空白，用一个“-”占位符替代，历史原因导致还存在。
4. $remote_user：远程客户端用户名称，用于记录浏览者进行身份验证时提供的名字，如登录百度的用户名scq2099yt，如果没有登录就是空白。
5. [$time_local]：访问的时间与时区，比如18/Jul/2012:17:00:01 +0800，时间信息最后的"+0800"表示服务器所处时区位于UTC之后的8小时。
6. $request：请求的URI和HTTP协议，这是整个PV日志记录中最有用的信息，记录服务器收到一个什么样的请求
7. $status：记录请求返回的http状态码，比如成功是200。
8. $uptream_status：upstream状态，比如成功是200.
9. $body_bytes_sent：发送给客户端的文件主体内容的大小，比如899，可以将日志每条记录中的这个值累加起来以粗略估计服务器吞吐量。
10. $http_referer：记录从哪个页面链接访问过来的。 
11. $http_user_agent：客户端浏览器信息
12. \$http_x_forwarded_for：客户端的真实ip，通常web服务器放在反向代理的后面，这样就不能获取到客户的IP地址了，通过$remote_add拿到的IP地址是反向代理服务器的iP地址。反向代理服务器在转发请求的http头信息中，可以增加x_forwarded_for信息，用以记录原有客户端的IP地址和原来客户端的请求的服务器地址。
13. $ssl_protocol：SSL协议版本，比如TLSv1。
14. $ssl_cipher：交换数据中的算法，比如RC4-SHA。 
15. $upstream_addr：upstream的地址，即真正提供服务的主机地址。 
16. $request_time：整个请求的总时间。 
17. $upstream_response_time：请求过程中，upstream的响应时间。

**访问日志中一个典型的记录如下：**
192.168.1.102 - scq2099yt [18/Mar/2013:23:30:42 +0800] "GET /stats/awstats.pl?config=scq2099yt HTTP/1.1" 200 899 "http://192.168.1.1/pv/" "Mozilla/4.0 (compatible; MSIE 6.0; Windows XXX; Maxthon)"
**需要注意的是：log_format配置必须放在http内，否则会出现如下警告信息：**
nginx: [warn] the "log_format" directive may be used only on "http" level in /etc/nginx/nginx.conf:97

#####（2）access_log

access_log指令用来指定日志文件的存放路径（包含日志文件名）、格式和缓存大小，具体如下：
access_log path(存放路径) [format(自定义日志格式名称) [buffer=size | off]]
举例说明如下：
access_log logs/access.log main;
如果想关闭日志，可以如下：
access_log off;
能够使用access_log指令的字段包括：http、server、location。
需要注意的是：Nginx进程设置的用户和组必须对日志路径有创建文件的权限，否则，会报错。



#### 2、错误日志

​	错误日志主要记录客户端访问Nginx出错时的日志，格式不支持自定义。通过错误日志，你可以得到系统某个服务或server的性能瓶颈等。因此，将日志好好利用，你可以得到很多有价值的信息。错误日志由指令error_log来指定，具体格式如下：
error_log path(存放路径) level(日志等级)
path含义同access_log，level表示日志等级，具体如下：
[ debug | info | notice | warn | error | crit ]
从左至右，日志详细程度逐级递减，即debug最详细，crit最少。
**举例说明如下：**
error_log logs/error.log info;
需要注意的是：error_log off并不能关闭错误日志，而是会将错误日志记录到一个文件名为off的文件中。
正确的关闭错误日志记录功能的方法如下：
error_log /dev/null;
上面表示将存储日志的路径设置为“垃圾桶”。



### 二、切割日志脚本

​	nginx的日志文件没有rotate功能。编写每天生成一个日志，我们可以写一个nginx日志切割脚本来自动切割日志文件。

​	第一步就是重命名日志文件，不用担心重命名后nginx找不到日志文件而丢失日志。在你未重新打开原名字的日志文件前，nginx还是会向你重命名的文件写日志，[Linux](http://lib.csdn.net/base/linux)是靠文件描述符而不是文件名定位文件。

​	第二步向nginx主进程发送USR1信号。nginx主进程接到信号后会从配置文件中读取日志文件名称，重新打开日志文件(以配置文件中的日志名称命名)，并以工作进程的用户作为日志文件的所有者。重新打开日志文件后，nginx主进程会关闭重名的日志文件并通知工作进程使用新打开的日志文件。工作进程立刻打开新的日志文件并关闭重名名的日志文件。然后你就可以处理旧的日志文件了。[或者重启nginx服务]。

        #!/bin/bash
        #设置日志文件存放目录
        LOG_HOME="/usr/local/software/nginx/logs/"
    
        #备分文件名称
        LOG_PATH_BAK="$(date -d yesterday +%Y%m%d%H%M)".abc.access.log
    
        #重命名日志文件
        mv ${LOG_HOME}/abc.access.log ${LOG_HOME}/${LOG_PATH_BAK}.log
    
        #向nginx主进程发信号重新打开日志 
        kill -USR1 `cat /usr/local/software/nginx/logs/nginx.pid`
