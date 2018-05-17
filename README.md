# lanporxy
使用lanproxy进行内网穿透
96  VoidKing 关注
2017.09.13 22:03* 字数 1573 阅读 2001评论 3喜欢 8
内网穿透

《微信本地调试》一文中，小编提到了使用ngrok、natapp和花生壳进行内网穿透。但是，想要使用自定义域名，都是要收费的。

本文中，我们要搭建一个免费的内网穿透服务器。内网穿透服务器，可选的软件有lanproxy、frp、n2n等等，今天我们选择的是lanproxy。

原文地址：http://www.voidking.com/2017/09/13/deve-lanproxy/

准备

1、一台公网服务器（运行proxy-server）。
2、一台内网pc或服务器（运行proxy-client）。

服务端配置

安装java

1、删除自带jdk

rpm -e --nodeps `rpm -qa | grep java`
2、查看yum库中有哪些jdk版本。
yum search java | grep jdk

3、选择java-1.8.0-openjdk-devel.x86_64 : OpenJDK Development Environment版本进行安装。
yum install java-1.8.0-openjdk-devel.x86_64

默认安装目录为/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.141-1.b16.el7_3.x86_64。

4、配置环境变量
vim /etc/profile

在最后添加：

#set java environment
JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.141-1.b16.el7_3.x86_64
JRE_HOME=$JAVA_HOME/jre
CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
export JAVA_HOME JRE_HOME CLASS_PATH PATH
5、让修改立即生效
source /etc/profile

6、查看安装结果
java，javac，java -version

安装lanproxy

1、访问lanproxy下载地址，下载proxy-server-0.1.zip，上传到公网服务器。

或者，直接在服务器上下载
wget https://github.com/ffay/lanproxy/files/1274739/proxy-server-0.1.zip

curl -C - -O -L https://github.com/ffay/lanproxy/files/1274739/proxy-server-0.1.zip

2、解压安装
unzip proxy-server-0.1.zip

mv proxy-server-0.1 /usr/local/

3、修改配置文件
vim /usr/local/proxy-server-0.1/conf/config.properties
修改管理员的用户名和密码。

4、启动服务
cd /usr/local/proxy-server-0.1/bin

chmod +x startup.sh

./startup.sh

5、访问 http://host_ip:8090 ，即可看到登录界面。

nginx反向代理

1、添加域名解析local到公网ip。

2、在nginx虚拟主机配置目录中，添加local.voidking.com.conf，内容如下：

server {
    listen 80;
    server_name local.voidking.com;
    charset utf-8;
    location /{
        proxy_set_header   Host             $host;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;
        client_max_body_size       1024m;
        client_body_buffer_size    128k;
        client_body_temp_path      data/client_body_temp;
        proxy_connect_timeout      90;
        proxy_send_timeout         90;
        proxy_read_timeout         90;
        proxy_buffer_size          4k;
        proxy_buffers              4 32k;
        proxy_busy_buffers_size    64k;
        proxy_temp_file_write_size 64k;
        proxy_temp_path            data/proxy_temp;
        
        proxy_pass http://127.0.0.1:8090;
    }
}
3、测试nginx
./nginx -t，也许会提示缺少目录，那么新建目录。
mkdir -p /usr/local/nginx/data/client_body_temp

mkdir -p /usr/local/nginx/data/proxy_temp

4、重启nginx
./nginx -s reload

5、访问 http://local.voidking.com/ ，即可看到登录界面。

使用

服务端配置

1、登录lanproxy，添加客户端，输入客户端备注名称，生成随机密钥，提交添加。



2、客户端列表中，配置管理中，都会出现新添加的客户端。



3、单击配置管理中的客户端，添加配置（每个客户端可以添加多个配置）。



代理名称，推荐输入客户端要代理出去的端口，或者是客户端想要发布到公网的项目名称。
公网端口，填入一个服务器空闲端口，用来转发请求给客户端。
代理IP端口，填入客户端端口，公网会转发请求给该客户端端口。
客户端配置

1、访问lanproxy下载地址，下载proxy-client-0.1.zip，解压到喜欢的目录。

2、进入proxy-client-0.1/conf目录，修改config.properties为：

#与在proxy-server配置后台创建客户端时填写的秘钥保持一致；没有服务器可以登录 https://lanproxy.org/ 创建客户端获取秘钥
client.key=7533f855416741d88732954991668715
ssl.enable=true
ssl.jksPath=test.jks
ssl.keyStorePassword=123456

