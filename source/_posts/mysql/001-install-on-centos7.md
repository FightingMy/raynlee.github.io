---
title: mysql8 install on centos7
date: 2013/7/13 20:46:25
tags: mysql,centos7
---
# 安装

### 1.下载安装包

```
wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.25-1.el7.x86_64.rpm-bundle.tar
https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-8.0.30-1.el7.x86_64.rpm-bundle.tar
https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-8.0.30-1.el7.x86_64.rpm-bundle.tar

```

### 2.解开压缩包

    # 查看包里面的内容
    rpm -qa |grep -i mysql-8.0.30-1.el7.x86_64.rpm-bundle.tar
    # 解压
    tar -xvf mysql-5.7.25-1.el7.x86_64.rpm-bundle.tar -C 指定文件夹

### 3.卸载mariadb

    # 查看系统自带的Mariadb
    rpm -qa|grep mariadb
    # 卸载系统自带的Mariadb
    rpm -e mariadb-libs-5.5.44-2.el7.centos.x86_64 --nodeps
    # 删除etc目录下的my.cnf
    rm /etc/my.cnf

### 4.安装mysql

#### 4.1 安装前准备

    # 创建mysql用户组
    groupadd mysql
    # 创建一个用户名为mysql的用户，并加入mysql用户组
    useradd -g mysql mysql
    # 修改用户密码
    passwd mysql

#### 4.2 安装perl

```
# 安装编译环境
yum -y install gcc cpan

0）下载源码包和解压
wget http://www.cpan.org/src/5.0/perl-5.16.1.tar.gz
tar -zvf perl-5.16.1.tar.gz
1）配置perl安装目录
./Configure -des -Dprefix=/usr/bin/perl
2) 编译perl
make
3) 安装perl
make install

make && make install

# 查看版本
perl -v



mysql5:
rpm -ivh mysql-community-common-5.7.25-1.el7.x86_64.rpm
rpm -ivh mysql-community-libs-5.7.25-1.el7.x86_64.rpm
rpm -ivh mysql-community-client-5.7.25-1.el7.x86_64.rpm
rpm -ivh mysql-community-server-5.7.25-1.el7.x86_64.rpm

mysql8:  
rpm -ivh mysql-community-common-8.0.32-1.el7.x86_64.rpm
rpm -ivh mysql-community-libs-8.0.32-1.el7.x86_64.rpm
rpm -ivh mysql-community-client-plugins-8.0.32-1.el7.x86_64.rpm
rpm -ivh mysql-community-libs-8.0.32-1.el7.x86_64.rpm
rpm -ivh mysql-community-client-8.0.32-1.el7.x86_64.rpm
rpm -ivh mysql-community-icu-data-files-8.0.32-1.el7.x86_64.rpm
rpm -ivh mysql-community-server-8.0.32-1.el7.x86_64.rpm

```

> 在安装rpm -ivh mysql-community-server-5.7.16-1.el7.x86\_64.rpm的时候报错如下：

      [root@linux_node_1 src]# rpm -ivh mysql-community-server-5.7.25-1.el7.x86_64.rpm
      warning: mysql-community-server-5.7.25-1.el7.x86_64.rpm: Header V3 DSA/SHA1 Signature, key ID 5072e1f5: NOKEY
      error: Failed dependencies:
      libaio.so.1()(64bit) is needed by mysql-community-server-5.7.25-1.el7.x86_64
      libaio.so.1(LIBAIO_0.1)(64bit) is needed by mysql-community-server-5.7.25-1.el7.x86_64
      libaio.so.1(LIBAIO_0.4)(64bit) is needed by mysql-community-server-5.7.25-1.el7.x86_64
      net-tools is needed by mysql-community-server-5.7.25-1.el7.x86_64

这个报错的意思是需要安装libaio包和net-tools包：可以yum安装一下

    yum install numactl net-tools -y

安装完成之后再执行安装

    rpm -ivh mysql-community-server-5.7.25-1.el7.x86_64.rpm

### 6.修改数据库默认字符集

    1.打开配置文件
    vi /etc/my.cnf

    2.修改配置文件

    [client]
    default-character-set=utf8mb4

    [mysqld]
    default-storage-engine=INNODB
    character-set-server=utf8mb4
    collation-server=utf8mb4_general_ci

    3.重新启动数据库服务

    systemctl restart mysqld

    4. 设置为开机启动
    systemctl enable mysqld

### mysql8 初始化

```
mysqld --initialize
chown mysql:mysql /var/lib/mysql -R
systemctl start mysqld.service
systemctl enable mysqld

// 2.查看数据库初始化生成的随机密码
cat /var/log/mysqld.log | grep password

```

### 5.配置mysql

```
// 1.启动MySQL
service mysqld start
// 2.查看数据库初始化生成的随机密码
grep "password" /var/log/mysqld.log


2019-02-15T08:25:38.632902Z 1 [Note] A temporary password is generated for root@localhost: 6jTyZfB(qaIL
2019-02-15T08:27:11.388665Z 2 [Note] Access denied for user 'root'@'localhost' (using password: NO)
// 3.设置密码和添加用户
mysql -uroot -p
// 修改root用户密码
alter user 'root'@'localhost' identified by '123456';
alter user 'root'@'localhost' identified by '123456';
// 创建新用户
create user 'ljy'@'%' identified by '123456'; 
// 给用户赋值权限
grant all privileges on *.* to 'ljy'@'%' with grant option;
flush privileges;

```

### 重置密码

    # 1.停止mysql服务
    systemctl stop mysqld
    # 2.启动跳过密码验证的临时服务
    mysqld --console  --skip-grant-tables --shared-memory

    # 3.使用一个新窗口免密登录mysql
    mysql -u root -P3308 -hlocalhost
    # 4. 将密码置空
    use mysql
    update user set authentication_string='' where user='root';
    exit

    # 5.关闭临时服务，打开正常服务
    mysql -uroot -p
    alter user 'root'@'localhost' identified by '123456';

# Docker 安装

    docker run -d --name mysql5 -p 3306:3306 -v /d/dev/dockerdata/mysql5:/var/lib/mysql -env MYSQL_ROOT_PASSWORD=123456 mysql:5.7 --restart=always

# yum 安装

```shell
# 浏览器打开查找版本
 https://repo.mysql.com
# 添加rpm源
rpm -ivh https://repo.mysql.com//mysql57-community-release-el7-11.noarch.rpm
# 通过yum搜索
yum search mysql-community
# 安装x64位的 mysql server
yum install -y mysql-community-server.x86_64
```

# 问题

## Authentication to host '192.168.1.11' for user 'ljy' using method 'mysql\_native\_password' failed with message: Access denied for user 'ljy'@'192.168.1.11' (using password: YES)

    1. 添加配置
    # /etc/my.cof
    [mysqld]
    default-authentication-plugin=mysql_native_password

    2.修改密码
    use mysql;
    ALTER USER 'ljy'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
    FLUSH PRIVILEGES;

    3. 重启服务
    systemctl restart mysqld

## 查询配置文件位置

    1. 查询自定义配置文件位置
    ps aux|grep mysql|grep 'my.cnf'

    2. 如果1 没有输出，则查询默认文件位置
    # 读取位置依次排列
    mysql --help|grep 'my.cnf'

    # 查看默认查找路径
    mysqld --verbose --help |grep -A 1 'Default options'

