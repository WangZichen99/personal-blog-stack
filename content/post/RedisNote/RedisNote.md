---
title: Redis笔记
slug: RedisNote
author: wzc
date: 2022-08-14
categories: 
    - 学习笔记
tags: 
    - Redis
    - NoSQL
image: img/2022/11/autumn_forest.jpg
draft: false
---

# Redis 笔记

## 概述

### Redis能干嘛

1. 内存存储、持久化
2. 高速缓存
3. 发布订阅系统
4. 地图信息分析
5. 计时器、计数器（浏览量）
6. ...

## Windows安装Redis

1. 下载地址：https://github.com/tporadowski/redis/releases
2. 下载解压后打开redis-server.exe和redis-cli.exe

​	![image1](../../img/2022/11/redis_note/image1.png)

​	![image2](../../img/2022/11/redis_note/image2.png)

3. 输入ping测试连接，set设置key value，get获取

![image3](../../img/2022/11/redis_note/image3.png)

## Linux安装Redis

1. 下载安装包https://redis.io/download/
2. `tar -zxvf redis-7.0.4.tar.gz` 解压
3. `yum install gcc-c++`安装环境
4. 进入redis解压目录`make && make install`
5. redis默认安装目录：/usr/local/bin

6. redis默认不是后台启动，修改配置daemonize为yes

    ![image4](../../img/2022/11/redis_note/image4.png)

	7. 启动redis服务：进入/usr/local/bin/，执行`redis-server ../src/redis-7.0.4/redis.conf`
	7. 客户端连接redis：`redis-cli -p 6379`
	7. 查看redis进程：`ps -ef|grep redis`
	7. 关闭redis服务：shutdown，exit

​	![image5](../../img/2022/11/redis_note/image5.png)

## 性能测试

redis-benchmark是一个压力测试工具

- 测试100个并发连接，100000个请求
    - `redis-benchmark -h localhost -p 6379 -c 100 -n 100000`

## 基础知识

1. 默认16个数据库，默认使用第0个

![image6](../../img/2022/11/redis_note/image6.png)

```bash
select <dbid> #切换数据库

dbsize #查看数据库大小

keys * #查看数据库中所有key

flushdb #清空当前数据库

flushall #清空所有数据库
```

2. Redis是单线程的
    - 官方表示Redis是基于内存操作，CPU不是Redis的性能瓶颈，Redis的瓶颈是根据机器的内存和网路带宽，既然可以用单线程实现，就使用了单线程
    - **单线程为什么还这么快？**
        - 误区1：高性能的服务器一定是多线程的
        - 误区2：多线程一定比单线程效率高
        - 对于Redis来说，所有数据全部放在内存中，对于内存系统来说不需要上下文切换，所以单线程比多线程效率更高

## 五大数据类型

### Redis-Key

```sh
exists {key} #当前数据库是否存在某个key

move {key} {db} #将key移动到数据库db中

expire {key} {seconds} #设置key的过期时间为seconds秒

ttl {key} #查看key的剩余过期时间

type {key} #查看key的类型
```

### String (字符串)

```sh
set {key} {value}

get {key} {value}

append {key} {value} #在key值中追加value，如果key不存在就相当于set key

strlen {key} #查看key的长度

incr {key} #key值自增1

decr {key} #key值自减1

incrby {key} {increment} #key值自增increment

decrby {key} {increment} #key值自减increment

getrange {key} {start} {end} #截取key值的start到end

setrange {key} {start} {value} #start开始替换key值为value

setex {key} {seconds} {value} #设置key值为value，seconds秒后过期，setex (set with expire)

setnx {key} {value} #如果key不存在，设置key值为value，setnx (set if not expire)

mset {key1} {value1} {key2} {value2} #批量设置

mget {key1} {key2} #批量获取

msetnx {key1} {value1} {key2} {value2} #如果所有key都不存在，批量设置

mset {object}:{id}:{field} #key命名方式

getset {key} {value} #先获取key值，再设置key
```

### List (列表)

```sh
lpush {key} {value} #左插入value

rpush {key} {value} #右插入value

lrange {key} {start} {end} #获取从start到end的值

lpop {key} #移除list左侧的值

rpop {key} #移除list右侧的值

lindex {key} {index} #获取下标为index的值

llen {key} #获取list长度

lrem {key} {count} {value} #移除key中count个值为value的元素

ltrim {key} {start} {end} #从start到end截取list

rpoplpush {source} {destination} #移除source中的右侧值插入destination中左侧

lset {key} {index} {item} #将list中下标为index的元素替换为item

linsert {key} [before|after] {pivot} {element} #在第一个值为pivot的元素前或后面插入element
```

### Set (集合)

- 集合中的元素不能重复

```sh
sadd {key} {members...} #向集合中添加元素

smembers {key} #查看集合中的元素

sismember {key} {member} #查看元素是否存在集合中

scard {key} #查看集合中的元素个数

srem {key} {member} #移除集合中的元素

srandmember {key} (count) #随机输出集合中的count个元素

spop {key} (count) #随机弹出集合中的count个元素

smove {source} {destination} {member} #将一个集合中的元素移动到另一个集合

sdiff {key} {key...} #查看多个集合与第一个集合的差集

sinter {key} {key...} #查看多个集合的交集

sunion {key} {key...} #查看多个集合的并集
```

### Hash (哈希)

```sh
hset {key} {field} {value} #设置键和值

hget {key} {field} #获取键的值

hmset {key} {field value...} #设置hash的多个键和值

hmget {key} {field...} #获取hash的多个键的值

hgetall {key} #获取hash的所有键和值

hdel {key} {field...} #删除键

hlen {key} #获取hash长度

hexists {key} {field} #判断hash的键是否存在

hkeys {key} #获取hash的所有键

hvals {key} #获取hash的所有值

hincrby {key} {field} {increment} #hash的键自增increment

hsetnx {key} {field} {value} #如果hash中没有这个键就创建
```

### Zset (有序集合)

- 一般用来做排行榜

```sh
zadd {key} {score} {value} #添加value

zrange {key} {start} {end} #获取zset中的值

zrangebyscore {key} {min} {max} [withscores] #升序排列zset中score在min到max之间的元素，withscores表示打印份数

zrevrange {key} {start} {end} [withscores] #根据score降序排列zset中下标为start到end的元素

zrem {key} {member...} #移除zset中的某个元素

zcard {key} #获取集合中的个数

zcount {key} {min} {max} #获取分数在min到max之间的元素个数
```

## 三种特殊数据类型

### Geospatial (地理位置)

- 一般用来进行附近好友的展示

```sh
geoadd {key} {longitude} {latitude} {member} #添加地理位置

geopos {key} {member...} #获取指定成员的位置信息

geodist {key} {member1} {member2} [m|km|mi|ft] #获取两个成员之间的绝对距离，单位：m（米）、km（千米）、mi（英里）、ft（英尺）

georadius {key} {longitude} {latitude} {radius} [m|km|mi|ft] [withcoord|withdist] [count num] #获取圆形区域内的所有成员 withcoord返回成员经纬度，withdist返回距离圆心的直线距离

georadiusbymember {key} {member} {radius}  [m|km|mi|ft] [withcoord|withdist] [count num] #获取圆形区域内的所有成员

geohash {key} {member...} #返回成员的hash值
```

### Hyperloglog (基数统计)

- 一般用来进行用户访问统计。如果容错，可以使用hyperloglog，如果不容错，则使用set集合来存储用户id进行统计

```sh
pfadd {key} {element...} #添加元素

pfcount {key} #返回去重后的元素数量

pfmerge {distkey} {sourcekey...} #合并所有sourcekey并去重得到distkey
```

### Bitmaps (位图)

- 统计用户信息（是否活跃、是否登录），365天打卡信息等

```sh
setbit {key} {offset} {value} #设置第offset位的值value

getbit {key} {offset} #获取第offset位的值

bitcount {key} [start] [end] #从start到end计数
```

##  Redis事务

- **redis单条命令是原子性的，但是事务是不保证原子性的**
- **redis事务没有隔离级别的概念**

```
------ 队列  命令1  命令2  命令3  执行  ------
```

- 所有命令在事务中没有直接被执行，只有发起执行命令的时候才会被执行 

```sh
multi #开启事务

... #命令入队

exec #执行事务

discard #放弃事务
```

>编译型异常（代码有问题，命令有错），事务中所有命令都不会被执行
>
>运行时异常（1 / 0），错误命令抛出异常，其他命令正常执行

## Redis实现悲观锁乐观锁

**悲观锁：**

- 很悲观，认为什么时候都会出问题，做什么事情都要加锁

**乐观锁：**

- 很乐观，认为什么时候都不会出问题，所以不会上锁，更新数据的时候判断一下，是否有人修改过这个数据

1. 获取version
2. 更新的时候比较version

```sh
watch {key} #加乐观锁监视key，如果监视key的值和事务执行前key的值不一致就报错，无法执行事务

unwatch #解除监视 (解锁)，事务执行后会自动解锁
```