#这里填写实际的proxy-server地址；没有服务器默认即可，自己有服务器的更换为自己的proxy-server（IP）地址
server.host=local.voidking.com

#proxy-server ssl默认端口4993，默认普通端口4900
#ssl.enable=true时这里填写ssl端口，ssl.enable=false时这里填写普通端口
server.port=4993
3、进入proxy-client-0.1/bin目录，双击startup.bat，即可启动lanproxy客户端。

如果启动失败，一般是因为jdk没有安装配置成功，参考《IDEA的常用配置》中的安装jdk，安装配置jdk后再次启动即可。

4、访问地址 http://local.voidking.com:50000/ ，即可看到本地访问客户端80端口相同的页面。


至此，代理成功！

进阶配置

一个端口一个项目

假设，我们本地的4000端口开启了node服务。那么，怎么把这个服务优雅地提供给整个互联网？

1、服务端添加配置



2、启动本地node服务



3、已经启动lanproxy客户端，访问 http://local.voidking.com:50001/

此时，整个互联网都能访问到这个node项目，但是，带着端口号很不友好。那么，我们就给这个项目添加一个单独的域名。

1、添加域名解析node.local到公网ip。

2、在nginx虚拟主机配置目录中，添加node.local.voidking.com.conf，内容如下：

server {
    listen 80;
    server_name node.local.voidking.com;
    charset utf-8;
    location /{
        proxy_set_header   Host             $host;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;
        client_max_body_size       1024m;
        client_body_buffer_size    128k;
        client_body_temp_path      data/client_body_temp;
        proxy_connect_timeout      90;
        proxy_send_timeout         90;
        proxy_read_timeout         90;
        proxy_buffer_size          4k;
        proxy_buffers              4 32k;
        proxy_busy_buffers_size    64k;
        proxy_temp_file_write_size 64k;
        proxy_temp_path            data/proxy_temp;
        
        proxy_pass http://127.0.0.1:50001;
    }
}
3、重启nginx
./nginx -s reload

4、访问地址 http://node.local.voidking.com/ ，即可看到本地node服务。

一个端口多个项目

1、通过我们开放出的80端口，可以访问web根目录下的很多项目，比如在其他文章中提到过的basic项目和vkphp项目，下文以vkphp项目为例。

2、当前，vkphp项目首页是简单的文字显示。



3、通过外网访问的地址为 http://local.voidking.com:50000/vkphp

此时，整个互联网都能访问到这个vkphp项目，但是，带着端口号和项目名，感觉像是个欺诈网站。那么，我们能否给这个项目添加一个单独的域名呢？当然也是可以的。

1、添加域名解析vkphp.local到公网ip。

2、在nginx虚拟主机配置目录中，添加vkphp.local.voidking.com.conf，内容如下：

server {
    listen 80;
    server_name vkphp.local.voidking.com;
    charset utf-8;
    location /{
        proxy_set_header   Host             $host;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;
        client_max_body_size       1024m;
        client_body_buffer_size    128k;
        client_body_temp_path      data/client_body_temp;
        proxy_connect_timeout      90;
        proxy_send_timeout         90;
        proxy_read_timeout         90;
        proxy_buffer_size          4k;
        proxy_buffers              4 32k;
        proxy_busy_buffers_size    64k;
        proxy_temp_file_write_size 64k;
        proxy_temp_path            data/proxy_temp;
        
        proxy_pass http://127.0.0.1:50000;
    }
}
3、重启nginx
./nginx -s reload

4、打开本地apache的http-vhosts.conf，添加配置：

<VirtualHost *:80> #laragon magic!
    DocumentRoot "C:/laragon/www/vkphp/"
    ServerName vkphp.local.voidking.com
    ServerAlias vkphp.local.voidking.com
</VirtualHost>
5、重启本地apache

6、访问地址 http://vkphp.local.voidking.com/ ，即可看到本地vkphp项目。

有趣的是，访问时该地址会自动在后面加上vkphp，成为 http://vkphp.local.voidking.com/vkphp/

结语

由上配置我们发现，nginx的反向代理非常好用。稍微调整，便可以适应大多数项目，实在是美化url的神器，哇咔咔。

书签

lanproxy源码地址

业余草推荐一款局域网（内网）穿透工具lanproxy

frp源码地址

frp中文文档

使用frp实现内网穿透

n2n源码地址

n2n内网穿透神器(一条命令实现穿透)

n2n内网穿透神器
