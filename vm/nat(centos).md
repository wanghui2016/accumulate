在使用CentOS虚拟机时，出现了无法上网的情况，使用主机ping虚机地址可以ping通，而虚机ping不通主机，同时虚机也无法ping通其他的网址或ip，显示内容为Network is unreachable，后来经过在网上查找解决方法，解决问题，记录如下：
 
    首先打开服务，在services.msc中将VMware的DHCP和NAT服务开启。并修改虚机的接入方式，可以在“编辑虚拟网络”中查看，如下图
CentOS虚拟机NAT方式无法上网解决方法
 
打开后如下
VMnet0是桥接方式，VMnet1是Host-only方式，VMnet8是NAT方式，子网IP可以自己设置，见1，修改后，需要把2,3中的地址段同时对应修改。
CentOS虚拟机NAT方式无法上网解决方法
这时候最好把除了NAT外其它两个连接方式停掉，将1上面，connect的勾去掉就可以了

之后需要在虚机设置中选择NAT连接方式，，如果没有网络连接方式需要自己添加一下。以上这些设置方法网上有很多，不再赘述。
CentOS虚拟机NAT方式无法上网解决方法


之后仍旧无法联网的，需要打开虚机看看虚机的网络设置了。命令如下
#vi /etc/sysconfig/network-scripts/ifcfg-eth0
其中部分内容如下：
DEVICE=eth0  #设备名称
BOOTPROTO=dhcp  #连接方式，dhcp会自动分配地址，此时不需要在下面设置ip和网关
HWADDR=00:0C:29:AD:66:9F  #硬件地址，不要修改
ONBOOT=yes  #yes表示启动就执行该配置，需要改为yes
网上会有些方法需要在这里添加ip地址，子网掩码，dns之类的，之前安装这些方法试验过，都不行，而且添加的这些内容后来还影响到了上网，所以，不建议采用那些方式添加这些内容。
 
修改完后需要重启网络设置，可以
# service network restart
或者
# /etc/init.d/network restart
此时如果还是无法连接网络，再回到物理主机，查看网络连接中的本地连接的共享是否打开，在状态->属性->共享中查看，如果没有共享选项卡，就找百度。如果共享已经打开，将Host-Only Network和VMnet8中的ipv4和ipv6服务停掉，前面的勾去掉
CentOS虚拟机NAT方式无法上网解决方法

 
至此，我的虚机网络连接正常了

来源： http://blog.sina.com.cn/s/blog_55b497690101fgxi.html