## Jedis

1. 导入依赖：

    ```xml
    <dependency>
        <groupId>redis.clients</groupId>
        <artifactId>jedis</artifactId>
        <version>4.3.0-m2</version>
    </dependency>
    ```

2. 测试

    ```java
    public class JedisTest {
    
        @Test
        public void test() {
            Jedis jedis = new Jedis("127.0.0.1", 6379);
            jedis.flushDB();
            // String相关命令测试
            stringTest(jedis);
    
            // 事务测试
            multiTest(jedis);
        }
    
        private void multiTest(Jedis jedis) {
            jedis.set("hello", "init");
            jedis.watch("hello"); //监视hello，如果值发生变化，事务不执行
            jedis.set("hello", "before");
            Transaction transaction = jedis.multi();
            try {
                transaction.set("multi", "test");
                // int i = 1 / 0; //运行时异常，错误命令抛出异常，其他命令正常执行
                transaction.exec();
            } catch (Exception e) {
                e.printStackTrace();
                transaction.discard();
            } finally {
                System.out.println(jedis.get("multi"));
                jedis.close();
            }
        }
    
        private void stringTest(Jedis jedis) {
            System.out.println(jedis.ping());
            System.out.println(jedis.set("name", "wzc"));
            System.out.println(jedis.get("name"));
            System.out.println(jedis.exists("name"));
            System.out.println(jedis.keys("*"));
            System.out.println(jedis.append("age", "22"));
            System.out.println(jedis.incrBy("age", 2));
            System.out.println(jedis.strlen("name"));
            System.out.println(jedis.getrange("name", 0, -1));
            System.out.println(jedis.setrange("name", 0, "robin"));
            System.out.println(jedis.get("name"));
            System.out.println(jedis.mset("play", "game", "climb", "mountain"));
            System.out.println(jedis.mget("play", "climb"));
        }
    }
    ```

    

## SpringBoot整合Redis

> 说明：在Spring2.X之后，原来使用的Jedis被替换为了Lettuce
>
> Jedis采用直连，多线程操作不安全，解决方法是使用Jedis Pool连接池，BIO模式
>
> Lettuce采用Netty，实例可以在多个线程中共享，不存在线程不安全的情况，NIO模式

源码分析：

```java
public class RedisAutoConfiguration {
    public RedisAutoConfiguration() {
    }

    @Bean
    @ConditionalOnMissingBean(name = {"redisTemplate"})
    @ConditionalOnSingleCandidate(RedisConnectionFactory.class)
    //默认的RedisTemplate没有过多的设置，由于泛型不是<String, Object>，可以自己定义redisTemplate替换默认的
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<Object, Object> template = new RedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }

    @Bean
    @ConditionalOnMissingBean
    @ConditionalOnSingleCandidate(RedisConnectionFactory.class)
    //由于String是Redis中最常使用的类型，所以单独提出来了一个template
    public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory) {
        StringRedisTemplate template = new StringRedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }
}
```

1. 导入依赖

    ```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
    ```

2. 配置连接

    ```yaml
     spring:
     	redis:
            host: 127.0.0.1
            port: 6379
    ```

    

## Redis.conf详解

配置文件位置：/usr/local/src/redis-7.0.4/redis.conf

1. 配置文件中units对大小写不敏感

![image7](../../img/2022/11/redis_note/image7.png)

2. redis配置文件可以引用一些外部配置

![image8](../../img/2022/11/redis_note/image8.png)

3. 绑定ip和端口

    ```bash
    ################################## NETWORK #####################################
    
    # By default, if no "bind" configuration directive is specified, Redis listens
    # for connections from all available network interfaces on the host machine.
    # It is possible to listen to just one or multiple selected interfaces using
    # the "bind" configuration directive, followed by one or more IP addresses.
    # Each address can be prefixed by "-", which means that redis will not fail to
    # start if the address is not available. Being not available only refers to
    # addresses that does not correspond to any network interface. Addresses that
    # are already in use will always fail, and unsupported protocols will always BE
    # silently skipped.
    #
    # Examples:
    #
    # bind 192.168.1.100 10.0.0.1     # listens on two specific IPv4 addresses
    # bind 127.0.0.1 ::1              # listens on loopback IPv4 and IPv6
    # bind * -::*                     # like the default, all available interfaces
    #
    # ~~~ WARNING ~~~ If the computer running Redis is directly exposed to the
    # internet, binding to all the interfaces is dangerous and will expose the
    # instance to everybody on the internet. So by default we uncomment the
    # following bind directive, that will force Redis to listen only on the
    # IPv4 and IPv6 (if available) loopback interface addresses (this means Redis
    # will only be able to accept client connections from the same host that it is
    # running on).
    #
    # IF YOU ARE SURE YOU WANT YOUR INSTANCE TO LISTEN TO ALL THE INTERFACES
    # COMMENT OUT THE FOLLOWING LINE.
    #
    # You will also need to set a password unless you explicitly disable protected
    # mode.
    # ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    bind 127.0.0.1 -::1  #绑定本地的IPV4和IPV6连接，::1代表IPV6地址
    
    # By default, outgoing connections (from replica to master, from Sentinel to
    # instances, cluster bus, etc.) are not bound to a specific local address. In
    # most cases, this means the operating system will handle that based on routing
    # and the interface through which the connection goes out.
    #
    # Using bind-source-addr it is possible to configure a specific address to bind
    # to, which may also affect how the connection gets routed.
    #
    # Example:
    #
    # bind-source-addr 10.0.0.1
    
    # Protected mode is a layer of security protection, in order to avoid that
    # Redis instances left open on the internet are accessed and exploited.
    #
    # When protected mode is on and the default user has no password, the server
    # only accepts local connections from the IPv4 address (127.0.0.1), IPv6 address
    # (::1) or Unix domain sockets.
    #
    # By default protected mode is enabled. You should disable it only if
    # you are sure you want clients from other hosts to connect to Redis
    # even if no authentication is configured.
    protected-mode yes  #保护模式，默认开启
    
    # Redis uses default hardened security configuration directives to reduce the
    # attack surface on innocent users. Therefore, several sensitive configuration
    # directives are immutable, and some potentially-dangerous commands are blocked.
    #
    # Configuration directives that control files that Redis writes to (e.g., 'dir'
    # and 'dbfilename') and that aren't usually modified during runtime
    # are protected by making them immutable.
    #
    # Commands that can increase the attack surface of Redis and that aren't usually
    # called by users are blocked by default.
    #
    # These can be exposed to either all connections or just local ones by setting
    # each of the configs listed below to either of these values:
    #
    # no    - Block for any connection (remain immutable)
    # yes   - Allow for any connection (no protection)
    # local - Allow only for local connections. Ones originating from the
    #         IPv4 address (127.0.0.1), IPv6 address (::1) or Unix domain sockets.
    #
    # enable-protected-configs no
    # enable-debug-command no
    # enable-module-command no
    
    # Accept connections on the specified port, default is 6379 (IANA #815344).
    # If port 0 is specified Redis will not listen on a TCP socket.
    port 6379  #端口设置
    
    # TCP listen() backlog.
    #
    # In high requests-per-second environments you need a high backlog in order
    # to avoid slow clients connection issues. Note that the Linux kernel
    # will silently truncate it to the value of /proc/sys/net/core/somaxconn so
    # make sure to raise both the value of somaxconn and tcp_max_syn_backlog
    # in order to get the desired effect.
    tcp-backlog 511
    
    # Unix socket.
    #
    # Specify the path for the Unix socket that will be used to listen for
    # incoming connections. There is no default, so Redis will not listen
    # on a unix socket when not specified.
    #
    # unixsocket /run/redis.sock
    # unixsocketperm 700
    
    # Close the connection after a client is idle for N seconds (0 to disable)
    timeout 0
    
    # TCP keepalive.
    #
    # If non-zero, use SO_KEEPALIVE to send TCP ACKs to clients in absence
    # of communication. This is useful for two reasons:
    #
    # 1) Detect dead peers.
    # 2) Force network equipment in the middle to consider the connection to be
    #    alive.
    #
    # On Linux, the specified value (in seconds) is the period used to send ACKs.
    # Note that to close the connection the double of the time is needed.
    # On other kernels the period depends on the kernel configuration.
    #
    # A reasonable value for this option is 300 seconds, which is the new
    # Redis default starting with Redis 3.2.1.
    tcp-keepalive 300
    
    # Apply OS-specific mechanism to mark the listening socket with the specified
    # ID, to support advanced routing and filtering capabilities.
    #
    # On Linux, the ID represents a connection mark.
    # On FreeBSD, the ID represents a socket cookie ID.
    # On OpenBSD, the ID represents a route table ID.
    #
    # The default value is 0, which implies no marking is required.
    # socket-mark-id 0
    ```

