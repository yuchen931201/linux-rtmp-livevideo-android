本文目的:

在VPS服务器上配置一个直播环境，在Android&iOS客户端实现,直播推流到服务器上,在任意播放器上实现,拉取服务器上的流观看直播。(附android源码)

服务器环境:


LSB Version:    :core-4.1-amd64:core-4.1-noarch

Distributor ID: CentOS

Description:    CentOS Linux release 7.2.1511 (Core) 

Release:        7.2.1511

Codename:       Core

使用下面命令可查看服务器信息

#lsb_release -a


自我介绍：我是一个Android程序员，在一家创业公司工作，公司需要做一个直播应用，因为公司没有服务器运维的人员，所以我花了一个星期的时间，参考网络上的帖子,完成了自己的直播系统。

如果你认为这篇文章对你有帮助，请在GitHub的右上角上给我一个star，这里面有服务器所需的配置文件,一些工具包,和Android端的推流源代码,非常感谢！

https://github.com/yuchen931201/linux-rtmp-livevideo-android/blob/master/README.md



(一) 服务器篇:

首先你得购买一台服务器,可在任意服务商购买,本文是在阿里云上购买的VPS(千万别买云虚拟主机了);

服务器理解:服务器其实就是一台高配电脑,全年7X24小时的为你服务器, VPS(virtual private server)可以理解为电脑里分配出来的一块空间,并利用虚拟机创建了一台新的服务器,它拥有独立的IP,独立的内存,独立的带宽...可独立开关机,相当于一台真实的机器,而"云虚拟主机"只是一台服务器中分配一块内存供你的站点使用,按照级别和价格排序,都是 实体主机电脑服务器 > VPS >云虚拟主机.

--------------------------------废话结束的分割线--------------------------------

(1)准备nginx-rtmp-module , Git 和 openssl

1.使用yum安装git:


[java] view plain copy
yum install git  
2.下载nginx-rtmp-module,官方github地址：https://github.com/arut/nginx-rtmp-module


[java] view plain copy
git clone https://github.com/arut/nginx-rtmp-module.git  
3.yum安装openssl:

[java] view plain copy
yum -y install openssl openssl-devel   


(2)下载安装Nginx,官方网站为：http://nginx.org/en/download.html

1.下载nginx解压,并添加rtmp和openssl支持

[java] view plain copy
wget http://nginx.org/download/nginx-1.8.1.tar.gz    
tar -zxvf nginx-1.8.1.tar.gz    
cd nginx-1.8.1    
./configure --prefix=/usr/local/nginx  --add-module=../nginx-rtmp-module  --with-http_ssl_module      
make && make install   
2.如果你已经安装了nginx, 则只需要在nginx的源码目录添加rtmp支持,nginx的源码目录与安装目录?查看安装目录
[java] view plain copy
whereis nginx  
而我们这里是要找源码目录,这就需要你自己找了, 这个取决于你当时下载nginx时存放的目录, 推荐一个命令供你快速查到它,首先查询自己nginx的版本
[java] view plain copy
/usr/local/nginx/sbin/nginx -v  
[java] view plain copy
nginx version: nginx/1.8.1  
如果输出如上,那么你的nginx源码目录可能为:
nginx-1.8.1

再使用find命令查找其位置
[java] view plain copy
find / -name nginx-1.8.1  
结果我的装在 这个位置,进入此目录里面有一个绿色的configure可执行文件,那就说明找对了
/root/nginx-1.8.1

然后继续执行第一步剩下的内容

[java] view plain copy
./configure --prefix=/usr/local/nginx  --add-module=../nginx-rtmp-module  --with-http_ssl_module      
make && make install   
3.如果你以前使用的yum安装的,则需要先停止nginx运行,并卸载nginx,重新使用源码安装的方式即做第一步的操作,卸载命令

[java] view plain copy
yum remove nginx  


(3)修改nginx配置文件,没有vim的可以yum install vim 安装一个,或者用vi也行

[java] view plain copy
vim /usr/local/nginx/conf/nginx.conf   

修改内容如下,在http的上面加入,这里只是简单的配置,更多配置点击这里:

[java] view plain copy
rtmp {      
    server {      
        listen 1935;  #监听的端口    
        chunk_size 4000;      
             
        application hls {  #rtmp推流请求路径    
            live on;      
            hls on;      
            hls_path /usr/share/nginx/html/hls;      
            hls_fragment 5s;      
        }      
    }      
}  
并修改http中的server为如下:

[java] view plain copy
server {    
    listen       81;    
    server_name  localhost;    
    #charset koi8-r;    
    #access_log  logs/host.access.log  main;    
    
    location / {    
        root   /usr/share/nginx/html;    
        index  index.html index.htm;    
    }    
    
    #error_page  404              /404.html;    
    
    # redirect server error pages to the static page /50x.html    
    #    
    error_page   500 502 503 504  /50x.html;    
    location = /50x.html {    
        root   html;    
    }  

:wq 保存并退出

1.在/usr/share/目录下创建nginx/html/hls

[java] view plain copy
cd /usr/share  
mkdir nginx  
cd nginx  
mkdir html  
cd html  
mkdir hls  
chomd -R 777 /usr/share/nginx  

2.回到/usr/share/目录下,查看nginx及其子目录是否都有读写权限

[java] view plain copy
ls -ld nginx/   

(4)最后一步启动Nginx

[java] view plain copy
/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf   

现在服务端就配置好了.


(二) 客户端篇:
本人的老本行是Android开发,所以只有android的源码,不过朋友也推荐过一个ios的源码,网上都能搜到的,这里也链接一下吧,我就不做源码详解,没什么好讲的代码很少;




(三) 测试篇:

推流地址:  rtmp://****:1935/hls/test

拉流地址(观看地址):http://*****:81/hls/test.m3u8

*替换为你的IP地址,推流使用源码推或者直接使用obs来推流了,Mac版的obs可在这里的centos-package-utils目录中下载URL:https://github.com/yuchen931201/linux-rtmp-livevideo-android
