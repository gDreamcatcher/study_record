**Mysql学习总结**

> 简介：MySQL 是最流行的关系型数据库管理系统，在 WEB 应用方面 MySQL 是最好的 RDBMS(Relational Database Management System：关系数据库管理系统)应用软件之一。由于其体积小、速度快、总体拥有成本低，尤其是开放源码这一特点，许多中小型网站为了降低网站总体拥有成本而选择了MySQL作为网站数据库。目前常用的mysql版本有5.6.51，5.7.33和8.0.23.本文主要针对mysql 8以后的版本。

- [1. 下载并安装](#1-下载并安装)
  - [1.1. docker-最简单的方式](#11-docker-最简单的方式)
  - [1.2. 可执行文件](#12-可执行文件)
  - [1.3. dockerfile](#13-dockerfile)


# 1. 下载并安装
## 1.1. docker-最简单的方式
```bash
docker pull mysql:8.0.23
# Starting a MySQL instance is simple:
docker run --name contained-name -e MYSQL_ROOT_PASSWORD=123456 -d mysql:8.0.23
# Connect to MySQL from the MySQL command line client
docker run -it --rm mysql mysql -hcontained-name-ip -uroot -p123456
```

## 1.2. 可执行文件
```bash
# install.sh
#!/bin/bash
# 安装依赖
yum update -y && yum install -y wget numactl-devel.x86_64 libaio-devel.x86_64
# 下载安装包并解压
wget https://cdn.mysql.com/archives/mysql-8.0/mysql-8.0.22-linux-glibc2.12-x86_64.tar
tar xf mysql-8.0.22-linux-glibc2.12-x86_64.tar
rm -rf mysql-router-8.0.22-linux-glibc2.12-x86_64.tar.xz mysql-test-8.0.22-linux-glibc2.12-x86_64.tar.xz mysql-8.0.22-linux-glibc2.12-x86_64.tar
xz -d  mysql-8.0.22-linux-glibc2.12-x86_64.tar.xz  && tar -xf  mysql-8.0.22-linux-glibc2.12-x86_64.tar && mv  mysql-8.0.22-linux-glibc2.12-x86_64 mysql && cd mysql

# 创建mysql用户组和用户，并赋给/usr/local/mysql文件夹权限
groupadd  mysql && useradd -g mysql mysql && chown -R mysql:mysql /usr/local/mysql && chmod -R 755 /usr/local/mysql 

# 修改/etc/my.cnf
cat > /etc/my.cnf <<EOF
[mysqld]
# mysql8使用的验证密码为caching_sha2_password, 需要修改为mysql_native_password
default_authentication_plugin=mysql_native_password
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
datadir = /export/data/mysql/data
socket  = /export/data/mysql/data/mysqld.sock
log-error = /export/data/mysql/data/error.log
pid-file = /export/data/mysql/data/mysqld.pid
EOF
# mysql8安装的时候会默认生产一个临时密码，
# 为了初始化时制定密码需要加上 --init-file=/usr/local/mysql/mysql-init
# 文件内容如下：
cat > /usr/local/mysql/mysql-init <<EOF
ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';
use mysql; 
update user set host='%' where user ='root';
FLUSH PRIVILEGES;
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%'WITH GRANT OPTION;
EOF
# 创建mysql的data目录，并指定用户组和用户名为mysql，并添加读写权限
mkdir -p /export/data/mysql/data;chown -R mysql:mysql /export/data/mysql/data;chmod -R 755 /export/data/mysql/data

# 初始化mysql，如果启动失败就删除/export/data/mysql/data中的所有文件
/usr/local/mysql/bin/mysqld --initialize --init-file=/opt/mysql-init --user=mysql --datadir=/export/data/mysql/data --basedir=/usr/local/mysql

# 启动mysql，启动时会读取my.cnf中的配置，所以my.cnf中的datadir路径要和初始化中设置的一致
# ln -s /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql 
/usr/local/mysql/support-files/mysql.server restart

# 启动mysql
ln -s /usr/local/mysql/bin/mysql /usr/bin/mysql 
mysql -uroot -p123456
```

## 1.3. dockerfile 

```dockerfile
FROM centos:centos7
ENV LANG zh_CN.UTF-8
RUN yum update -y && yum install -y wget numactl-devel.x86_64 libaio-devel.x86_64 pam-devel.x86_64 wget make gcc-c++

# 下载mysql
RUN wget https://cdn.mysql.com//Downloads/MySQL-8.0/mysql-8.0.22-linux-glibc2.12-x86_64.tar -O /tmp/mysql-8.0.22-linux-glibc2.12-x86_64.tar && \
	cd /usr/local && tar xf /tmp/mysql-8.0.22-linux-glibc2.12-x86_64.tar && mv  mysql-8.0.22-linux-glibc2.12-x86_64 mysql 

# 下载并编译nginx
RUN wget https://nginx.org/download/nginx-1.18.0.tar.gz -O /tmp/nginx-1.18.0.tar.gz && \
    cd /tmp && tar xf /tmp/nginx-1.18.0.tar.gz && cd nginx-1.18.0 && \
    ./configure --prefix=/usr/local/nginx  --with-http_ssl_module --with-stream && \
    make -j && make install && \
    rm -rf /tmp/*
   
RUN groupadd  mysql && useradd -g mysql mysql && chown -R mysql:mysql /usr/local/mysql && chmod -R 755 /usr/local/mysql 
COPY . /opt/
ENTRYPOINT /usr/sbin/sshd && /usr/sbin/crond && bash /opt/start.sh && sleep 9999999d

```

```bash
# install.sh
#!/bin/bash
echo "##################################################      mysql    ###################################################################"
# 创建mysql用户组和用户，并赋给/usr/local/mysql文件夹权限
groupadd  mysql && useradd -g mysql mysql && chown -R mysql:mysql /usr/local/mysql && chmod -R 755 /usr/local/mysql 

# 修改/etc/my.cnf
cat > /etc/my.cnf <<EOF
[mysqld]
# mysql8使用的验证密码为caching_sha2_password, 需要修改为mysql_native_password
default_authentication_plugin=mysql_native_password
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
datadir = /export/data/mysql/data
socket  = /export/data/mysql/data/mysqld.sock
log-error = /export/data/mysql/data/error.log
pid-file = /export/data/mysql/data/mysqld.pid
EOF
# mysql8安装的时候会默认生产一个临时密码，
# 为了初始化时制定密码需要加上 --init-file=/usr/local/mysql/mysql-init
# 文件内容如下：
cat > /usr/local/mysql/mysql-init <<EOF
ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';
use mysql; 
update user set host='%' where user ='root';
FLUSH PRIVILEGES;
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%'WITH GRANT OPTION;
EOF
# 创建mysql的data目录，并指定用户组和用户名为mysql，并添加读写权限
mkdir -p /export/data/mysql/data;chown -R mysql:mysql /export/data/mysql/data;chmod -R 755 /export/data/mysql/data

# 初始化mysql，如果启动失败就删除/export/data/mysql/data中的所有文件
/usr/local/mysql/bin/mysqld --initialize --init-file=/opt/mysql-init --user=mysql --datadir=/export/data/mysql/data --basedir=/usr/local/mysql

# 启动mysql，启动时会读取my.cnf中的配置，所以my.cnf中的datadir路径要和初始化中设置的一致
# ln -s /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql 
/usr/local/mysql/support-files/mysql.server restart

echo "##################################################      nginx    ###################################################################"
# 配置nginx.conf
cat > /usr/local/nginx/nginx.conf <<EOF
worker_processes  1;
events {
    worker_connections  1024;
}
stream {
    upstream mysql_3306 {
        server localhost:3306 weight=10 max_fails=2 fail_timeout=30s;
    }
    server {
        listen 80;
        proxy_connect_timeout 20s;
        proxy_pass mysql_3306;
    }
}
EOF
# 启动nginx
/usr/local/nginx/sbin/nginx
```