4. 通用配置

    ```bash
    ################################# GENERAL #####################################
    
    # By default Redis does not run as a daemon. Use 'yes' if you need it.
    # Note that Redis will write a pid file in /var/run/redis.pid when daemonized.
    # When Redis is supervised by upstart or systemd, this parameter has no impact.
    daemonize yes  #默认以守护进程方式运行
    
    # If you run Redis from upstart or systemd, Redis can interact with your
    # supervision tree. Options:
    #   supervised no      - no supervision interaction
    #   supervised upstart - signal upstart by putting Redis into SIGSTOP mode
    #                        requires "expect stop" in your upstart job config
    #   supervised systemd - signal systemd by writing READY=1 to $NOTIFY_SOCKET
    #                        on startup, and updating Redis status on a regular
    #                        basis.
    #   supervised auto    - detect upstart or systemd method based on
    #                        UPSTART_JOB or NOTIFY_SOCKET environment variables
    # Note: these supervision methods only signal "process is ready."
    #       They do not enable continuous pings back to your supervisor.
    #
    # The default is "no". To run under upstart/systemd, you can simply uncomment
    # the line below:
    #
    # supervised auto
    
    # If a pid file is specified, Redis writes it where specified at startup
    # and removes it at exit.
    #
    # When the server runs non daemonized, no pid file is created if none is
    # specified in the configuration. When the server is daemonized, the pid file
    # is used even if not specified, defaulting to "/var/run/redis.pid".
    #
    # Creating a pid file is best effort: if Redis is not able to create it
    # nothing bad happens, the server will start and run normally.
    #
    # Note that on modern Linux systems "/run/redis.pid" is more conforming
    # and should be used instead.
    pidfile /var/run/redis_6379.pid  #如果以守护进程方式运行，需要指定一个pid进程文件
    
    # Specify the server verbosity level.
    # This can be one of:
    # debug (a lot of information, useful for development/testing)
    # verbose (many rarely useful info, but not a mess like the debug level)
    # notice (moderately verbose, what you want in production probably)
    # warning (only very important / critical messages are logged)
    loglevel notice  #日志级别
    
    # Specify the log file name. Also the empty string can be used to force
    # Redis to log on the standard output. Note that if you use standard
    # output for logging but daemonize, logs will be sent to /dev/null
    logfile ""  #日志的文件名
    
    # To enable logging to the system logger, just set 'syslog-enabled' to yes,
    # and optionally update the other syslog parameters to suit your needs.
    # syslog-enabled no
    
    # Specify the syslog identity.
    # syslog-ident redis
    
    # Specify the syslog facility. Must be USER or between LOCAL0-LOCAL7.
    # syslog-facility local0
    
    # To disable the built in crash log, which will possibly produce cleaner core
    # dumps when they are needed, uncomment the following:
    #
    # crash-log-enabled no
    
    # To disable the fast memory check that's run as part of the crash log, which
    # will possibly let redis terminate sooner, uncomment the following:
    #
    # crash-memcheck-enabled no
    
    # Set the number of databases. The default database is DB 0, you can select
    # a different one on a per-connection basis using SELECT <dbid> where
    # dbid is a number between 0 and 'databases'-1
    databases 16  #默认数据库数量
    
    # By default Redis shows an ASCII art logo only when started to log to the
    # standard output and if the standard output is a TTY and syslog logging is
    # disabled. Basically this means that normally a logo is displayed only in
    # interactive sessions.
    #
    # However it is possible to force the pre-4.0 behavior and always show a
    # ASCII art logo in startup logs by setting the following option to yes.
    always-show-logo no  #是否总是显示logo
    
    # By default, Redis modifies the process title (as seen in 'top' and 'ps') to
    # provide some runtime information. It is possible to disable this and leave
    # the process name as executed by setting the following to no.
    set-proc-title yes
    
    # When changing the process title, Redis uses the following template to construct
    # the modified title.
    #
    # Template variables are specified in curly brackets. The following variables are
    # supported:
    #
    # {title}           Name of process as executed if parent, or type of child process.
    # {listen-addr}     Bind address or '*' followed by TCP or TLS port listening on, or
    #                   Unix socket if only that's available.
    # {server-mode}     Special mode, i.e. "[sentinel]" or "[cluster]".
    # {port}            TCP port listening on, or 0.
    # {tls-port}        TLS port listening on, or 0.
    # {unixsocket}      Unix domain socket listening on, or "".
    # {config-file}     Name of configuration file used.
    #
    proc-title-template "{title} {listen-addr} {server-mode}"
    ```

5. 快照 (SNAPSHOTTING)

    与持久化相关，

    ```bash
    ################################ SNAPSHOTTING  ################################
    
    # Save the DB to disk.
    #
    # save <seconds> <changes> [<seconds> <changes> ...]
    #
    # Redis will save the DB if the given number of seconds elapsed and it
    # surpassed the given number of write operations against the DB.
    #
    # Snapshotting can be completely disabled with a single empty string argument
    # as in following example:
    #
    # save ""
    #
    # Unless specified otherwise, by default Redis will save the DB:
    #   * After 3600 seconds (an hour) if at least 1 change was performed
    #   * After 300 seconds (5 minutes) if at least 100 changes were performed
    #   * After 60 seconds if at least 10000 changes were performed
    #
    # You can set these explicitly by uncommenting the following line.
    #
    # save 3600 1 300 100 60 10000
    # 如果一小时内至少有一次修改、五分钟内至少100次修改、一分钟内至少10000次修改则进行rdb持久化
    
    # By default Redis will stop accepting writes if RDB snapshots are enabled
    # (at least one save point) and the latest background save failed.
    # This will make the user aware (in a hard way) that data is not persisting
    # on disk properly, otherwise chances are that no one will notice and some
    # disaster will happen.
    #
    # If the background saving process will start working again Redis will
    # automatically allow writes again.
    #
    # However if you have setup your proper monitoring of the Redis server
    # and persistence, you may want to disable this feature so that Redis will
    # continue to work as usual even if there are problems with disk,
    # permissions, and so forth.
    stop-writes-on-bgsave-error yes  #如果持久化出错，是否继续工作
    
    # Compress string objects using LZF when dump .rdb databases?
    # By default compression is enabled as it's almost always a win.
    # If you want to save some CPU in the saving child set it to 'no' but
    # the dataset will likely be bigger if you have compressible values or keys.
    rdbcompression yes  #是否压缩rdb文件，需要消耗一些cpu资源
    
    # Since version 5 of RDB a CRC64 checksum is placed at the end of the file.
    # This makes the format more resistant to corruption but there is a performance
    # hit to pay (around 10%) when saving and loading RDB files, so you can disable it
    # for maximum performances.
    #
    # RDB files created with checksum disabled have a checksum of zero that will
    # tell the loading code to skip the check.
    rdbchecksum yes  #保存rdb文件时是否进行校验
    
    # Enables or disables full sanitization checks for ziplist and listpack etc when
    # loading an RDB or RESTORE payload. This reduces the chances of a assertion or
    # crash later on while processing commands.
    # Options:
    #   no         - Never perform full sanitization
    #   yes        - Always perform full sanitization
    #   clients    - Perform full sanitization only for user connections.
    #                Excludes: RDB files, RESTORE commands received from the master
    #                connection, and client connections which have the
    #                skip-sanitize-payload ACL flag.
    # The default should be 'clients' but since it currently affects cluster
    # resharding via MIGRATE, it is temporarily set to 'no' by default.
    #
    # sanitize-dump-payload no
    
    # The filename where to dump the DB
    dbfilename dump.rdb  #rdb文件名
    
    # Remove RDB files used by replication in instances without persistence
    # enabled. By default this option is disabled, however there are environments
    # where for regulations or other security concerns, RDB files persisted on
    # disk by masters in order to feed replicas, or stored on disk by replicas
    # in order to load them for the initial synchronization, should be deleted
    # ASAP. Note that this option ONLY WORKS in instances that have both AOF
    # and RDB persistence disabled, otherwise is completely ignored.
    #
    # An alternative (and sometimes better) way to obtain the same effect is
    # to use diskless replication on both master and replicas instances. However
    # in the case of replicas, diskless is not always an option.
    rdb-del-sync-files no  #rdb文件是否删除同步锁
    
    # The working directory.
    #
    # The DB will be written inside this directory, with the filename specified
    # above using the 'dbfilename' configuration directive.
    #
    # The Append Only File will also be created inside this directory.
    #
    # Note that you must specify a directory here, not a file name.
    dir ./  #rdb文件保存目录
    ```

