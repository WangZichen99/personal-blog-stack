---
title: Nginx笔记 
slug: NginxNote 
author: wzc 
date: 2022-07-24 
categories:
    - Study Note 
tags:
    - Nginx 
image: img/2022/07/colorful_river.jpg 
draft: false
---

# NginxNote

## 1、介绍

1. 反向代理 （1）正向代理

    - 在客户端配置代理服务器，通过代理服务器进行访问
      ![image-20220710114341138](../../img/2022/07/nginx_note/image-20220710114341138.png)

   （2）反向代理
    - 客户端无需配置，将请求发送到反向代理服务器，反向代理服务器从真实服务器获取数据后返回到客户端。此时反向代理服务器和真实服务器被看做一个服务器，暴露的是反向代理服务器，隐藏了真实服务器的ip地址

2. 负载均衡
    - 将客户端的请求分发到不同服务器上
      ![image-20220710121014443](../../img/2022/07/nginx_note/image-20220710121014443.png)

3. 动静分离
    - 使用不同的服务器解析动态页面和静态页面，加快解析速度，降低单个服务器的压力
      ![image-20220710122411951](../../img/2022/07/nginx_note/image-20220710122411951.png)

## 2、安装Nginx

1. xshell连接linux操作系统
   ![image-20220710170800526](../../img/2022/07/nginx_note/image-20220710170800526.png)
