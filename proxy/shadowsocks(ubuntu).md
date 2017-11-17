####  1  安装shadowsocks
        sudo apt update
        sudo apt install shadowsocks-libev
####  2 修改配置文件,并启动服务(设置服务器ip和密码端口已及加密方式)
        sslocal -c /etc/shadowsocks-libev/config.json -d start
####  3 安装配置polipo
        git clone https://github.com/jech/polipo.git
        cd polipo
        make all
        make install

        [root@thatsit polipo]# cat /etc/polipo/config
        socksParentProxy = "127.0.0.1:1080"
        socksProxyType = socks5
        logFile = /var/log/polipo
        logLevel = 99
        logSyslog = true
        [root@thatsit polipo]# /usr/local/bin/polipo -c /etc/polipo/config
        [root@thatsit polipo]# ss -lntup|grep polipo
        tcp LISTEN 0 128 127.0.0.1:8123 *:* users:(("polipo",23399,5))
        [root@thatsit polipo]#

        export http_proxy=http://localhost:8123