6. 主从复制 (REPLICATION)

    ```bash
    ################################# REPLICATION #################################
    
    # Master-Replica replication. Use replicaof to make a Redis instance a copy of
    # another Redis server. A few things to understand ASAP about Redis replication.
    #
    #   +------------------+      +---------------+
    #   |      Master      | ---> |    Replica    |
    #   | (receive writes) |      |  (exact copy) |
    #   +------------------+      +---------------+
    #
    # 1) Redis replication is asynchronous, but you can configure a master to
    #    stop accepting writes if it appears to be not connected with at least
    #    a given number of replicas.
    # 2) Redis replicas are able to perform a partial resynchronization with the
    #    master if the replication link is lost for a relatively small amount of
    #    time. You may want to configure the replication backlog size (see the next
    #    sections of this file) with a sensible value depending on your needs.
    # 3) Replication is automatic and does not need user intervention. After a
    #    network partition replicas automatically try to reconnect to masters
    #    and resynchronize with them.
    #
    # replicaof <masterip> <masterport>  #配置主机的ip和端口
    
    # If the master is password protected (using the "requirepass" configuration
    # directive below) it is possible to tell the replica to authenticate before
    # starting the replication synchronization process, otherwise the master will
    # refuse the replica request.
    #
    # masterauth <master-password>  #配置主机的密码
    #
    # However this is not enough if you are using Redis ACLs (for Redis version
    # 6 or greater), and the default user is not capable of running the PSYNC
    # command and/or other commands needed for replication. In this case it's
    # better to configure a special user to use with replication, and specify the
    # masteruser configuration as such:
    #
    # masteruser <username>  #配置主机的用户名
    #
    # When masteruser is specified, the replica will authenticate against its
    # master using the new AUTH form: AUTH <username> <password>.
    
    # When a replica loses its connection with the master, or when the replication
    # is still in progress, the replica can act in two different ways:
    #
    # 1) if replica-serve-stale-data is set to 'yes' (the default) the replica will
    #    still reply to client requests, possibly with out of date data, or the
    #    data set may just be empty if this is the first synchronization.
    #
    # 2) If replica-serve-stale-data is set to 'no' the replica will reply with error
    #    "MASTERDOWN Link with MASTER is down and replica-serve-stale-data is set to 'no'"
    #    to all data access commands, excluding commands such as:
    #    INFO, REPLICAOF, AUTH, SHUTDOWN, REPLCONF, ROLE, CONFIG, SUBSCRIBE,
    #    UNSUBSCRIBE, PSUBSCRIBE, PUNSUBSCRIBE, PUBLISH, PUBSUB, COMMAND, POST,
    #    HOST and LATENCY.
    #
    replica-serve-stale-data yes  #当主机和从机之前断开连接后，从机是否继续向客户端提供查询服务；yes表示继续提供，no表示除了不涉及查询的命令可以执行，其他命令均返回提示信息
    
    # You can configure a replica instance to accept writes or not. Writing against
    # a replica instance may be useful to store some ephemeral data (because data
    # written on a replica will be easily deleted after resync with the master) but
    # may also cause problems if clients are writing to it because of a
    # misconfiguration.
    #
    # Since Redis 2.6 by default replicas are read-only.
    #
    # Note: read only replicas are not designed to be exposed to untrusted clients
    # on the internet. It's just a protection layer against misuse of the instance.
    # Still a read only replica exports by default all the administrative commands
    # such as CONFIG, DEBUG, and so forth. To a limited extent you can improve
    # security of read only replicas using 'rename-command' to shadow all the
    # administrative / dangerous commands.
    replica-read-only yes  #设置从节点只读命令
    
    # Replication SYNC strategy: disk or socket.
    #
    # New replicas and reconnecting replicas that are not able to continue the
    # replication process just receiving differences, need to do what is called a
    # "full synchronization". An RDB file is transmitted from the master to the
    # replicas.
    #
    # The transmission can happen in two different ways:
    #
    # 1) Disk-backed: The Redis master creates a new process that writes the RDB
    #                 file on disk. Later the file is transferred by the parent
    #                 process to the replicas incrementally.
    # 2) Diskless: The Redis master creates a new process that directly writes the
    #              RDB file to replica sockets, without touching the disk at all.
    #
    # With disk-backed replication, while the RDB file is generated, more replicas
    # can be queued and served with the RDB file as soon as the current child
    # producing the RDB file finishes its work. With diskless replication instead
    # once the transfer starts, new replicas arriving will be queued and a new
    # transfer will start when the current one terminates.
    #
    # When diskless replication is used, the master waits a configurable amount of
    # time (in seconds) before starting the transfer in the hope that multiple
    # replicas will arrive and the transfer can be parallelized.
    #
    # With slow disks and fast (large bandwidth) networks, diskless replication
    # works better.
    repl-diskless-sync yes
    
    # When diskless replication is enabled, it is possible to configure the delay
    # the server waits in order to spawn the child that transfers the RDB via socket
    # to the replicas.
    #
    # This is important since once the transfer starts, it is not possible to serve
    # new replicas arriving, that will be queued for the next RDB transfer, so the
    # server waits a delay in order to let more replicas arrive.
    #
    # The delay is specified in seconds, and by default is 5 seconds. To disable
    # it entirely just set it to 0 seconds and the transfer will start ASAP.
    repl-diskless-sync-delay 5
    
    # When diskless replication is enabled with a delay, it is possible to let
    # the replication start before the maximum delay is reached if the maximum
    # number of replicas expected have connected. Default of 0 means that the
    # maximum is not defined and Redis will wait the full delay.
    repl-diskless-sync-max-replicas 0
    ...
    ```

    

7. 安全 (SECURITY)

    ```bash
    config get requirepass  #查看密码设置
    
    config set requirepass 123456  #设置密码
    
    auth 123456  #使用密码登录
    ```

8. 客户端限制

    ```bash
    ################################### CLIENTS ####################################
    
    # Set the max number of connected clients at the same time. By default
    # this limit is set to 10000 clients, however if the Redis server is not
    # able to configure the process file limit to allow for the specified limit
    # the max number of allowed clients is set to the current file limit
    # minus 32 (as Redis reserves a few file descriptors for internal uses).
    #
    # Once the limit is reached Redis will close all the new connections sending
    # an error 'max number of clients reached'.
    #
    # IMPORTANT: When Redis Cluster is used, the max number of connections is also
    # shared with the cluster bus: every node in the cluster will use two
    # connections, one incoming and another outgoing. It is important to size the
    # limit accordingly in case of very large clusters.
    #
    # maxclients 10000  #设置最大客户端数量
    
    ############################## MEMORY MANAGEMENT ################################
    
    # Set a memory usage limit to the specified amount of bytes.
    # When the memory limit is reached Redis will try to remove keys
    # according to the eviction policy selected (see maxmemory-policy).
    #
    # If Redis can't remove keys according to the policy, or if the policy is
    # set to 'noeviction', Redis will start to reply with errors to commands
    # that would use more memory, like SET, LPUSH, and so on, and will continue
    # to reply to read-only commands like GET.
    #
    # This option is usually useful when using Redis as an LRU or LFU cache, or to
    # set a hard memory limit for an instance (using the 'noeviction' policy).
    #
    # WARNING: If you have replicas attached to an instance with maxmemory on,
    # the size of the output buffers needed to feed the replicas are subtracted
    # from the used memory count, so that network problems / resyncs will
    # not trigger a loop where keys are evicted, and in turn the output
    # buffer of replicas is full with DELs of keys evicted triggering the deletion
    # of more keys, and so forth until the database is completely emptied.
    #
    # In short... if you have replicas attached it is suggested that you set a lower
    # limit for maxmemory so that there is some free RAM on the system for replica
    # output buffers (but this is not needed if the policy is 'noeviction').
    #
    # maxmemory <bytes>  #最大内存设置
    
    # MAXMEMORY POLICY: how Redis will select what to remove when maxmemory
    # is reached. You can select one from the following behaviors:
    #
    # volatile-lru -> Evict using approximated LRU, only keys with an expire set.
    # allkeys-lru -> Evict any key using approximated LRU.
    # volatile-lfu -> Evict using approximated LFU, only keys with an expire set.
    # allkeys-lfu -> Evict any key using approximated LFU.
    # volatile-random -> Remove a random key having an expire set.
    # allkeys-random -> Remove a random key, any key.
    # volatile-ttl -> Remove the key with the nearest expire time (minor TTL)
    # noeviction -> Don't evict anything, just return an error on write operations.
    #
    # LRU means Least Recently Used
    # LFU means Least Frequently Used
    #
    # Both LRU, LFU and volatile-ttl are implemented using approximated
    # randomized algorithms.
    #
    # Note: with any of the above policies, when there are no suitable keys for
    # eviction, Redis will return an error on write operations that require
    # more memory. These are usually commands that create new keys, add data or
    # modify existing keys. A few examples are: SET, INCR, HSET, LPUSH, SUNIONSTORE,
    # SORT (due to the STORE argument), and EXEC (if the transaction includes any
    # command that requires memory).
    #
    # The default is:
    #
    # maxmemory-policy noeviction  #内存到达上限之后的处理策略
    #
    # 1.volatile-lru: 只对设置了过期时间的key进行LRU
    # 2.allkeys-lru: 删除lru算法的key
    # 3.volatile-random: 随机删除即将过期的key
    # 4.allkeys-random: 随机删除
    # 5.volatile-ttl: 删除即将过期的
    # 6.noeviction: 永不过期, 返回错误
    ```

