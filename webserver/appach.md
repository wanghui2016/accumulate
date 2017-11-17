httpd: Could not reliably determine the server's fully qualified domain name, using 127.0.0.1 for ServerName
解决办法非常简单：
root@raspberrypi:/etc/apache2# vi apache2.conf 
# Include the virtual host configurations:
ServerName  localhost:80
Include sites-enabled/
root@raspberrypi:/etc/apache2# service apache2 restart
[ ok ] Restarting web server: apache2 ... waiting ..

#vim /web/apache/conf/httpd.conf (在这里/web/apahce是我安装apache的目录，你默认安装的话应该是/usr/local/apache2/icons)
找到#ServerName www.example.com:80   把#去掉，再重启apache即可没事了。

现象：
bogon:~/webserver/httpd-2.0.59 # /usr/local/apache2/bin/apachectl start
httpd: Could not determine the server's fully qualified domain name, using 127.0.0.1 for ServerName
httpd (pid 20183) already running

這個問題應該是沒有在 /etc/httpd/conf/httpd.conf 中設定 ServerName  

vi /usr/local/apache2/conf/httpd.conf

最简单的，修改httpd.conf文件，增加:
ServerName www.example.com:80
我的改为:

ServerName www.example.com:80


再次启动就正常了!
------------------------------------------------------------------------------------------------------------------------------------------------------------
apache2: Could not reliably determine the server's fully qualified domain name, using 127.0.1.1 for ServerName
解决方案：
用记事本打开 httpd.conf
将里面的 #ServerName localhost:80 注释去掉即可。
再执行 httpd
然后可以通过浏览器访问 http://localhost:80 ，如果页面显示 “It works！” ，即表示apache已安装并启动成功。
++++++++++++++++++++++++++++++++++++++++++++
using localhost.localdomain for ServerName  说不能确认服务器完全确认域名 localhost.localdoman  这个问题怎么解决
最佳答案： 
vi /etc/httpd/conf/httpd.conf   加入一句  ServerName  localhost:80
解决步骤：
为了解决这个问题，你需要编辑下面这个httpd.conf文件，打开它并根据如下操作进行编辑：
[c-sharp] view plaincopy
sudo gedit /etc/apache2/httpd.conf  
默认的httpd.conf是个空文件，现在向里面加入如下内容：
[c-sharp] view plaincopy
ServerName localhost  
保存并退出。
------------------------------------------------------------------------------------------------------------------------------------------------------------

AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using localhost.localdomain. Set the 'ServerName' directive globally to suppress this message
httpd (pid 7907) already running
修改：去掉注释即可。
# If your host doesn't have a registered DNS name, enter its IP address here.
#
ServerName www.example.com:8080


解决方案：
进入apache的安装目录:
Windows : D:\Program Files\Apache Software Foundation\Apache2.2\conf
linux : /usr/local/apache/conf
用记事本打开httpd.conf
将里面的#ServerName localhost:80注释去掉即可。
再执行httpd
然后可以通过浏览器访问http://localhost：80，如果页面显示“It works！”，即表示apache已安装并启动成功。
来源： http://www.justwinit.cn/post/3140/
