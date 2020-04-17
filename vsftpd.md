CentOS7安装配置VSFTPD（FTP）服务详解

[toc]

#### 安装VSFTPD服务

* yum安装

```shell
yum -y install vsftpd
```

- 查询安装位置

```shell
which vsftpd
whereis vsftpd
```

- 开启vsfpd服务

```java
// 开启服务
systemctl start vsftpd 或者service vsftpd start都行
// 查询服务状态
systemctl status vsftpd
// 配置服务开机自启
systemctl enable vsftpd
```

- 创建防火墙策略（开发端口）

<!-- 关闭防火墙或者干脆永久关闭防火墙 -->

```shell
firewall-cmd --zone=public --permanent --add-port=21/tcp

firewall-cmd --zone=public --permanent --add-service=ftp

firewall-cmd –reload
```

***永久关闭防火墙***

![微信截图_20200417105848](https://i.loli.net/2020/04/17/S5B3k7fdZ4sgFzJ.png)

#### 配置

* 先备份config文件

```shell
# 切换到/etc/vsftpd
cd /etc/vsftpd
# 复制备份
cp vsftpd.conf vsftpd.conf.bak
# 去除注释后空白行，编辑的时候清爽多了
grep -Ev '^$|#' vsftpd.conf.bak > vsftpd.conf
```

* 编辑vsftpd.conf

```shell
vi /etc/vsftpd/vsftpd.conf
```

```shell
# 禁用匿名用户
anonymous_enable=NO
# 启动本地用户
local_enable=YES
# 开启登录用户上传文件权限
write_enable=YES
# 登录用户只能访问自己的目录
chroot_local_user=YES
allow_writeable_chroot=YES
# 开启可登录用户列表管理
userlist_enable=YES
# 用户列表位置
userlist_file=/etc/vsftpd/user_list
# 不拒绝用户列表
userlist_deny=NO
# 这一行非常重要，别问为什么
pam_service_name=vsftpd
```

[vsfptd官方详细配置](http://vsftpd.beasts.org/vsftpd_conf.html)

* 编辑/etc/pam.d/vsftpd

```shell
#注释auth required pam_shells.so
```

* 重启服务

```shell
service vsftpd restart
```

<img src="https://i.loli.net/2020/04/17/BCThV2uK1MzLrvl.png" alt="微信截图_20200417112508" style="zoom:200%;" />

#### 配置ftp用户

- 新增用户

```
adduser ftpuser
```

- 设置密码

```shell
# youwinok
passwd ftpuser 
```

- 添加用户到可登录用户列表文件

```shell
echo  "ftpuser"  | sudo tee –a /etc/vsftpd/user_list
```

#### 验证登录

##### 问题记录

![微信截图_20200417132758](https://i.loli.net/2020/04/17/uMyJp9baZU3nIji.png)

##### 解决方案:关闭selinux

```shell
# 编辑selinux配置文件
vi /etc/selinux/config
# SELINUX=disabled
# 重启
shutdown -r now
# 查看selinux状态
sestatus
```

##### 禁止ftp用户shell登录

```shell
# 编辑文件passwd找到对应用户/bin/bash修改为/sbin/nologin
vi /etc/passwd
# 或者
usermod -s /sbin/nologin ftpuser
```

##### 验证通过

![微信截图_20200417133907](https://i.loli.net/2020/04/17/holXGeHb4I9nwFp.png)

**收工:happy::happy:**

#### 参考

[how to install vsftpd on centos7]:https://phoenixnap.com/kb/how-to-setup-ftp-server-install-vsftpd-centos-7