2. [下载nginx](https://nginx.org/en/download.html)
3. 使用```sudo -```命令切换到root用户
4. 安装编译工具及库文件```yum -y install make zlib zlib-devel gcc-c++ libtool  openssl openssl-devel```
5. 安装PCRE
   ```shell
   cd /usr/local/src
   wget http://downloads.sourceforge.net/project/pcre/pcre/8.45/pcre-8.45.tar.gz
   ```
6. 解压```tar -zxvf pcre-8.45.tar.gz```
7. 编译安装
   ```shell
   cd pcre-8.45
   ./configure
   make && make install
   ```
8. 查看版本```pcre-config --version```
9. 上传nginx安装包并解压```tar -zxvf nginx-1.22.0.tar.gz```
10. 编译安装
   ```shell
   cd nginx-1.22.0
   ./configure
   make && make install
   ```
11. 安装成功后，在/usr/local/nginx/sbin/下有nginx的启动脚本，启动nginx
   ```shell
   cd /usr/local/nginx/sbin/
   ./nginx
   ```
12. 查看是否成功启动```ps -ef|grep nginx```

## 3、配置Nginx开放80端口

nginx配置文件位置在/usr/local/nginx/conf/nginx.conf中

```shell
#开启防火墙
service firewalld start

#重启防火墙
service firewalld restart

#关闭防火墙
service firewalld stop

#查看开放的端口号
firewall-cmd --list-all

#设置添加http服务
firewall-cmd --add-service=http --permanent

#设置开放的端口号, --permanent表示持久的
firewall-cmd --add-port=80/tcp --permanent

#移除端口号
firewall-cmd --remove-port=80/tcp --permanent

#重启防火墙
firewall-cmd --reload
```

在浏览器中输入linux的ip地址可以看到nginx的页面
![image-20220710182916916](../../img/2022/07/nginx_note/image-20220710182916916.png)

## 4、Nginx常用命令

- 必须在/usr/local/nginx/sbin中执行命令

1. 查看nginx版本号

```shell
./nginx -v
```

2. 启动nginx

```shell
./nginx
```

3. 安全退出nginx

```shell
./nginx -s quit
```

4. 关闭nginx

```shell
./nginx -s stop
```

5. 重新加载nginx

```shell
./nginx -s reload
```

## 5、Nginx配置文件

- nginx配置文件位置：/usr/local/nginx/conf/nginx.conf

### 5.1、配置文件组成部分

#### 5.1.1、全局块

- 从配置文件开始到events块之前的内容，包含一些nginx服务器运行的配置命令，例如：用户组、worker process数、进程PID存放路径、日志路径等等

```conf
worker_process 1; 值越大，可支持的并发量越大，一般设置成CPU核心数或auto
```

#### 5.1.2、events块

- events块设置的是nginx服务器与用户的网络连接，例如：是否允许同时接收多个网络连接，每个worker_process支持的最大连接数等等，这部分配置对nginx的性能影响较大，在实际中应该灵活配置

```conf
worker_connections  1024; 每个worker process支持的最大连接数为1024
```

#### 5.1.3、http块

- 配置最频繁的部分，例如：代理、缓存和日志等等，http块包括http全局块和server块

1. http全局块：包括文件引入、MIME-TYPE定义、日志定义、连接超时时间、单链接请求上限数等等
2. server块：和虚拟主机有密切关系，每个http块可以包含多个server块，每个server块就相当于一个虚拟主机，server块可以分为全局server块和location块，一个server块可以包含多个location块
    1. 全局server块：配置虚拟主机的监听配置和虚拟主机的名称及IP配置等等
    2. location块：根据url对特定的请求做特定的处理，地址定向、数据缓存和应答控制等功能

![image-20220716201554852](../../img/2022/07/nginx_note/image-20220716201554852.png)

## 6、Nginx反向代理

- 准备工作：安装tomcat

    1. 安装jdk

        ```sh
        yum install -y java-1.8.0-openjdk
        ```

    2. 查看是否安装成功

        ```sh
        java -version
        ```

    3. 查看jdk安装位置，默认路径为/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.332.b09-1.el7_9.x86_64/jre/bin/java

        ```sh
        find / -name 'java'
        ```

    4. 创建文件夹

        ```sh
        mkdir /usr/local/tomcat
        ```

    5. 下载tomcat

        ```sh
        wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.64/bin/apache-tomcat-9.0.64.tar.gz --no-check-certificate
        ```

    6. 解压

        ```sh
        tar -zxvf apache-tomcat-9.0.64.tar.gz
        ```

    7. 添加防火墙端口

        ```sh
        firewall-cmd --add-port=8080/tcp --permanent && firewall-cmd --reload
        ```

    8. 启动tomcat

        ```sh
        cd apache-tomcat-9.0.64/bin/
        
        ./startup.sh
        ```

    9. 访问8080端口查看

       ![image-20220717113005933](../../img/2022/07/nginx_note/image-20220717113005933.png)

- 反向代理过程

  ![image-20220717115259965](../../img/2022/07/nginx_note/image-20220717115259965.png)

- nginx配置

  ![image-20220717153209628](../../img/2022/07/nginx_note/image-20220717153209628.png)

> 注意：
> - proxy_pass只是替换server_name:listen，location不会改变，所以如果proxy_pass上没有location，就会404
> - location **~** /user/ 表示使用正则匹配，并且区分大小写，如果url中包含user就响应
    >

- **~*** 表示正则匹配，并且不区分大小写

>     - **=** 表示严格匹配，如果匹配成功则立即处理该请求

## 7、Nginx负载均衡

- nginx配置

![image-20220723173116288](../../img/2022/07/nginx_note/image-20220723173116288.png)

- nginx负载均衡分配策略

    1. 轮询（默认）：每个请求按照时间顺序注意分配到不同服务器上，如果某一服务器挂掉，自动剔除改服务器
    2. weight：权重默认为1，权重越高，分配到改服务器上的请求越多
    3. ip_hash：每个请求按访问ip的hash结果分配，每个请求固定访问某一台服务器，可以解决session共享问题

  ![image-20220723174644184](../../img/2022/07/nginx_note/image-20220723174644184.png)

    4. fair（第三方）：按照后端响应时间分配，最短响应时间优先

## 8、Nginx动静分离

- 动静分离指将动态请求与静态请求分开，实现角度分为两种：
    1. 将静态文件放在单独的静态服务器上访问（主流方案）
    2.
  静态文件和动态文件放在同一个服务器上，通过location指定不同的后缀名实现静态文件的区分。通过expires参数设置浏览器缓存过期时间可以减少客户端与服务端之间的请求，过期时间由浏览器验证，到达过期时间后向服务端发送请求对比静态文件最后更新时间，如果有变化则从服务器获取最新静态文件，这种方法适合不经常变动的静态资源

- nginx配置

    ```conf
    location / {
    	root /app/static/;
    	autoindex on;
    }
    
    location /html/ {
    	root   /app/static/;
    	index  index.html index.htm;
    	autoindex on;
    }
    
    location /img/ {
    	root  /app/static/;
    	autoindex on;
    }
    ```

  ![image-20220723200624360](../../img/2022/07/nginx_note/image-20220723200624360.png)

  ![image-20220723200722738](../../img/2022/07/nginx_note/image-20220723200722738.png)

## 9、Nginx高可用

- 当反向代理服务器挂掉后会导致用户无法访问服务端，所以需要增加一台Nginx反向代理服务器来保证高可用

![image-20220724092910742](../../img/2022/07/nginx_note/image-20220724092910742.png)

- 当主服务器挂掉后keepalived会检测到服务器状态，将虚拟ip映射到备用服务器上，保证客户端的正常访问

    - 准备工作：

        1. 安装keepalived
            ```sh
                # yum安装keepalived
                yum install keepalived -y

                # 查看版本
                keepalived -v
            ```

        2. keepalived配置文件位置：/etc/keepalived/keepalived.conf

            ```conf
            ! Configuration File for keepalived
      
            global_defs {
               #notification_email { # 通知邮件，一般不用
               #  acassen@firewall.loc
               #  failover@firewall.loc
               #  sysadmin@firewall.loc
               #}
               #notification_email_from Alexandre.Cassen@firewall.loc
               smtp_server 192.168.200.1
               smtp_connect_timeout 30
               router_id LVS_DEVEL # 标识本节点字符串
               vrrp_skip_check_adv_addr
               vrrp_strict chk_http_port { # 检查主服务器状态执行脚本配置
                   script "/usr/local/src/nginx_check.sh" # 脚本位置
                   interval 2 # 执行脚本时间间隔
                   weight 2
               }
               vrrp_garp_interval 0
               vrrp_gna_interval 0
            }
      
            vrrp_instance VI_1 {
                state MASTER # 主服务器为MASTER，备服务器为BACKUP
                interface ens33 # 网卡
                virtual_router_id 51 # 主备服务器需要相同
                priority 100 # 优先级，主服务器大于备服务器
                advert_int 1 # MASTER和BACKUP节点之间的同步检查时间间隔，单位为秒
                authentication { # 验证类型和验证密码
                    auth_type PASS
                    auth_pass 1111
                }
                virtual_ipaddress { # 虚拟IP地址池，可以多个IP
                    192.168.200.16
                    192.168.200.17
                    192.168.200.18
                }
            }
            ```

        3. nginx_check.sh

            ```sh
            #!/bin/bash
      
            if [ `ps -C nginx --no-header |wc -l` -eq 0];then
                /usr/local/nginx/sbin/nginx
                sleep 2
                if [ `ps -C nginx --no-header |wc -l` -eq 0];then
                    killall keepalived # systemctl stop keepalived
                fi
            fi
            ```

        4. 启动keepalived

            ```sh
            systemctl start keepalived.service
            ```

- 通过ip a可以查看到虚拟ip

  ![image-20220724155145881](../../img/2022/07/nginx_note/image-20220724155145881.png)

## 10、Nginx原理

![image-20220724163125071](../../img/2022/07/nginx_note/image-20220724163125071.png)

- nginx分为master进程和worker进程，其中worker进程可能有多个，当客户端发送请求后，master收到客户端的请求，所有的worker进程会争抢客户端的请求，然后进行发送到服务端处理
- 多个worker进程的好处：
    1. 在reload时，没有争抢请求的worker进程可以进行reload，有利于热部署
    2. 每个worker是独立的进程，如果某一个worker进程挂掉，其他worker还可以正常工作，worker进程的数量通常和服务器cpu数相等

> 1. worker发送请求占用了几个连接数？
     >
     >     2个（静态请求）或4个（动态请求）
>
> 2. 如果Nginx有一个master，四个worker，每个worker最大连接数为1024，那么最大并发数是多少？
     >
     >     如果是静态请求，并发数为4（worker的数量） * 1024（worker最大连接数） / 2
     >
     >     如果是动态请求，并发数为4 * 1024 / 4
