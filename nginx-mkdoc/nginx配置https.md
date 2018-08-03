# Linux nginx配置https

最近在研究Nginx，用到了https协议，这里总结一下开发过程，积累学习经验。

# **一、SSL证书申请**

我查看文章发现现在主流的ssl证书有openssl和startssl，这里我们使用openssl证书来开发

##**1、安装openssl支持包**

**（1）首先查看服务器有没有安装openssl 支持包**

	rpm -qa | grep  openssl

如果有结果输出，说明已经有了openssl 包的支持

如果没有结果输入 通过命令

	yum instance -y openssl            yum install -y openssl-devel 

 来安装openssl 支持

## **2、生成私钥和csr文件**

**（1）进入想要生成私钥的目录，如果没有就创建一个**

```
#进入nginx的安装目录
cd /usr/local/nginx

#创建一个存放私钥的文件夹（自定义）
mkdir key

#进入key文件夹
cd key
```

**（2）生成私钥**

```
openssl genrsa -out server.key 2048
```

![](C:\Users\z15746\Desktop\Nginx\Nginx-https\create_keys.png)

**（3）生成csr文件**

```
openssl req -new -key server.key -out certreq.csr
```

**以下黑色标识文字仅供参考，请根据商户自己实际情况进行填写**

```
Country Name： CN                      //您所在国家的ISO标准代号，中国为CN
State or Province Name：zhengjiang       //您单位所在地省/自治区/直辖市
Locality Name：hangzhou                 //您单位所在地的市/县/区
Organization Name： Tencent Technology (Shenzhen) Company Limited                 //您单位/机构/企业合法的名称 
Organizational Unit Name： R&D         //部门名称 
Common Name： www.example.com     //通用名，例如：www.itrus.com.cn。此项必须与您访问提供SSL服务的服务器时所应用的域名完全匹配。
Email Address：                          //您的邮件地址，不必输入，直接回车跳过
"extra"attributes                        //以下信息不必输入，回车跳过直到命令执行完毕。
```
**（4）利用上面生成的秘钥  和证书请求文件来生成证书 **

`openssl x509 -req -days 365 -in certreq.csr -signkey server.key  -out  mydomain.crt`

![](C:\Users\z15746\Desktop\Nginx\Nginx-https\producte.png)

**（5）现在开始配置nginx ** 

核心配置直接贴上来， 方便大家直接粘贴。

        #HTTPS server
    
         server {
    
            listen       443 ssl;
    
            server_name  localhost;
    
        #ssl证书文件位置(常见证书文件格式为：crt/pem)
        ssl_certificate      ../key/mydomain.crt;
        #ssl证书key位置
        ssl_certificate_key  ../key/server.key;
    
        ssl_session_cache    shared:SSL:1m;
        ssl_session_timeout  5m;
    
        ssl_ciphers  HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers  on;
    
        location / {
            root   html;
            index  index.html index.htm;
        }
    
        location ~ \.(gif|jpg|png)$ {
            root /data/images;
        }
       }
   配置完成后千万要记得重启 nginx 服务器，让配置生效。

​    检查一下配置文件是否合法：

​    `/usr/local/nginx/sbin/nginx -t`

​    `/usr/local/nginx/sbin/nginx -s reload`

![](C:\Users\z15746\Desktop\Nginx\Nginx-https\nginx-config.png)

重启过程中如果出现：

[emerg] 10464#0: unknown directive "ssl" in /usr/local/nginx-0.6.32/conf/nginx.conf 的问题，则需要重新编译nginx 

**（6）查看编译安装nginx 的时候，编译安装了哪些模块** 

`/usr/local/nginx/sbin/nginx -V`

![](C:\Users\z15746\Desktop\Nginx\Nginx-https\ssl.png)

**（7）访问https服务**

![](C:\Users\z15746\Desktop\Nginx\Nginx-https\view.png)