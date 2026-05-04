centos 重置数据库 root 密码。

- 编辑MySQL配置文件：

```
vim /etc/my.cnf
```
- 按【I】键在[mysqld]的段中插入：
 
```
skip-grant-tables
```

- 此段命令应该是跳过相关验证，当改完密码为了安全建议删除或者注释掉。
- 按【ESC】键退出，在光标处输入【:wq】保存关闭配置文件。

- 停止重启MySQL：

```
systemctl stop mysqld
systemctl start mysqld
```

- 登陆MySQL重置密码：

```
mysql
mysql> use mysql
mysql> update user set password=password("12345678") where user='root'
mysql> flush privileges
exit
```
- 【12345678】就是重置完的root密码。