9. AOF配置

    ```bash
    ############################## APPEND ONLY MODE ###############################
    
    # By default Redis asynchronously dumps the dataset on disk. This mode is
    # good enough in many applications, but an issue with the Redis process or
    # a power outage may result into a few minutes of writes lost (depending on
    # the configured save points).
    #
    # The Append Only File is an alternative persistence mode that provides
    # much better durability. For instance using the default data fsync policy
    # (see later in the config file) Redis can lose just one second of writes in a
    # dramatic event like a server power outage, or a single write if something
    # wrong with the Redis process itself happens, but the operating system is
    # still running correctly.
    #
    # AOF and RDB persistence can be enabled at the same time without problems.
    # If the AOF is enabled on startup Redis will load the AOF, that is the file
    # with the better durability guarantees.
    #
    # Please check https://redis.io/topics/persistence for more information.
    
    appendonly no  #默认不开启aof模式，使用rdb方式持久化，大多数情况下rdb完全够用
    
    # The base name of the append only file.
    #
    # Redis 7 and newer use a set of append-only files to persist the dataset
    # and changes applied to it. There are two basic types of files in use:
    #
    # - Base files, which are a snapshot representing the complete state of the
    #   dataset at the time the file was created. Base files can be either in
    #   the form of RDB (binary serialized) or AOF (textual commands).
    # - Incremental files, which contain additional commands that were applied
    #   to the dataset following the previous file.
    #
    # In addition, manifest files are used to track the files and the order in
    # which they were created and should be applied.
    #
    # Append-only file names are created by Redis following a specific pattern.
    # The file name's prefix is based on the 'appendfilename' configuration
    # parameter, followed by additional information about the sequence and type.
    #
    # For example, if appendfilename is set to appendonly.aof, the following file
    # names could be derived:
    #
    # - appendonly.aof.1.base.rdb as a base file.
    # - appendonly.aof.1.incr.aof, appendonly.aof.2.incr.aof as incremental files.
    # - appendonly.aof.manifest as a manifest file.
    
    appendfilename "appendonly.aof"  #aof文件名设置
    
    # For convenience, Redis stores all persistent append-only files in a dedicated
    # directory. The name of the directory is determined by the appenddirname
    # configuration parameter.
    
    appenddirname "appendonlydir"  #aof文件目录设置
    
    # The fsync() call tells the Operating System to actually write data on disk
    # instead of waiting for more data in the output buffer. Some OS will really flush
    # data on disk, some other OS will just try to do it ASAP.
    #
    # Redis supports three different modes:
    #
    # no: don't fsync, just let the OS flush the data when it wants. Faster.
    # always: fsync after every write to the append only log. Slow, Safest.
    # everysec: fsync only one time every second. Compromise.
    #
    # The default is "everysec", as that's usually the right compromise between
    # speed and data safety. It's up to you to understand if you can relax this to
    # "no" that will let the operating system flush the output buffer when
    # it wants, for better performances (but if you can live with the idea of
    # some data loss consider the default persistence mode that's snapshotting),
    # or on the contrary, use "always" that's very slow but a bit safer than
    # everysec.
    #
    # More details please check the following article:
    # http://antirez.com/post/redis-persistence-demystified.html
    #
    # If unsure, use "everysec".
    
    # appendfsync always  #每次修改都同步，消耗性能
    appendfsync everysec  #每秒执行一次同步数据，如果服务器宕机可能会丢失数据
    # appendfsync no  #不同步
    ```

## Redis持久化

Redis是内存数据库，为了防止断电易失。Redis默认采用RDB持久化方式。

### RDB

![image9](../../img/2022/11/redis_note/image9.png)

在指定的时间间隔内将内存中的数据集快照写入磁盘，恢复时直接将快照读入内存中。

> 步骤解读：
>
> 1. Redis会单独创建(fork)一个子进程来进行持久化
> 2. 子进程将内存中的数据写入临时的rdb文件(dump.rdb)
> 3. 父进程则继续处理Redis客户端发送的指令
> 4. 临时rdb生成完成后，将新的rdb文件替换之前持久化生成的文件，子进程结束
>
> 整个过程主进程不会进行任何IO操作，所以保证了性能，但缺点是最后一次持久化后的数据可能会丢失。

**RDB持久化触发情景：**

1. 满足配置文件中设置的save配置时
2. redis客户端发送save命令
3. 退出redis服务(shutdown命令)时
4. 执行flushall命令时

只要将rdb文件放在redis启动目录下，redis启动时会自动检查并读取rdb文件进行数据恢复

查看redis启动目录：`config get dir`

**优点**

1. 适合大规模的数据恢复
2. 对数据完成性要求不高，如果redis服务器在还不满足save配置的时候宕机，就会发生数据丢失

**缺点**

1. 需要满足save配置的时间间隔才会触发rdb持久化
2. 如果服务器宕机，最后一次持久化后修改的数据会丢失

### AOF (Append Only File)

将所有命令记录下来（历史记录），恢复的时候就是把这个文件重新执行一遍。

![image10](../../img/2022/11/redis_note/image10.png)

以日志的形式记录每个写命令，追加到日志结尾。

redis启动后会读取该文件重新构建数据，也就是重新执行一遍日志中记录的所有写操作。

```bash
############################## APPEND ONLY MODE ###############################

# By default Redis asynchronously dumps the dataset on disk. This mode is
# good enough in many applications, but an issue with the Redis process or
# a power outage may result into a few minutes of writes lost (depending on
# the configured save points).
#
# The Append Only File is an alternative persistence mode that provides
# much better durability. For instance using the default data fsync policy
# (see later in the config file) Redis can lose just one second of writes in a
# dramatic event like a server power outage, or a single write if something
# wrong with the Redis process itself happens, but the operating system is
# still running correctly.
#
# AOF and RDB persistence can be enabled at the same time without problems.
# If the AOF is enabled on startup Redis will load the AOF, that is the file
# with the better durability guarantees.
#
# Please check https://redis.io/topics/persistence for more information.

appendonly no  #默认是不开启的，设置成yes为开启

# The base name of the append only file.
#
# Redis 7 and newer use a set of append-only files to persist the dataset
# and changes applied to it. There are two basic types of files in use:
#
# - Base files, which are a snapshot representing the complete state of the
#   dataset at the time the file was created. Base files can be either in
#   the form of RDB (binary serialized) or AOF (textual commands).
# - Incremental files, which contain additional commands that were applied
#   to the dataset following the previous file.
#
# In addition, manifest files are used to track the files and the order in
# which they were created and should be applied.
#
# Append-only file names are created by Redis following a specific pattern.
# The file name's prefix is based on the 'appendfilename' configuration
# parameter, followed by additional information about the sequence and type.
#
# For example, if appendfilename is set to appendonly.aof, the following file
# names could be derived:
#
# - appendonly.aof.1.base.rdb as a base file.
# - appendonly.aof.1.incr.aof, appendonly.aof.2.incr.aof as incremental files.
# - appendonly.aof.manifest as a manifest file.

appendfilename "appendonly.aof"

# For convenience, Redis stores all persistent append-only files in a dedicated
# directory. The name of the directory is determined by the appenddirname
# configuration parameter.

appenddirname "appendonlydir"

# The fsync() call tells the Operating System to actually write data on disk
# instead of waiting for more data in the output buffer. Some OS will really flush
# data on disk, some other OS will just try to do it ASAP.
#
# Redis supports three different modes:
#
# no: don't fsync, just let the OS flush the data when it wants. Faster.
# always: fsync after every write to the append only log. Slow, Safest.
# everysec: fsync only one time every second. Compromise.
#
# The default is "everysec", as that's usually the right compromise between
# speed and data safety. It's up to you to understand if you can relax this to
# "no" that will let the operating system flush the output buffer when
# it wants, for better performances (but if you can live with the idea of
# some data loss consider the default persistence mode that's snapshotting),
# or on the contrary, use "always" that's very slow but a bit safer than
# everysec.
#
# More details please check the following article:
# http://antirez.com/post/redis-persistence-demystified.html
#
# If unsure, use "everysec".

# appendfsync always  #每次修改追加一次，并同步到磁盘
appendfsync everysec  #aof持久化策略，每秒钟追加一次，并同步到磁盘中
# appendfsync no  #只追加到aof文件，不写入磁盘，交给操作系统去决定什么时候要写到磁盘

# When the AOF fsync policy is set to always or everysec, and a background
# saving process (a background save or AOF log background rewriting) is
# performing a lot of I/O against the disk, in some Linux configurations
# Redis may block too long on the fsync() call. Note that there is no fix for
# this currently, as even performing fsync in a different thread will block
# our synchronous write(2) call.
#
# In order to mitigate this problem it's possible to use the following option
# that will prevent fsync() from being called in the main process while a
# BGSAVE or BGREWRITEAOF is in progress.
#
# This means that while another child is saving, the durability of Redis is
# the same as "appendfsync no". In practical terms, this means that it is
# possible to lose up to 30 seconds of log in the worst scenario (with the
# default Linux settings).
#
# If you have latency problems turn this to "yes". Otherwise leave it as
# "no" that is the safest pick from the point of view of durability.

no-appendfsync-on-rewrite no  #当写入磁盘时由于大量IO操作，可能会阻塞主进程。如果设置为yes那么就不再写入磁盘中，而是交给操作系统决定何时写入磁盘，减少了IO操作，也就不会阻塞，但是数据可能丢失，因为没有及时写入磁盘中。默认为no，这样是安全的。

# Automatic rewrite of the append only file.
# Redis is able to automatically rewrite the log file implicitly calling
# BGREWRITEAOF when the AOF log size grows by the specified percentage.
#
# This is how it works: Redis remembers the size of the AOF file after the
# latest rewrite (if no rewrite has happened since the restart, the size of
# the AOF at startup is used).
#
# This base size is compared to the current size. If the current size is
# bigger than the specified percentage, the rewrite is triggered. Also
# you need to specify a minimal size for the AOF file to be rewritten, this
# is useful to avoid rewriting the AOF file even if the percentage increase
# is reached but it is still pretty small.
#
# Specify a percentage of zero in order to disable the automatic AOF
# rewrite feature.

auto-aof-rewrite-percentage 100  #当aof文件大小超过上一次重写的aof文件的百分之多少进行重写
auto-aof-rewrite-min-size 64mb  #设置允许重写的最小文件大小。防止超过百分比后文件依然很小，这时如果没超过最小文件大小就不重写

# An AOF file may be found to be truncated at the end during the Redis
# startup process, when the AOF data gets loaded back into memory.
# This may happen when the system where Redis is running
# crashes, especially when an ext4 filesystem is mounted without the
# data=ordered option (however this can't happen when Redis itself
# crashes or aborts but the operating system still works correctly).
#
# Redis can either exit with an error when this happens, or load as much
# data as possible (the default now) and start if the AOF file is found
# to be truncated at the end. The following option controls this behavior.
#
# If aof-load-truncated is set to yes, a truncated AOF file is loaded and
# the Redis server starts emitting a log to inform the user of the event.
# Otherwise if the option is set to no, the server aborts with an error
# and refuses to start. When the option is set to no, the user requires
# to fix the AOF file using the "redis-check-aof" utility before to restart
# the server.
#
# Note that if the AOF file will be found to be corrupted in the middle
# the server will still exit with an error. This option only applies when
# Redis will try to read more data from the AOF file but not enough bytes
# will be found.
aof-load-truncated yes

# Redis can create append-only base files in either RDB or AOF formats. Using
# the RDB format is always faster and more efficient, and disabling it is only
# supported for backward compatibility purposes.
aof-use-rdb-preamble yes

# Redis supports recording timestamp annotations in the AOF to support restoring
# the data from a specific point-in-time. However, using this capability changes
# the AOF format in a way that may not be compatible with existing AOF parsers.
aof-timestamp-enabled no

```

