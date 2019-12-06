---
title: Centos7.4安装nginx-tomcat8-mysql
date: 2019-12-03 10:38:11
categories: 
- Linux
tags:
- 系统服务
- CentOs7.4
---

### 修改hostname

1.  修改/etc/hostname文件内容：$ sudo vim /etc/hostname 将里面的内容修改为新的主机名称
2.  修改/etc/hosts文件：$ sudo vim /etc/hosts 将127.0.0.1这一行的内容修改为新的主机名称
3.  重启

### 安装 nginx-1.17.0.tar.gz

1. nginx 的 http 模块使用 pcre 来解析正则表达式，所以需要在 linux 上安装 pcre 库，pcre-devel 是使用 pcre 开发的一个二次开发库
    ```
    yum install -y pcre pcre-devel
    ```

2. zlib 库提供了很多种压缩和解压缩的方式， nginx 使用 zlib 对 http 包的内容进行 gzip ，所以需要在 Centos 上安装 zlib 库
    ```
    yum install -y zlib zlib-devel
    ```

3.  如果有需要，根据error提示可能还需要GCC和OpenSSL
    ```
    yum install gcc-c++
    yum install -y openssl openssl-devel
    ```

4.  开始安装nginx,1.17.0这个是目前最新版本
    ```
    wget -c https://nginx.org/download/nginx-1.17.0.tar.gz
    ```

5.  解压并进入nginx目录
    ```
    tar -zxvf nginx-1.17.0.tar.gz
    cd nginx-1.17.0
    ```
  
6.  使用nginx的默认配置
    ```
    ./configure
    ```

7.  查找安装路径
    ```
    whereis nginx
    ```

8.  安装好后，把nginx添加到系统服务中
    ```
    # 添加nginx.service
    # vim /lib/systemd/system/nginx.service
    [Unit]
    Description=nginx service
    After=network.target 
    [Service] 
    Type=forking
    # 路径对应安装路径
    ExecStart=/usr/local/nginx/sbin/nginx
    ExecReload=/usr/local/nginx/sbin/nginx -s reload
    ExecStop=/usr/local/nginx/sbin/nginx -s quit
    PrivateTmp=true 
    [Install] 
    WantedBy=multi-user.target
    ```

9.  添加完成之后即可
    ```
    systemctl enable|status/|start|restart|stop nginx.service
    ```

### firewalld 防火墙配置

1.  不改变状态下重载防火墙
    ```
    firewall-cmd --reload
    ```

2.  基本使用
    ```
    启动： systemctl start firewalld
    关闭： systemctl stop firewalld
    查看状态： systemctl status firewalld 
    开机禁用  ： systemctl disable firewalld
    开机启用  ： systemctl enable firewalld
    ```

3.  systemctl是CentOS7的服务管理工具中主要的工具，它融合之前service和chkconfig的功能于一体
    ```
    启动一个服务：systemctl start firewalld.service
    关闭一个服务：systemctl stop firewalld.service
    重启一个服务：systemctl restart firewalld.service
    显示一个服务的状态：systemctl status firewalld.service
    在开机时启用一个服务：systemctl enable firewalld.service
    在开机时禁用一个服务：systemctl disable firewalld.service
    查看服务是否开机启动：systemctl is-enabled firewalld.service
    查看已启动的服务列表：systemctl list-unit-files|grep enabled
    查看启动失败的服务列表：systemctl --failed
    ```

4.  配置firewalld-cmd
    ```
    查看版本： firewall-cmd --version
    查看帮助： firewall-cmd --help
    显示状态： firewall-cmd --state
    查看所有打开的端口： firewall-cmd --zone=public --list-ports
    更新防火墙规则： firewall-cmd --reload
    查看区域信息:  firewall-cmd --get-active-zones
    查看指定接口所属区域： firewall-cmd --get-zone-of-interface=eth0
    拒绝所有包：firewall-cmd --panic-on
    取消拒绝状态： firewall-cmd --panic-off
    查看是否拒绝： firewall-cmd --query-panic
    ```

5.  开启一个端口
    ```
    添加
    firewall-cmd --zone=public --add-port=80/tcp --permanent    （--permanent永久生效，没有此参数重启后失效）
    重新载入
    firewall-cmd --reload
    查看
    firewall-cmd --zone=public --query-port=80/tcp
    删除
    firewall-cmd --zone=public --remove-port=80/tcp --permanent
    ```

### CentOs7.4安装Mysql8.0.17,[参考文献](https://www.cnblogs.com/yangk1996/p/10700537.html)

1.  [Mysql下载传送门](https://dev.mysql.com/downloads/mysql/)
2.  卸载Linux自带mariadb[什么是mariadb?](http://www.sohu.com/a/252174706_355140)
    ````
    查看是否存在
    rpm -qa | grep mariadb
    卸载
    rpm -e mariadb-libs-5.5.56-2.el7.x86_64 --nodeps
    在查看是否存在
    rpm -qa | grep mariadb
    ````
3.  通过FTP将下载好的mysql-8.0.18-1.el7.x86_64.rpm-bundle.tar传输到/khgo/volume/service
4.  解压MySQL
    ````
    tar -xvf mysql-8.0.17-1.el7.x86_64.rpm-bundle.tar
    ````
5.  依此安装所需RPM
    ````
    rpm -ivh mysql-community-common-8.0.17-1.el7.x86_64.rpm  --nodeps --force
    rpm -ivh mysql-community-libs-8.0.17-1.el7.x86_64.rpm  --nodeps --force
    rpm -ivh mysql-community-client-8.0.17-1.el7.x86_64.rpm --nodeps --force
    rpm -ivh mysql-community-server-8.0.17-1.el7.x86_64.rpm --nodeps --force
    查看依赖
    rpm -qa | grep mysql
    ````
    
6.  依此执行执行初始化命令
    ````
    $ mysqld --initialize
    报错：mysqld: error while loading shared libraries: libaio.so.1: cannot open shared object file: No such file or directory
    解决：yum install -y libaio
    $ chown mysql:mysql /var/lib/mysql -R
    $ systemctl start mysqld.service
    报错：Job for mysqld.service failed because the control process exited with error code. See "systemctl status mysqld.service" and "journalctl -xe" for details.
    解决：rm -rf /var/lib/mysql
    $ systemctl  enable mysqld
    查看初始Mysql密码
    $ cat /var/log/mysqld.log | grep password
    结果：2019-12-03T05:54:50.131567Z 5 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: 0wVr<#prLEow
    使用命令登录MySQL
    mysql -uroot -p
    password: 0wVr<#prLEow
    ````
    
7.  修改MySQL密码，并远程授权
    
    ````
    ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'khgo2019';
    失败：ERROR 1819 (HY000): Your password does not satisfy the current policy requirements
    原因：密码强度太低
    为了安全我们就不修改mysql默认安全策略了(大小写，特殊字符都加上)
    ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'khGo_2019';
    成功！
    远程访问Mysql,通过下面命令授权
    create user 'root'@'%' identified with mysql_native_password by 'mypassword';
    grant all privileges on *.* to 'root'@'%' with grant option;
    flush privileges;
    exit;
    最后使用navicat看是否能连接数据库！
    ````
    
    
    



