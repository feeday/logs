Linux 系统上常用的防火墙命令代码。

## 常用命令
```
systemctl status firewalld   #查看系统自带防火墙是否运行
iptables -L -n               #查看防火墙规则
systemctl stop firewalld     #停止firewall防火墙
systemctl disable firewalld  #禁止firewall开机启动
systemctl mask firewalld     #禁用firewalld服务
yum install  -y iptables  iptables-services   #安装iptables防火墙
systemctl start iptables         #启动防火墙
systemctl status iptables        #查看防火墙是否启动

firewall-cmd --list-ports  #查看已经开放的端口
firewall-cmd --zone=public --add-port=80/tcp --permanent  #开启端口

#配置文件目录 /etc/sysconfig/iptables
iptables -P INPUT ACCEPT   #先允许所有,不然有可能会杯具
iptables -F      #清空所有的防火墙规则
iptables -X      #清空所有自定义规则
iptables -Z      #所有计数器归0
iptables -A INPUT -p tcp --dport 22 -j ACCEPT   #开放22端口
iptables -A INPUT -p tcp --dport 21 -j ACCEPT   #开放21端口(FTP)
iptables -A INPUT -p tcp --dport 80 -j ACCEPT   #开放80端口(HTTP)
iptables -A INPUT -p tcp --dport 443 -j ACCEPT  #开放443端口(HTTPS)
iptables -A INPUT -p icmp --icmp-type 8 -j ACCEPT  #允许ping
iptables -A INPUT -i lo -j ACCEPT    #允许来自于lo接口的数据包(本地访问)
iptables -P INPUT DROP  #其他入站一律丢弃
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT   #允许由服务器本身请求的数据通过
iptables -A OUTPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -P OUTPUT ACCEPT  #所有出站一律绿灯
iptables -P FORWARD DROP   #所有转发一律丢弃
iptables -A OUTPUT -j ACCEPT         #允许所有本机向外的访问
service iptables save                #保存上述规则
systemctl restart iptables.service   #重启服务
systemctl enable iptables            #设置开机启动
systemctl start iptables.service    #启动服务
systemctl status iptables.service   #查看服务状态
```

## 屏蔽IP段
```
iptables -I INPUT -s 123.45.6.7 -j DROP       #屏蔽单个IP的命令
iptables -I INPUT -s 123.0.0.0/8 -j DROP      #封整个段即从123.0.0.1到123.255.255.254的命令
iptables -I INPUT -s 124.45.0.0/16 -j DROP    #封IP段即从123.45.0.1到123.45.255.254的命令
iptables -I INPUT -s 123.45.6.0/24 -j DROP    #封IP段即从123.45.6.1到123.45.6.254的命令是
```