如果aof文件有错误，那么无法启动redis客户端，可以用redis-check-aof来修复aof文件，但是数据会有所缺失

```sh
redis-check-aof --fix appendonly.aof
```

**优点**

1. 每一次修改都追加，文件完整性好，但是性能消耗高
2. 每秒同步一次，可能会丢失一秒的数据
3. 从不同步，效率最高

**缺点**

1. 相对于数据文件来说，aof文件要远远大于rdb文件，修复的速度也比rdb慢
2. 因为aof本质上是追加写文件，是IO操作，所以aof运行效率也比rdb慢，所以redis默认就是rdb持久化

**扩展**

> 如果两种持久化同时开启，redis重启时会优先载入aof文件来恢复数据
>
> 一般rdb用于备份redis数据，只在从服务器上进行备份，15分钟备份一次，也就是save 900 1

## Redis发布订阅

redis发布订阅是一种消息模式，发送者发送消息，订阅者接收消息

![image11](../../img/2022/11/redis_note/image11.png)

redis客户端可以订阅任意数量的频道

```bash
psubscribe pattern...  #订阅一个或多个符合给定模式的频道

pubsub subcommand argument...  #查看订阅与发布系统状态

publish channel message  #将信息发送到指定频道

subscribe channel...  #订阅一个或多个频道

punsubscribe pattern...  #退订所有给定模式的频道

unsubscribe channel...  #退订给定的频道
```

**底层实现**

每个频道是一个字典，字典的键用来标识频道，字典的值是一个链表，链表中是该频道的所有订阅者

**使用场景**

1. 实时消息系统
2. 实时聊天室
3. 订阅公众号等

## Redis主从复制

### 概念

主从复制是指将一台Redis服务器的数据复制到从节点的服务器上，其中主节点负责写数据，从节点负责读取数据。主从复制是单向的，有主节点到从节点。主节点可以有任意个从节点，而从节点只能有一个主节点。

![image12](../../img/2022/11/redis_note/image12.png)

### 作用

1. 数据备份：主从复制实现了数据的热备份
2. 故障修复：当主节点出现问题可以使用从节点
3. 负载均衡：主从复制加读写分离，写数据时请求主服务器，读数据时请求从服务器，可以分担服务器的访问压力
4. 高可用基石：主从复制是哨兵模式和集群实施的基础

一般redis需要配置两从一主，只配置一台redis服务器有两个弊端：

1. 从结构上来说，单个服务器可能宕机，并且所有请求访问一台服务器，压力太大
2. 从容量上来说，单个服务器容量有限，一般单台redis服务器的最大内存使用量应该不超过20G

### 搭建环境

```bash
info replication  #查看主从复制信息

# Replication
role:master  #角色
connected_slaves:0  #连接的从节点个数
master_failover_state:no-failover
master_replid:9a9ee312a4acaeb3ec299c0a9082833d116fe92d
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0
second_repl_offset:-1
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
```

在一台服务器上启动3个redis服务进程来模拟一主两从

1. 复制出3个配置文件，并修改端口号、日志文件名称、dump文件名称、pid名称
2. 使用不同的配置文件启动redis服务

![image13](../../img/2022/11/redis_note/image13.png)

3. 配置从机

    ```redis
    slaveof host port  #配置以host:port为主机
    ```

4. 在主机上可以查看到从机的信息

![image14](../../img/2022/11/redis_note/image14.png)

除了通过命令配置主从之外还可以修改配置文件进行配置，这样是永久配置。具体看conf详解

### 测试

**当主机断开连接后，两台从机依然是从机的角色，主机重新连接后，依然可以写数据。**

**当从机断开连接后再次重新连接，如果从机的角色为主机 (通过命令配置从机的方式重启后角色变为主机) ，则无法同步主机的数据，只要重新配置为从机就可以同步到主机上的所有数据；如果从机的角色还是从机 (修改配置文件的方式重启后依然为从机) ，则能够同步到宕机期间主机写入的数据。**

**只要从机连接到主机，从机就会向主机发送sync同步命令，主机将数据文件传送给从机，进行一次全量同步。全量同步之后主机上的写操作都是增量同步给从机。**

### 第二种主从复制方法

![image15](../../img/2022/11/redis_note/image15.png)

如果主机宕机后，需要手动执行```slaveof no one```命令将一个从机设置为主机。当原来的主机重新连接后就会出现两个主机，还需要用```slaveof ip porrt```手动改为从机。

## 哨兵模式

Redis2.8之后开始提供哨兵模式，当主服务器宕机后能够自动将一台从机切换为主机

### 原理

哨兵是一个独立的进程，通过不断向所有redis服务器发送命令来判断各个redis服务器是否宕机

当哨兵检测到主服务器宕机后，会自动将一台从服务器切换为主服务器，然后通过发布订阅模式通知其他的从服务器，修改配置文件，让从服务器切换主机

![image16](../../img/2022/11/redis_note/image16.png)

但是如果只有一个哨兵，哨兵进程也可能死掉，所以多哨兵模式就是防止一个哨兵挂掉

![image17](../../img/2022/11/redis_note/image17.png)

如果主服务器宕机，哨兵1检测到之后不会马上重新选择主服务器，只是认为主服务器现在不可用，这个现象称为主观下线，如果还有其他哨兵也检测到主服务器不可用，达到一定数量之后，所有哨兵会投票选出一个从服务器来暂时充当主服务器，由某个哨兵发布消息通知服务器切换自己的主服务器。

### 配置哨兵模式

哨兵配置文件为sentinel.conf

```bash
# Example sentinel.conf

# By default protected mode is disabled in sentinel mode. Sentinel is reachable
# from interfaces different than localhost. Make sure the sentinel instance is
# protected from the outside world via firewalling or other means.
protected-mode no  #被保护模式，默认关闭

# port <sentinel-port>
# The port that this sentinel instance will run on
port 26379  #哨兵运行端口

# By default Redis Sentinel does not run as a daemon. Use 'yes' if you need it.
# Note that Redis will write a pid file in /var/run/redis-sentinel.pid when
# daemonized.
daemonize no  #守护进程默认关闭

# When running daemonized, Redis Sentinel writes a pid file in
# /var/run/redis-sentinel.pid by default. You can specify a custom pid file
# location here.
pidfile /var/run/redis-sentinel.pid  #开启守护进程后哨兵进程的pid文件

# Specify the log file name. Also the empty string can be used to force
# Sentinel to log on the standard output. Note that if you use standard
# output for logging but daemonize, logs will be sent to /dev/null
logfile ""  #日志文件

# sentinel announce-ip <ip>
# sentinel announce-port <port>
#
# The above two configuration directives are useful in environments where,
# because of NAT, Sentinel is reachable from outside via a non-local address.
#
# When announce-ip is provided, the Sentinel will claim the specified IP address
# in HELLO messages used to gossip its presence, instead of auto-detecting the
# local address as it usually does.
#
# Similarly when announce-port is provided and is valid and non-zero, Sentinel
# will announce the specified TCP port.
#
# The two options don't need to be used together, if only announce-ip is
# provided, the Sentinel will announce the specified IP and the server port
# as specified by the "port" option. If only announce-port is provided, the
# Sentinel will announce the auto-detected local IP and the specified port.
#
# Example:
#
# sentinel announce-ip 1.2.3.4

# dir <working-directory>
# Every long running process should have a well-defined working directory.
# For Redis Sentinel to chdir to /tmp at startup is the simplest thing
# for the process to don't interfere with administrative tasks such as
# unmounting filesystems.
dir /tmp  #哨兵sentinel的工作目录

# sentinel monitor <master-name> <ip> <redis-port> <quorum>
#
# Tells Sentinel to monitor this master, and to consider it in O_DOWN
# (Objectively Down) state only if at least <quorum> sentinels agree.
#
# Note that whatever is the ODOWN quorum, a Sentinel will require to
# be elected by the majority of the known Sentinels in order to
# start a failover, so no failover can be performed in minority.
#
# Replicas are auto-discovered, so you don't need to specify replicas in
# any way. Sentinel itself will rewrite this configuration file adding
# the replicas using additional configuration options.
# Also note that the configuration file is rewritten when a
# replica is promoted to master.
#
# Note: master name should not include special characters or spaces.
# The valid charset is A-z 0-9 and the three characters ".-_".
sentinel monitor mymaster 127.0.0.1 6379 2  #quorum表示当多少个哨兵检测到主服务器宕机之后开始重新选择

# sentinel auth-pass <master-name> <password>
#
# Set the password to use to authenticate with the master and replicas.
# Useful if there is a password set in the Redis instances to monitor.
#
# Note that the master password is also used for replicas, so it is not
# possible to set a different password in masters and replicas instances
# if you want to be able to monitor these instances with Sentinel.
#
# However you can have Redis instances without the authentication enabled
# mixed with Redis instances requiring the authentication (as long as the
# password set is the same for all the instances requiring the password) as
# the AUTH command will have no effect in Redis instances with authentication
# switched off.
#
# Example:
#
# sentinel auth-pass mymaster MySUPER--secret-0123passw0rd  #哨兵连接主从的密码

# sentinel auth-user <master-name> <username>
#
# This is useful in order to authenticate to instances having ACL capabilities,
# that is, running Redis 6.0 or greater. When just auth-pass is provided the
# Sentinel instance will authenticate to Redis using the old "AUTH <pass>"
# method. When also an username is provided, it will use "AUTH <user> <pass>".
# In the Redis servers side, the ACL to provide just minimal access to
# Sentinel instances, should be configured along the following lines:
#
#     user sentinel-user >somepassword +client +subscribe +publish \
#                        +ping +info +multi +slaveof +config +client +exec on

# sentinel down-after-milliseconds <master-name> <milliseconds>
#
# Number of milliseconds the master (or any attached replica or sentinel) should
# be unreachable (as in, not acceptable reply to PING, continuously, for the
# specified period) in order to consider it in S_DOWN state (Subjectively
# Down).
#
# Default is 30 seconds.
sentinel down-after-milliseconds mymaster 30000  #指定多少毫秒之后，如果主节点没有应答，哨兵就认为主节点下线，默认30秒

# IMPORTANT NOTE: starting with Redis 6.2 ACL capability is supported for
# Sentinel mode, please refer to the Redis website https://redis.io/topics/acl
# for more details.

# Sentinel's ACL users are defined in the following format:
#
#   user <username> ... acl rules ...
#
# For example:
#
#   user worker +@admin +@connection ~* on >ffa9203c493aa99
#
# For more information about ACL configuration please refer to the Redis
# website at https://redis.io/topics/acl and redis server configuration 
# template redis.conf.

# ACL LOG
#
# The ACL Log tracks failed commands and authentication events associated
# with ACLs. The ACL Log is useful to troubleshoot failed commands blocked 
# by ACLs. The ACL Log is stored in memory. You can reclaim memory with 
# ACL LOG RESET. Define the maximum entry length of the ACL Log below.
acllog-max-len 128

# Using an external ACL file
#
# Instead of configuring users here in this file, it is possible to use
# a stand-alone file just listing users. The two methods cannot be mixed:
# if you configure users here and at the same time you activate the external
# ACL file, the server will refuse to start.
#
# The format of the external ACL user file is exactly the same as the
# format that is used inside redis.conf to describe users.
#
# aclfile /etc/redis/sentinel-users.acl

# requirepass <password>
#
# You can configure Sentinel itself to require a password, however when doing
# so Sentinel will try to authenticate with the same password to all the
# other Sentinels. So you need to configure all your Sentinels in a given
# group with the same "requirepass" password. Check the following documentation
# for more info: https://redis.io/topics/sentinel
#
# IMPORTANT NOTE: starting with Redis 6.2 "requirepass" is a compatibility
# layer on top of the ACL system. The option effect will be just setting
# the password for the default user. Clients will still authenticate using
# AUTH <password> as usually, or more explicitly with AUTH default <password>
# if they follow the new protocol: both will work.
#
# New config files are advised to use separate authentication control for
# incoming connections (via ACL), and for outgoing connections (via
# sentinel-user and sentinel-pass) 
#
# The requirepass is not compatible with aclfile option and the ACL LOAD
# command, these will cause requirepass to be ignored.

# sentinel sentinel-user <username>
#
# You can configure Sentinel to authenticate with other Sentinels with specific
# user name. 

# sentinel sentinel-pass <password>
#
# The password for Sentinel to authenticate with other Sentinels. If sentinel-user
# is not configured, Sentinel will use 'default' user with sentinel-pass to authenticate.

# sentinel parallel-syncs <master-name> <numreplicas>
#
# How many replicas we can reconfigure to point to the new replica simultaneously
# during the failover. Use a low number if you use the replicas to serve query
# to avoid that all the replicas will be unreachable at about the same
# time while performing the synchronization with the master.
sentinel parallel-syncs mymaster 1  #这个配置项指定了在主备切换时最多可以有多少个slave同时对新的master进行同步，这个数字越小，完成主备切换的时间就越长。但是如果这个数字太大，就意味有多少个slave因为同步数据而不可用，一般应该保证每次只有一个slave处于不能处理命令请求的状态。

# sentinel failover-timeout <master-name> <milliseconds>
#
# Specifies the failover timeout in milliseconds. It is used in many ways:
#
# - The time needed to re-start a failover after a previous failover was
#   already tried against the same master by a given Sentinel, is two
#   times the failover timeout.
#
# - The time needed for a replica replicating to a wrong master according
#   to a Sentinel current configuration, to be forced to replicate
#   with the right master, is exactly the failover timeout (counting since
#   the moment a Sentinel detected the misconfiguration).
#
# - The time needed to cancel a failover that is already in progress but
#   did not produced any configuration change (SLAVEOF NO ONE yet not
#   acknowledged by the promoted replica).
#
# - The maximum time a failover in progress waits for all the replicas to be
#   reconfigured as replicas of the new master. However even after this time
#   the replicas will be reconfigured by the Sentinels anyway, but not with
#   the exact parallel-syncs progression as specified.
#
# Default is 3 minutes.
# 故障转移的超时时间 failover-timeout 可以用在以下这些方面： 
# 1. 同一个sentinel对同一个master两次failover之间的间隔时间。
# 2. 当一个slave从一个错误的master那里同步数据开始计算时间。直到slave被纠正为向正确的master那里同步数据时。
# 3.当想要取消一个正在进行的failover所需要的时间。  
# 4.当进行failover时，配置所有slaves指向新的master所需的最大时间。不过，即使过了这个超时，slaves依然会被正确配置为指向   master，但是就不按parallel-syncs所配置的规则来了
# 默认三分钟
#
sentinel failover-timeout mymaster 180000

# SCRIPTS EXECUTION
#
# sentinel notification-script and sentinel reconfig-script are used in order
# to configure scripts that are called to notify the system administrator
# or to reconfigure clients after a failover. The scripts are executed
# with the following rules for error handling:
#
# If script exits with "1" the execution is retried later (up to a maximum
# number of times currently set to 10).
#
# If script exits with "2" (or an higher value) the script execution is
# not retried.
#
# If script terminates because it receives a signal the behavior is the same
# as exit code 1.
#
# A script has a maximum running time of 60 seconds. After this limit is
# reached the script is terminated with a SIGKILL and the execution retried.

# NOTIFICATION SCRIPT
#
# sentinel notification-script <master-name> <script-path>
# 
# Call the specified notification script for any sentinel event that is
# generated in the WARNING level (for instance -sdown, -odown, and so forth).
# This script should notify the system administrator via email, SMS, or any
# other messaging system, that there is something wrong with the monitored
# Redis systems.
#
# The script is called with just two arguments: the first is the event type
# and the second the event description.
#
# The script must exist and be executable in order for sentinel to start if
# this option is provided.
#
# Example:
#
# 配置当某一事件发生时所需要执行的脚本，可以通过脚本来通知管理员，例如当系统运行不正常时发邮件通知相关人员。
# 对于脚本的运行结果有以下规则：
# 若脚本执行后返回1，那么该脚本稍后将会被再次执行，重复次数目前默认为10
# 若脚本执行后返回2，或者比2更高的一个返回值，脚本将不会重复执行。
# 如果脚本在执行过程中由于收到系统中断信号被终止了，则同返回值为1时的行为相同。
# 一个脚本的最大执行时间为60s，如果超过这个时间，脚本将会被一个SIGKILL信号终止，之后重新执行。
 
# 通知型脚本:当sentinel有任何警告级别的事件发生时（比如说redis实例的主观失效和客观失效等等），将会去调用这个脚本，
# 这时这个脚本应该通过邮件，SMS等方式去通知系统管理员关于系统不正常运行的信息。调用该脚本时，将传给脚本两个参数，一个是事件# 的类型，一个是事件的描述。
# 如果sentinel.conf配置文件中配置了这个脚本路径，那么必须保证这个脚本存在于这个路径，并且是可执行的，否则sentinel无法正
# 常启动成功。
#
# sentinel notification-script mymaster /var/redis/notify.sh

# CLIENTS RECONFIGURATION SCRIPT
#
# sentinel client-reconfig-script <master-name> <script-path>
#
# When the master changed because of a failover a script can be called in
# order to perform application-specific tasks to notify the clients that the
# configuration has changed and the master is at a different address.
# 
# The following arguments are passed to the script:
#
# <master-name> <role> <state> <from-ip> <from-port> <to-ip> <to-port>
#
# <state> is currently always "start"
# <role> is either "leader" or "observer"
# 
# The arguments from-ip, from-port, to-ip, to-port are used to communicate
# the old address of the master and the new address of the elected replica
# (now a master).
#
# This script should be resistant to multiple invocations.
#
# Example:
#
# 客户端重新配置主节点参数脚本
# 当一个master由于failover而发生改变时，这个脚本将会被调用，通知相关的客户端关于master地址已经发生改变的信息。
# 以下参数将会在调用脚本时传给脚本:
# <master-name> <role> <state> <from-ip> <from-port> <to-ip> <to-port>
# 目前<state>总是“failover”,
# <role>是“leader”或者“observer”中的一个。 
# 参数 from-ip, from-port, to-ip, to-port是用来和旧的master和新的master(即旧的slave)通信的
# 这个脚本应该是通用的，能被多次调用，不是针对性的。
#
# sentinel client-reconfig-script mymaster /var/redis/reconfig.sh

# SECURITY
#
# By default SENTINEL SET will not be able to change the notification-script
# and client-reconfig-script at runtime. This avoids a trivial security issue
# where clients can set the script to anything and trigger a failover in order
# to get the program executed.

sentinel deny-scripts-reconfig yes

# REDIS COMMANDS RENAMING (DEPRECATED)
#
# WARNING: avoid using this option if possible, instead use ACLs.
#
# Sometimes the Redis server has certain commands, that are needed for Sentinel
# to work correctly, renamed to unguessable strings. This is often the case
# of CONFIG and SLAVEOF in the context of providers that provide Redis as
# a service, and don't want the customers to reconfigure the instances outside
# of the administration console.
#
# In such case it is possible to tell Sentinel to use different command names
# instead of the normal ones. For example if the master "mymaster", and the
# associated replicas, have "CONFIG" all renamed to "GUESSME", I could use:
#
# SENTINEL rename-command mymaster CONFIG GUESSME
#
# After such configuration is set, every time Sentinel would use CONFIG it will
# use GUESSME instead. Note that there is no actual need to respect the command
# case, so writing "config guessme" is the same in the example above.
#
# SENTINEL SET can also be used in order to perform this configuration at runtime.
#
# In order to set a command back to its original name (undo the renaming), it
# is possible to just rename a command to itself:
#
# SENTINEL rename-command mymaster CONFIG CONFIG

# HOSTNAMES SUPPORT
#
# Normally Sentinel uses only IP addresses and requires SENTINEL MONITOR
# to specify an IP address. Also, it requires the Redis replica-announce-ip
# keyword to specify only IP addresses.
#
# You may enable hostnames support by enabling resolve-hostnames. Note
# that you must make sure your DNS is configured properly and that DNS
# resolution does not introduce very long delays.
#
SENTINEL resolve-hostnames no

# When resolve-hostnames is enabled, Sentinel still uses IP addresses
# when exposing instances to users, configuration files, etc. If you want
# to retain the hostnames when announced, enable announce-hostnames below.
#
SENTINEL announce-hostnames no

# When master_reboot_down_after_period is set to 0, Sentinel does not fail over
# when receiving a -LOADING response from a master. This was the only supported
# behavior before version 7.0.
#
# Otherwise, Sentinel will use this value as the time (in ms) it is willing to
# accept a -LOADING response after a master has been rebooted, before failing
# over.

SENTINEL master-reboot-down-after-period mymaster 0

```

启动哨兵```redis-sentinel ./sentinel.conf```

如果主服务器宕机后重新连接，只能作为从服务器使用，因为主服务器已经选出

**优点**

1. 哨兵集群，基于主从复制，所以包含了主从复制的优点
2. 主从可以自动切换，系统可用性更好
3. 哨兵模式就是主从模式的升级，手动到自动，更加健壮

**缺点**

1. redis不好在线扩容，集群容量一旦到达上限，在线扩容十分麻烦
2. 实现哨兵模式的配置非常繁琐麻烦

## Redis缓存穿透和雪崩

### 缓存穿透

#### 概念

缓存穿透是指用户查询的数据在redis缓存中不存在，然后向数据库中查询也没有查到数据导致查询失败。这种现象发生很多次导致给数据库增加很大压力，这就是缓存穿透。比如在秒杀中，请求量剧增但是缓存没有命中导致服务器压力剧增。

#### 解决方案

1. 布隆过滤器

    布隆过滤器是一种数据结构，对所有可能查询的参数以hash形式存储，在控制层先进行校验，不符合就丢弃，避免了对底层数据库造成压力。

    ![image18](../../img/2022/11/redis_note/image18.png)

2. 缓存空对象

    当数据库没有命中时，将空对象加到缓存中，同时设置一个过期时间，如果用户再查这个对象就会命中缓存，保护了底层数据库。

    但是这种方法会存在两个问题：

    1. 存储空值没有意义，并且会有越来越多没有意义的键
    2. 即使对空值设置了过期时间，如果原来没有值的对象被存到了数据库中就会导致一段时间缓存的值和数据库中的数据不一致的问题。

    ![image19](../../img/2022/11/redis_note/image19.png)

    ### 缓存击穿

    #### 概念

    如果缓存中的某一个key非常热点，有大量的请求命中这个key，那么在这个key失效的瞬间，这些请求就会访问数据库，给数据库带来巨大的压力。

    #### 解决方案

    1. 设置热点数据不过期

        不过期就会一直命中，但是可能会浪费空间

    2. 加互斥锁

        使用分布式锁，保证对于每个key同时只有一个线程去查询数据库，其他的线程没有获得分布式锁的权限，因此需要等待。这种方式将高并发的压力转移到了分布式锁上，对分布式锁的考验很大。

    ### 缓存雪崩

    #### 概念

    缓存雪崩是指在某一个时间段缓存集中失效，或者是redis服务器宕机。原因是某些热点数据在同一时间被加入到缓存中并且过期时间也一致，这样就导致了在同一时间失效。服务器宕机带来的雪崩，其访问量是不可知的，所以有可能给数据库带来巨大的压力。

    ![image20](../../img/2022/11/redis_note/image20.png)

    #### 解决方案

    1. redis高可用

        增加redis服务器，搭建集群，高可用。

    2. 限流降级

        在缓存失效后，通过加锁或者队列来控制读数据库的线程数量，比如对同一个key只允许一个线程查询数据并写入缓存，其他线程只能等待。或者是将一些无关的服务停掉

    3. 数据预热

        在正式部署之前，将可能的热点数据全部访问一遍，使这些数据写入缓存。在即将发生大量并发请求前手动触发加载缓存，设置不同的过期时间，就能防止在某一时间同时失效的问题。

