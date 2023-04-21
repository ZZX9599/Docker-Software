# 使用Docker安装常用软件

# 0:前提准备

## Linux环境

常用的Linux环境我这里就不准备了，我们配置了Tomcat和JDK

![image-20230131224644899](https://zzx-note.oss-cn-beijing.aliyuncs.com/docker/image-20230131224644899.png)

## 安装宝塔

```sh
yum install -y wget && wget -O install.sh http://download.bt.cn/install/install_6.0.sh && sh install.sh
```

## 卸载(可选)

如果之前安装过旧版本的Docker，可以使用下面命令卸载：

```sh
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine \
                  docker-ce
```



## 安装docker

首先需要大家虚拟机联网，安装yum工具

```sh
yum install -y yum-utils \
           device-mapper-persistent-data \
           lvm2 --skip-broken
```



然后更新本地镜像源：

```shell
yum-config-manager \
    --add-repo \
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
    sed -i 's/download.docker.com/mirrors.aliyun.com\/docker-ce/g' /etc/yum.repos.d/docker-ce.repo

yum makecache fast
```



然后输入命令：

```shell
yum install -y docker-ce
```

docker-ce为社区免费版本。稍等片刻，docker即可安装成功



## 启动docker

Docker应用需要用到各种端口，逐一去修改防火墙设置。非常麻烦，因此建议大家直接关闭防火墙！

启动docker前，一定要关闭防火墙后！！

启动docker前，一定要关闭防火墙后！！

启动docker前，一定要关闭防火墙后！！



```sh
# 关闭
systemctl stop firewalld
# 禁止开机启动防火墙
systemctl disable firewalld
```



通过命令启动docker：

```sh
systemctl start docker  # 启动docker服务

systemctl stop docker  # 停止docker服务

systemctl restart docker  # 重启docker服务
```



然后输入命令，可以查看docker版本：

```
docker -v
```

## 镜像加速

阿里云镜像获取地址：https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors

## 后续出问题

后续安装出现问题，记得使用 

```sh
docker logs -f 容器名
```

 来查看问题的原因

```sh
docker run -i -t -d --name baota \
 -p 20:20 -p 21:21 -p 90:80 -p 443:443 -p 888:888 -p 8888:8888 \
 --privileged=true \
 -v /home/www:/www centos:7.2.1511
```



## 安装DockerCompose

下载地址：

https://github.com/docker/compose/releases

![image-20230214103951776](https://zzx-note.oss-cn-beijing.aliyuncs.com/docker/image-20230214103951776.png)

下载之后改名为docker-compose

mv docker-compose-linux-x86_64 docker-compose



移动到/user/local/bin下面

添加权限

sudo chmod +x /usr/local/bin/docker-compose



查看版本

docker-compose version

# 1:安装Mysql

我这里以安装8.0.26的版本为例

## 1.1:拉取镜像

docker pull mysql:8.0.26

## 1.2:新建挂载目录

mkdir -p /home/docker/mysql/{data,conf}

进入/home/docker/mysql/conf目录：cd /home/docker/mysql/conf

## 1.3:创建配置文件

touch my.cnf

配置文件内容如下：直接复制即可！

```sh
[mysqld]
#Mysql服务的唯一编号 每个mysql服务Id需唯一
server-id=1

#服务端口号 默认3306
port=3306

#mysql安装根目录（default /usr）
#basedir=/usr/local/mysql

#mysql数据文件所在位置
datadir=/var/lib/mysql

#pid
pid-file=/var/run/mysqld/mysqld.pid

#设置socke文件所在目录
socket=/var/lib/mysql/mysql.sock

#设置临时目录
#tmpdir=/tmp

# 用户
user=mysql

# 允许访问的IP网段
bind-address=0.0.0.0

# 跳过密码登录
#skip-grant-tables

#主要用于MyISAM存储引擎,如果多台服务器连接一个数据库则建议注释下面内容
#skip-external-locking

#只能用IP地址检查客户端的登录，不用主机名
#skip_name_resolve=1

#事务隔离级别，默认为可重复读，mysql默认可重复读级别（此级别下可能参数很多间隙锁，影响性能）
#transaction_isolation=READ-COMMITTED

#数据库默认字符集,主流字符集支持一些特殊表情符号（特殊表情符占用4个字节）
character-set-server=utf8mb4

#数据库字符集对应一些排序等规则，注意要和character-set-server对应
collation-server=utf8mb4_general_ci

#设置client连接mysql时的字符集,防止乱码
init_connect='SET NAMES utf8mb4'

#是否对sql语句大小写敏感，1表示不敏感
lower_case_table_names=1

#最大连接数
max_connections=400

#最大错误连接数
max_connect_errors=1000

#TIMESTAMP如果没有显示声明NOT NULL，允许NULL值
explicit_defaults_for_timestamp=true

#SQL数据包发送的大小，如果有BLOB对象建议修改成1G
max_allowed_packet=128M

#MySQL连接闲置超过一定时间后(单位：秒)将会被强行关闭
#MySQL默认的wait_timeout  值为8个小时, interactive_timeout参数需要同时配置才能生效
interactive_timeout=1800
wait_timeout=1800

#内部内存临时表的最大值 ，设置成128M。
#比如大数据量的group by ,order by时可能用到临时表，
#超过了这个值将写入磁盘，系统IO压力增大
tmp_table_size=134217728
max_heap_table_size=134217728

#禁用mysql的缓存查询结果集功能
#后期根据业务情况测试决定是否开启
#大部分情况下关闭下面两项
#query_cache_size = 0
#query_cache_type = 0
 
#数据库错误日志文件
#log-error=/var/log/mysqld.log

#慢查询sql日志设置
#slow_query_log=1
#slow_query_log_file=/var/log/mysqld_slow.log

#检查未使用到索引的sql
log_queries_not_using_indexes=1

#针对log_queries_not_using_indexes开启后，记录慢sql的频次、每分钟记录的条数
log_throttle_queries_not_using_indexes=5

#作为从库时生效,从库复制中如何有慢sql也将被记录
log_slow_slave_statements=1

#慢查询执行的秒数，必须达到此值可被记录
long_query_time=8

#检索的行数必须达到此值才可被记为慢查询
min_examined_row_limit=100

#mysql binlog日志文件保存的过期时间，过期后自动删除
#expire_logs_days=5
binlog_expire_logs_seconds=604800
```

## 1.4:创建容器

```sh
docker run -itd -p 3306:3306  \
 --name mysql \
 -v /home/docker/mysql/conf/my.cnf:/etc/my.cnf \
 -v /home/docker/mysql/data:/var/lib/mysql\
 --privileged=true \
 -e MYSQL_ROOT_PASSWORD=123456 \
 -d mysql:8.0.26
```

![image-20230129104908653](https://zzx-note.oss-cn-beijing.aliyuncs.com/docker/image-20230129104908653.png)

也可以不加那三个参数

```sh
docker run -p 3306:3306  \
 --name mysql \
 -v /home/docker/mysql/conf/my.cnf:/etc/my.cnf \
 -v /home/docker/mysql/data:/var/lib/mysql  \
 --privileged=true \
 -e MYSQL_ROOT_PASSWORD=root \
 -d mysql:8.0.26
```

--restart always：自启动

# 2:安装Redis

## 2.1:拉取镜像

docker pull redis:6.2.6

## 2.2:新建挂载目录

mkdir -p /home/docker/redis/{data,conf}

进入/home/docker/redis/conf目录：cd /home/docker/redis/conf

## 2.3:创建配置文件

touch redis.conf

配置文件内容如下：直接复制即可！

```sh
# Redis configuration file example.

#

# Note that in order to read the configuration file, Redis must be

# started with the file path as first argument:

#

# ./redis-server /path/to/redis.conf

 

# Note on units: when memory size is needed, it is possible to specify

# it in the usual form of 1k 5GB 4M and so forth:

#

# 1k => 1000 bytes

# 1kb => 1024 bytes

# 1m => 1000000 bytes

# 1mb => 1024*1024 bytes

# 1g => 1000000000 bytes

# 1gb => 1024*1024*1024 bytes

#

# units are case insensitive so 1GB 1Gb 1gB are all the same.

 

################################## INCLUDES ###################################

 

# Include one or more other config files here.  This is useful if you

# have a standard template that goes to all Redis servers but also need

# to customize a few per-server settings.  Include files can include

# other files, so use this wisely.

#

# Notice option "include" won't be rewritten by command "CONFIG REWRITE"

# from admin or Redis Sentinel. Since Redis always uses the last processed

# line as value of a configuration directive, you'd better put includes

# at the beginning of this file to avoid overwriting config change at runtime.

#

# If instead you are interested in using includes to override configuration

# options, it is better to use include as the last line.

#

# include /path/to/local.conf

# include /path/to/other.conf

 

################################## MODULES #####################################

 

# Load modules at startup. If the server is not able to load modules

# it will abort. It is possible to use multiple loadmodule directives.

#

# loadmodule /path/to/my_module.so

# loadmodule /path/to/other_module.so

 

################################## NETWORK #####################################

 

# By default, if no "bind" configuration directive is specified, Redis listens

# for connections from all the network interfaces available on the server.

# It is possible to listen to just one or multiple selected interfaces using

# the "bind" configuration directive, followed by one or more IP addresses.

#

# Examples:

#

# bind 192.168.1.100 10.0.0.1

# bind 127.0.0.1 ::1

#

# ~~~ WARNING ~~~ If the computer running Redis is directly exposed to the

# internet, binding to all the interfaces is dangerous and will expose the

# instance to everybody on the internet. So by default we uncomment the

# following bind directive, that will force Redis to listen only into

# the IPv4 loopback interface address (this means Redis will be able to

# accept connections only from clients running into the same computer it

# is running).

#

# IF YOU ARE SURE YOU WANT YOUR INSTANCE TO LISTEN TO ALL THE INTERFACES

# JUST COMMENT THE FOLLOWING LINE.

# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#bind 127.0.0.1

 

# Protected mode is a layer of security protection, in order to avoid that

# Redis instances left open on the internet are accessed and exploited.

#

# When protected mode is on and if:

#

# 1) The server is not binding explicitly to a set of addresses using the

#    "bind" directive.

# 2) No password is configured.

#

# The server only accepts connections from clients connecting from the

# IPv4 and IPv6 loopback addresses 127.0.0.1 and ::1, and from Unix domain

# sockets.

#

# By default protected mode is enabled. You should disable it only if

# you are sure you want clients from other hosts to connect to Redis

# even if no authentication is configured, nor a specific set of interfaces

# are explicitly listed using the "bind" directive.

protected-mode no

 

# Accept connections on the specified port, default is 6379 (IANA #815344).

# If port 0 is specified Redis will not listen on a TCP socket.

port 6379

 

# TCP listen() backlog.

#

# In high requests-per-second environments you need an high backlog in order

# to avoid slow clients connections issues. Note that the Linux kernel

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

# unixsocket /tmp/redis.sock

# unixsocketperm 700

 

# Close the connection after a client is idle for N seconds (0 to disable)

timeout 0

 

# TCP keepalive.

#

# If non-zero, use SO_KEEPALIVE to send TCP ACKs to clients in absence

# of communication. This is useful for two reasons:

#

# 1) Detect dead peers.

# 2) Take the connection alive from the point of view of network

#    equipment in the middle.

#

# On Linux, the specified value (in seconds) is the period used to send ACKs.

# Note that to close the connection the double of the time is needed.

# On other kernels the period depends on the kernel configuration.

#

# A reasonable value for this option is 300 seconds, which is the new

# Redis default starting with Redis 3.2.1.

tcp-keepalive 300

 

################################# GENERAL #####################################

 

# By default Redis does not run as a daemon. Use 'yes' if you need it.

# Note that Redis will write a pid file in /var/run/redis.pid when daemonized.

daemonize no

 

# If you run Redis from upstart or systemd, Redis can interact with your

# supervision tree. Options:

#   supervised no      - no supervision interaction

#   supervised upstart - signal upstart by putting Redis into SIGSTOP mode

#   supervised systemd - signal systemd by writing READY=1 to $NOTIFY_SOCKET

#   supervised auto    - detect upstart or systemd method based on

#                        UPSTART_JOB or NOTIFY_SOCKET environment variables

# Note: these supervision methods only signal "process is ready."

#       They do not enable continuous liveness pings back to your supervisor.

supervised no

 

# If a pid file is specified, Redis writes it where specified at startup

# and removes it at exit.

#

# When the server runs non daemonized, no pid file is created if none is

# specified in the configuration. When the server is daemonized, the pid file

# is used even if not specified, defaulting to "/var/run/redis.pid".

#

# Creating a pid file is best effort: if Redis is not able to create it

# nothing bad happens, the server will start and run normally.

pidfile /var/run/redis_6379.pid

 

# Specify the server verbosity level.

# This can be one of:

# debug (a lot of information, useful for development/testing)

# verbose (many rarely useful info, but not a mess like the debug level)

# notice (moderately verbose, what you want in production probably)

# warning (only very important / critical messages are logged)

loglevel notice

 

# Specify the log file name. Also the empty string can be used to force

# Redis to log on the standard output. Note that if you use standard

# output for logging but daemonize, logs will be sent to /dev/null

logfile ""

 

# To enable logging to the system logger, just set 'syslog-enabled' to yes,

# and optionally update the other syslog parameters to suit your needs.

# syslog-enabled no

 

# Specify the syslog identity.

# syslog-ident redis

 

# Specify the syslog facility. Must be USER or between LOCAL0-LOCAL7.

# syslog-facility local0

 

# Set the number of databases. The default database is DB 0, you can select

# a different one on a per-connection basis using SELECT <dbid> where

# dbid is a number between 0 and 'databases'-1

databases 16

 

# By default Redis shows an ASCII art logo only when started to log to the

# standard output and if the standard output is a TTY. Basically this means

# that normally a logo is displayed only in interactive sessions.

#

# However it is possible to force the pre-4.0 behavior and always show a

# ASCII art logo in startup logs by setting the following option to yes.

always-show-logo yes

 

################################ SNAPSHOTTING  ################################

#

# Save the DB on disk:

#

#   save <seconds> <changes>

#

#   Will save the DB if both the given number of seconds and the given

#   number of write operations against the DB occurred.

#

#   In the example below the behaviour will be to save:

#   after 900 sec (15 min) if at least 1 key changed

#   after 300 sec (5 min) if at least 10 keys changed

#   after 60 sec if at least 10000 keys changed

#

#   Note: you can disable saving completely by commenting out all "save" lines.

#

#   It is also possible to remove all the previously configured save

#   points by adding a save directive with a single empty string argument

#   like in the following example:

#

#   save ""

 

save 900 1

save 300 10

save 60 10000

 

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

stop-writes-on-bgsave-error yes

 

# Compress string objects using LZF when dump .rdb databases?

# For default that's set to 'yes' as it's almost always a win.

# If you want to save some CPU in the saving child set it to 'no' but

# the dataset will likely be bigger if you have compressible values or keys.

rdbcompression yes

 

# Since version 5 of RDB a CRC64 checksum is placed at the end of the file.

# This makes the format more resistant to corruption but there is a performance

# hit to pay (around 10%) when saving and loading RDB files, so you can disable it

# for maximum performances.

#

# RDB files created with checksum disabled have a checksum of zero that will

# tell the loading code to skip the check.

rdbchecksum yes

 

# The filename where to dump the DB

dbfilename dump.rdb

 

# The working directory.

#

# The DB will be written inside this directory, with the filename specified

# above using the 'dbfilename' configuration directive.

#

# The Append Only File will also be created inside this directory.

#

# Note that you must specify a directory here, not a file name.

dir ./

 

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

# replicaof <masterip> <masterport>

 

# If the master is password protected (using the "requirepass" configuration

# directive below) it is possible to tell the replica to authenticate before

# starting the replication synchronization process, otherwise the master will

# refuse the replica request.

#

# masterauth <master-password>

 

# When a replica loses its connection with the master, or when the replication

# is still in progress, the replica can act in two different ways:

#

# 1) if replica-serve-stale-data is set to 'yes' (the default) the replica will

#    still reply to client requests, possibly with out of date data, or the

#    data set may just be empty if this is the first synchronization.

#

# 2) if replica-serve-stale-data is set to 'no' the replica will reply with

#    an error "SYNC with master in progress" to all the kind of commands

#    but to INFO, replicaOF, AUTH, PING, SHUTDOWN, REPLCONF, ROLE, CONFIG,

#    SUBSCRIBE, UNSUBSCRIBE, PSUBSCRIBE, PUNSUBSCRIBE, PUBLISH, PUBSUB,

#    COMMAND, POST, HOST: and LATENCY.

#

replica-serve-stale-data yes

 

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

replica-read-only yes

 

# Replication SYNC strategy: disk or socket.

#

# -------------------------------------------------------

# WARNING: DISKLESS REPLICATION IS EXPERIMENTAL CURRENTLY

# -------------------------------------------------------

#

# New replicas and reconnecting replicas that are not able to continue the replication

# process just receiving differences, need to do what is called a "full

# synchronization". An RDB file is transmitted from the master to the replicas.

# The transmission can happen in two different ways:

#

# 1) Disk-backed: The Redis master creates a new process that writes the RDB

#                 file on disk. Later the file is transferred by the parent

#                 process to the replicas incrementally.

# 2) Diskless: The Redis master creates a new process that directly writes the

#              RDB file to replica sockets, without touching the disk at all.

#

# With disk-backed replication, while the RDB file is generated, more replicas

# can be queued and served with the RDB file as soon as the current child producing

# the RDB file finishes its work. With diskless replication instead once

# the transfer starts, new replicas arriving will be queued and a new transfer

# will start when the current one terminates.

#

# When diskless replication is used, the master waits a configurable amount of

# time (in seconds) before starting the transfer in the hope that multiple replicas

# will arrive and the transfer can be parallelized.

#

# With slow disks and fast (large bandwidth) networks, diskless replication

# works better.

repl-diskless-sync no

 

# When diskless replication is enabled, it is possible to configure the delay

# the server waits in order to spawn the child that transfers the RDB via socket

# to the replicas.

#

# This is important since once the transfer starts, it is not possible to serve

# new replicas arriving, that will be queued for the next RDB transfer, so the server

# waits a delay in order to let more replicas arrive.

#

# The delay is specified in seconds, and by default is 5 seconds. To disable

# it entirely just set it to 0 seconds and the transfer will start ASAP.

repl-diskless-sync-delay 5

 

# Replicas send PINGs to server in a predefined interval. It's possible to change

# this interval with the repl_ping_replica_period option. The default value is 10

# seconds.

#

# repl-ping-replica-period 10

 

# The following option sets the replication timeout for:

#

# 1) Bulk transfer I/O during SYNC, from the point of view of replica.

# 2) Master timeout from the point of view of replicas (data, pings).

# 3) Replica timeout from the point of view of masters (REPLCONF ACK pings).

#

# It is important to make sure that this value is greater than the value

# specified for repl-ping-replica-period otherwise a timeout will be detected

# every time there is low traffic between the master and the replica.

#

# repl-timeout 60

 

# Disable TCP_NODELAY on the replica socket after SYNC?

#

# If you select "yes" Redis will use a smaller number of TCP packets and

# less bandwidth to send data to replicas. But this can add a delay for

# the data to appear on the replica side, up to 40 milliseconds with

# Linux kernels using a default configuration.

#

# If you select "no" the delay for data to appear on the replica side will

# be reduced but more bandwidth will be used for replication.

#

# By default we optimize for low latency, but in very high traffic conditions

# or when the master and replicas are many hops away, turning this to "yes" may

# be a good idea.

repl-disable-tcp-nodelay no

 

# Set the replication backlog size. The backlog is a buffer that accumulates

# replica data when replicas are disconnected for some time, so that when a replica

# wants to reconnect again, often a full resync is not needed, but a partial

# resync is enough, just passing the portion of data the replica missed while

# disconnected.

#

# The bigger the replication backlog, the longer the time the replica can be

# disconnected and later be able to perform a partial resynchronization.

#

# The backlog is only allocated once there is at least a replica connected.

#

# repl-backlog-size 1mb

 

# After a master has no longer connected replicas for some time, the backlog

# will be freed. The following option configures the amount of seconds that

# need to elapse, starting from the time the last replica disconnected, for

# the backlog buffer to be freed.

#

# Note that replicas never free the backlog for timeout, since they may be

# promoted to masters later, and should be able to correctly "partially

# resynchronize" with the replicas: hence they should always accumulate backlog.

#

# A value of 0 means to never release the backlog.

#

# repl-backlog-ttl 3600

 

# The replica priority is an integer number published by Redis in the INFO output.

# It is used by Redis Sentinel in order to select a replica to promote into a

# master if the master is no longer working correctly.

#

# A replica with a low priority number is considered better for promotion, so

# for instance if there are three replicas with priority 10, 100, 25 Sentinel will

# pick the one with priority 10, that is the lowest.

#

# However a special priority of 0 marks the replica as not able to perform the

# role of master, so a replica with priority of 0 will never be selected by

# Redis Sentinel for promotion.

#

# By default the priority is 100.

replica-priority 100

 

# It is possible for a master to stop accepting writes if there are less than

# N replicas connected, having a lag less or equal than M seconds.

#

# The N replicas need to be in "online" state.

#

# The lag in seconds, that must be <= the specified value, is calculated from

# the last ping received from the replica, that is usually sent every second.

#

# This option does not GUARANTEE that N replicas will accept the write, but

# will limit the window of exposure for lost writes in case not enough replicas

# are available, to the specified number of seconds.

#

# For example to require at least 3 replicas with a lag <= 10 seconds use:

#

# min-replicas-to-write 3

# min-replicas-max-lag 10

#

# Setting one or the other to 0 disables the feature.

#

# By default min-replicas-to-write is set to 0 (feature disabled) and

# min-replicas-max-lag is set to 10.

 

# A Redis master is able to list the address and port of the attached

# replicas in different ways. For example the "INFO replication" section

# offers this information, which is used, among other tools, by

# Redis Sentinel in order to discover replica instances.

# Another place where this info is available is in the output of the

# "ROLE" command of a master.

#

# The listed IP and address normally reported by a replica is obtained

# in the following way:

#

#   IP: The address is auto detected by checking the peer address

#   of the socket used by the replica to connect with the master.

#

#   Port: The port is communicated by the replica during the replication

#   handshake, and is normally the port that the replica is using to

#   listen for connections.

#

# However when port forwarding or Network Address Translation (NAT) is

# used, the replica may be actually reachable via different IP and port

# pairs. The following two options can be used by a replica in order to

# report to its master a specific set of IP and port, so that both INFO

# and ROLE will report those values.

#

# There is no need to use both the options if you need to override just

# the port or the IP address.

#

# replica-announce-ip 5.5.5.5

# replica-announce-port 1234

 

################################## SECURITY ###################################

 

# Require clients to issue AUTH <PASSWORD> before processing any other

# commands.  This might be useful in environments in which you do not trust

# others with access to the host running redis-server.

#

# This should stay commented out for backward compatibility and because most

# people do not need auth (e.g. they run their own servers).

#

# Warning: since Redis is pretty fast an outside user can try up to

# 150k passwords per second against a good box. This means that you should

# use a very strong password otherwise it will be very easy to break.

#

# requirepass foobared

 

# Command renaming.

#

# It is possible to change the name of dangerous commands in a shared

# environment. For instance the CONFIG command may be renamed into something

# hard to guess so that it will still be available for internal-use tools

# but not available for general clients.

#

# Example:

#

# rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52

#

# It is also possible to completely kill a command by renaming it into

# an empty string:

#

# rename-command CONFIG ""

#

# Please note that changing the name of commands that are logged into the

# AOF file or transmitted to replicas may cause problems.

 

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

# maxclients 10000

 

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

# maxmemory <bytes>

 

# MAXMEMORY POLICY: how Redis will select what to remove when maxmemory

# is reached. You can select among five behaviors:

#

# volatile-lru -> Evict using approximated LRU among the keys with an expire set.

# allkeys-lru -> Evict any key using approximated LRU.

# volatile-lfu -> Evict using approximated LFU among the keys with an expire set.

# allkeys-lfu -> Evict any key using approximated LFU.

# volatile-random -> Remove a random key among the ones with an expire set.

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

# Note: with any of the above policies, Redis will return an error on write

#       operations, when there are no suitable keys for eviction.

#

#       At the date of writing these commands are: set setnx setex append

#       incr decr rpush lpush rpushx lpushx linsert lset rpoplpush sadd

#       sinter sinterstore sunion sunionstore sdiff sdiffstore zadd zincrby

#       zunionstore zinterstore hset hsetnx hmset hincrby incrby decrby

#       getset mset msetnx exec sort

#

# The default is:

#

# maxmemory-policy noeviction

 

# LRU, LFU and minimal TTL algorithms are not precise algorithms but approximated

# algorithms (in order to save memory), so you can tune it for speed or

# accuracy. For default Redis will check five keys and pick the one that was

# used less recently, you can change the sample size using the following

# configuration directive.

#

# The default of 5 produces good enough results. 10 Approximates very closely

# true LRU but costs more CPU. 3 is faster but not very accurate.

#

# maxmemory-samples 5

 

# Starting from Redis 5, by default a replica will ignore its maxmemory setting

# (unless it is promoted to master after a failover or manually). It means

# that the eviction of keys will be just handled by the master, sending the

# DEL commands to the replica as keys evict in the master side.

#

# This behavior ensures that masters and replicas stay consistent, and is usually

# what you want, however if your replica is writable, or you want the replica to have

# a different memory setting, and you are sure all the writes performed to the

# replica are idempotent, then you may change this default (but be sure to understand

# what you are doing).

#

# Note that since the replica by default does not evict, it may end using more

# memory than the one set via maxmemory (there are certain buffers that may

# be larger on the replica, or data structures may sometimes take more memory and so

# forth). So make sure you monitor your replicas and make sure they have enough

# memory to never hit a real out-of-memory condition before the master hits

# the configured maxmemory setting.

#

# replica-ignore-maxmemory yes

 

############################# LAZY FREEING ####################################

 

# Redis has two primitives to delete keys. One is called DEL and is a blocking

# deletion of the object. It means that the server stops processing new commands

# in order to reclaim all the memory associated with an object in a synchronous

# way. If the key deleted is associated with a small object, the time needed

# in order to execute the DEL command is very small and comparable to most other

# O(1) or O(log_N) commands in Redis. However if the key is associated with an

# aggregated value containing millions of elements, the server can block for

# a long time (even seconds) in order to complete the operation.

#

# For the above reasons Redis also offers non blocking deletion primitives

# such as UNLINK (non blocking DEL) and the ASYNC option of FLUSHALL and

# FLUSHDB commands, in order to reclaim memory in background. Those commands

# are executed in constant time. Another thread will incrementally free the

# object in the background as fast as possible.

#

# DEL, UNLINK and ASYNC option of FLUSHALL and FLUSHDB are user-controlled.

# It's up to the design of the application to understand when it is a good

# idea to use one or the other. However the Redis server sometimes has to

# delete keys or flush the whole database as a side effect of other operations.

# Specifically Redis deletes objects independently of a user call in the

# following scenarios:

#

# 1) On eviction, because of the maxmemory and maxmemory policy configurations,

#    in order to make room for new data, without going over the specified

#    memory limit.

# 2) Because of expire: when a key with an associated time to live (see the

#    EXPIRE command) must be deleted from memory.

# 3) Because of a side effect of a command that stores data on a key that may

#    already exist. For example the RENAME command may delete the old key

#    content when it is replaced with another one. Similarly SUNIONSTORE

#    or SORT with STORE option may delete existing keys. The SET command

#    itself removes any old content of the specified key in order to replace

#    it with the specified string.

# 4) During replication, when a replica performs a full resynchronization with

#    its master, the content of the whole database is removed in order to

#    load the RDB file just transferred.

#

# In all the above cases the default is to delete objects in a blocking way,

# like if DEL was called. However you can configure each case specifically

# in order to instead release memory in a non-blocking way like if UNLINK

# was called, using the following configuration directives:

 

lazyfree-lazy-eviction no

lazyfree-lazy-expire no

lazyfree-lazy-server-del no

replica-lazy-flush no

 

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

# Please check http://redis.io/topics/persistence for more information.

 

appendonly no

 

# The name of the append only file (default: "appendonly.aof")

 

appendfilename "appendonly.aof"

 

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

 

# appendfsync always

appendfsync everysec

# appendfsync no

 

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

# the same as "appendfsync none". In practical terms, this means that it is

# possible to lose up to 30 seconds of log in the worst scenario (with the

# default Linux settings).

#

# If you have latency problems turn this to "yes". Otherwise leave it as

# "no" that is the safest pick from the point of view of durability.

 

no-appendfsync-on-rewrite no

 

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

 

auto-aof-rewrite-percentage 100

auto-aof-rewrite-min-size 64mb

 

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

 

# When rewriting the AOF file, Redis is able to use an RDB preamble in the

# AOF file for faster rewrites and recoveries. When this option is turned

# on the rewritten AOF file is composed of two different stanzas:

#

#   [RDB file][AOF tail]

#

# When loading Redis recognizes that the AOF file starts with the "REDIS"

# string and loads the prefixed RDB file, and continues loading the AOF

# tail.

aof-use-rdb-preamble yes

 

################################ LUA SCRIPTING  ###############################

 

# Max execution time of a Lua script in milliseconds.

#

# If the maximum execution time is reached Redis will log that a script is

# still in execution after the maximum allowed time and will start to

# reply to queries with an error.

#

# When a long running script exceeds the maximum execution time only the

# SCRIPT KILL and SHUTDOWN NOSAVE commands are available. The first can be

# used to stop a script that did not yet called write commands. The second

# is the only way to shut down the server in the case a write command was

# already issued by the script but the user doesn't want to wait for the natural

# termination of the script.

#

# Set it to 0 or a negative value for unlimited execution without warnings.

lua-time-limit 5000

 

################################ REDIS CLUSTER  ###############################

 

# Normal Redis instances can't be part of a Redis Cluster; only nodes that are

# started as cluster nodes can. In order to start a Redis instance as a

# cluster node enable the cluster support uncommenting the following:

#

# cluster-enabled yes

 

# Every cluster node has a cluster configuration file. This file is not

# intended to be edited by hand. It is created and updated by Redis nodes.

# Every Redis Cluster node requires a different cluster configuration file.

# Make sure that instances running in the same system do not have

# overlapping cluster configuration file names.

#

# cluster-config-file nodes-6379.conf

 

# Cluster node timeout is the amount of milliseconds a node must be unreachable

# for it to be considered in failure state.

# Most other internal time limits are multiple of the node timeout.

#

# cluster-node-timeout 15000

 

# A replica of a failing master will avoid to start a failover if its data

# looks too old.

#

# There is no simple way for a replica to actually have an exact measure of

# its "data age", so the following two checks are performed:

#

# 1) If there are multiple replicas able to failover, they exchange messages

#    in order to try to give an advantage to the replica with the best

#    replication offset (more data from the master processed).

#    Replicas will try to get their rank by offset, and apply to the start

#    of the failover a delay proportional to their rank.

#

# 2) Every single replica computes the time of the last interaction with

#    its master. This can be the last ping or command received (if the master

#    is still in the "connected" state), or the time that elapsed since the

#    disconnection with the master (if the replication link is currently down).

#    If the last interaction is too old, the replica will not try to failover

#    at all.

#

# The point "2" can be tuned by user. Specifically a replica will not perform

# the failover if, since the last interaction with the master, the time

# elapsed is greater than:

#

#   (node-timeout * replica-validity-factor) + repl-ping-replica-period

#

# So for example if node-timeout is 30 seconds, and the replica-validity-factor

# is 10, and assuming a default repl-ping-replica-period of 10 seconds, the

# replica will not try to failover if it was not able to talk with the master

# for longer than 310 seconds.

#

# A large replica-validity-factor may allow replicas with too old data to failover

# a master, while a too small value may prevent the cluster from being able to

# elect a replica at all.

#

# For maximum availability, it is possible to set the replica-validity-factor

# to a value of 0, which means, that replicas will always try to failover the

# master regardless of the last time they interacted with the master.

# (However they'll always try to apply a delay proportional to their

# offset rank).

#

# Zero is the only value able to guarantee that when all the partitions heal

# the cluster will always be able to continue.

#

# cluster-replica-validity-factor 10

 

# Cluster replicas are able to migrate to orphaned masters, that are masters

# that are left without working replicas. This improves the cluster ability

# to resist to failures as otherwise an orphaned master can't be failed over

# in case of failure if it has no working replicas.

#

# Replicas migrate to orphaned masters only if there are still at least a

# given number of other working replicas for their old master. This number

# is the "migration barrier". A migration barrier of 1 means that a replica

# will migrate only if there is at least 1 other working replica for its master

# and so forth. It usually reflects the number of replicas you want for every

# master in your cluster.

#

# Default is 1 (replicas migrate only if their masters remain with at least

# one replica). To disable migration just set it to a very large value.

# A value of 0 can be set but is useful only for debugging and dangerous

# in production.

#

# cluster-migration-barrier 1

 

# By default Redis Cluster nodes stop accepting queries if they detect there

# is at least an hash slot uncovered (no available node is serving it).

# This way if the cluster is partially down (for example a range of hash slots

# are no longer covered) all the cluster becomes, eventually, unavailable.

# It automatically returns available as soon as all the slots are covered again.

#

# However sometimes you want the subset of the cluster which is working,

# to continue to accept queries for the part of the key space that is still

# covered. In order to do so, just set the cluster-require-full-coverage

# option to no.

#

# cluster-require-full-coverage yes

 

# This option, when set to yes, prevents replicas from trying to failover its

# master during master failures. However the master can still perform a

# manual failover, if forced to do so.

#

# This is useful in different scenarios, especially in the case of multiple

# data center operations, where we want one side to never be promoted if not

# in the case of a total DC failure.

#

# cluster-replica-no-failover no

 

# In order to setup your cluster make sure to read the documentation

# available at http://redis.io web site.

 

########################## CLUSTER DOCKER/NAT support  ########################

 

# In certain deployments, Redis Cluster nodes address discovery fails, because

# addresses are NAT-ted or because ports are forwarded (the typical case is

# Docker and other containers).

#

# In order to make Redis Cluster working in such environments, a static

# configuration where each node knows its public address is needed. The

# following two options are used for this scope, and are:

#

# * cluster-announce-ip

# * cluster-announce-port

# * cluster-announce-bus-port

#

# Each instruct the node about its address, client port, and cluster message

# bus port. The information is then published in the header of the bus packets

# so that other nodes will be able to correctly map the address of the node

# publishing the information.

#

# If the above options are not used, the normal Redis Cluster auto-detection

# will be used instead.

#

# Note that when remapped, the bus port may not be at the fixed offset of

# clients port + 10000, so you can specify any port and bus-port depending

# on how they get remapped. If the bus-port is not set, a fixed offset of

# 10000 will be used as usually.

#

# Example:

#

# cluster-announce-ip 10.1.1.5

# cluster-announce-port 6379

# cluster-announce-bus-port 6380

 

################################## SLOW LOG ###################################

 

# The Redis Slow Log is a system to log queries that exceeded a specified

# execution time. The execution time does not include the I/O operations

# like talking with the client, sending the reply and so forth,

# but just the time needed to actually execute the command (this is the only

# stage of command execution where the thread is blocked and can not serve

# other requests in the meantime).

#

# You can configure the slow log with two parameters: one tells Redis

# what is the execution time, in microseconds, to exceed in order for the

# command to get logged, and the other parameter is the length of the

# slow log. When a new command is logged the oldest one is removed from the

# queue of logged commands.

 

# The following time is expressed in microseconds, so 1000000 is equivalent

# to one second. Note that a negative number disables the slow log, while

# a value of zero forces the logging of every command.

slowlog-log-slower-than 10000

 

# There is no limit to this length. Just be aware that it will consume memory.

# You can reclaim memory used by the slow log with SLOWLOG RESET.

slowlog-max-len 128

 

################################ LATENCY MONITOR ##############################

 

# The Redis latency monitoring subsystem samples different operations

# at runtime in order to collect data related to possible sources of

# latency of a Redis instance.

#

# Via the LATENCY command this information is available to the user that can

# print graphs and obtain reports.

#

# The system only logs operations that were performed in a time equal or

# greater than the amount of milliseconds specified via the

# latency-monitor-threshold configuration directive. When its value is set

# to zero, the latency monitor is turned off.

#

# By default latency monitoring is disabled since it is mostly not needed

# if you don't have latency issues, and collecting data has a performance

# impact, that while very small, can be measured under big load. Latency

# monitoring can easily be enabled at runtime using the command

# "CONFIG SET latency-monitor-threshold <milliseconds>" if needed.

latency-monitor-threshold 0

 

############################# EVENT NOTIFICATION ##############################

 

# Redis can notify Pub/Sub clients about events happening in the key space.

# This feature is documented at http://redis.io/topics/notifications

#

# For instance if keyspace events notification is enabled, and a client

# performs a DEL operation on key "foo" stored in the Database 0, two

# messages will be published via Pub/Sub:

#

# PUBLISH __keyspace@0__:foo del

# PUBLISH __keyevent@0__:del foo

#

# It is possible to select the events that Redis will notify among a set

# of classes. Every class is identified by a single character:

#

#  K     Keyspace events, published with __keyspace@<db>__ prefix.

#  E     Keyevent events, published with __keyevent@<db>__ prefix.

#  g     Generic commands (non-type specific) like DEL, EXPIRE, RENAME, ...

#  $     String commands

#  l     List commands

#  s     Set commands

#  h     Hash commands

#  z     Sorted set commands

#  x     Expired events (events generated every time a key expires)

#  e     Evicted events (events generated when a key is evicted for maxmemory)

#  A     Alias for g$lshzxe, so that the "AKE" string means all the events.

#

#  The "notify-keyspace-events" takes as argument a string that is composed

#  of zero or multiple characters. The empty string means that notifications

#  are disabled.

#

#  Example: to enable list and generic events, from the point of view of the

#           event name, use:

#

#  notify-keyspace-events Elg

#

#  Example 2: to get the stream of the expired keys subscribing to channel

#             name __keyevent@0__:expired use:

#

  notify-keyspace-events Ex

#

#  By default all notifications are disabled because most users don't need

#  this feature and the feature has some overhead. Note that if you don't

#  specify at least one of K or E, no events will be delivered.

#notify-keyspace-events ""

 

############################### ADVANCED CONFIG ###############################

 

# Hashes are encoded using a memory efficient data structure when they have a

# small number of entries, and the biggest entry does not exceed a given

# threshold. These thresholds can be configured using the following directives.

hash-max-ziplist-entries 512

hash-max-ziplist-value 64

 

# Lists are also encoded in a special way to save a lot of space.

# The number of entries allowed per internal list node can be specified

# as a fixed maximum size or a maximum number of elements.

# For a fixed maximum size, use -5 through -1, meaning:

# -5: max size: 64 Kb  <-- not recommended for normal workloads

# -4: max size: 32 Kb  <-- not recommended

# -3: max size: 16 Kb  <-- probably not recommended

# -2: max size: 8 Kb   <-- good

# -1: max size: 4 Kb   <-- good

# Positive numbers mean store up to _exactly_ that number of elements

# per list node.

# The highest performing option is usually -2 (8 Kb size) or -1 (4 Kb size),

# but if your use case is unique, adjust the settings as necessary.

list-max-ziplist-size -2

 

# Lists may also be compressed.

# Compress depth is the number of quicklist ziplist nodes from *each* side of

# the list to *exclude* from compression.  The head and tail of the list

# are always uncompressed for fast push/pop operations.  Settings are:

# 0: disable all list compression

# 1: depth 1 means "don't start compressing until after 1 node into the list,

#    going from either the head or tail"

#    So: [head]->node->node->...->node->[tail]

#    [head], [tail] will always be uncompressed; inner nodes will compress.

# 2: [head]->[next]->node->node->...->node->[prev]->[tail]

#    2 here means: don't compress head or head->next or tail->prev or tail,

#    but compress all nodes between them.

# 3: [head]->[next]->[next]->node->node->...->node->[prev]->[prev]->[tail]

# etc.

list-compress-depth 0

 

# Sets have a special encoding in just one case: when a set is composed

# of just strings that happen to be integers in radix 10 in the range

# of 64 bit signed integers.

# The following configuration setting sets the limit in the size of the

# set in order to use this special memory saving encoding.

set-max-intset-entries 512

 

# Similarly to hashes and lists, sorted sets are also specially encoded in

# order to save a lot of space. This encoding is only used when the length and

# elements of a sorted set are below the following limits:

zset-max-ziplist-entries 128

zset-max-ziplist-value 64

 

# HyperLogLog sparse representation bytes limit. The limit includes the

# 16 bytes header. When an HyperLogLog using the sparse representation crosses

# this limit, it is converted into the dense representation.

#

# A value greater than 16000 is totally useless, since at that point the

# dense representation is more memory efficient.

#

# The suggested value is ~ 3000 in order to have the benefits of

# the space efficient encoding without slowing down too much PFADD,

# which is O(N) with the sparse encoding. The value can be raised to

# ~ 10000 when CPU is not a concern, but space is, and the data set is

# composed of many HyperLogLogs with cardinality in the 0 - 15000 range.

hll-sparse-max-bytes 3000

 

# Streams macro node max size / items. The stream data structure is a radix

# tree of big nodes that encode multiple items inside. Using this configuration

# it is possible to configure how big a single node can be in bytes, and the

# maximum number of items it may contain before switching to a new node when

# appending new stream entries. If any of the following settings are set to

# zero, the limit is ignored, so for instance it is possible to set just a

# max entires limit by setting max-bytes to 0 and max-entries to the desired

# value.

stream-node-max-bytes 4096

stream-node-max-entries 100

 

# Active rehashing uses 1 millisecond every 100 milliseconds of CPU time in

# order to help rehashing the main Redis hash table (the one mapping top-level

# keys to values). The hash table implementation Redis uses (see dict.c)

# performs a lazy rehashing: the more operation you run into a hash table

# that is rehashing, the more rehashing "steps" are performed, so if the

# server is idle the rehashing is never complete and some more memory is used

# by the hash table.

#

# The default is to use this millisecond 10 times every second in order to

# actively rehash the main dictionaries, freeing memory when possible.

#

# If unsure:

# use "activerehashing no" if you have hard latency requirements and it is

# not a good thing in your environment that Redis can reply from time to time

# to queries with 2 milliseconds delay.

#

# use "activerehashing yes" if you don't have such hard requirements but

# want to free memory asap when possible.

activerehashing yes

 

# The client output buffer limits can be used to force disconnection of clients

# that are not reading data from the server fast enough for some reason (a

# common reason is that a Pub/Sub client can't consume messages as fast as the

# publisher can produce them).

#

# The limit can be set differently for the three different classes of clients:

#

# normal -> normal clients including MONITOR clients

# replica  -> replica clients

# pubsub -> clients subscribed to at least one pubsub channel or pattern

#

# The syntax of every client-output-buffer-limit directive is the following:

#

# client-output-buffer-limit <class> <hard limit> <soft limit> <soft seconds>

#

# A client is immediately disconnected once the hard limit is reached, or if

# the soft limit is reached and remains reached for the specified number of

# seconds (continuously).

# So for instance if the hard limit is 32 megabytes and the soft limit is

# 16 megabytes / 10 seconds, the client will get disconnected immediately

# if the size of the output buffers reach 32 megabytes, but will also get

# disconnected if the client reaches 16 megabytes and continuously overcomes

# the limit for 10 seconds.

#

# By default normal clients are not limited because they don't receive data

# without asking (in a push way), but just after a request, so only

# asynchronous clients may create a scenario where data is requested faster

# than it can read.

#

# Instead there is a default limit for pubsub and replica clients, since

# subscribers and replicas receive data in a push fashion.

#

# Both the hard or the soft limit can be disabled by setting them to zero.

client-output-buffer-limit normal 0 0 0

client-output-buffer-limit replica 256mb 64mb 60

client-output-buffer-limit pubsub 32mb 8mb 60

 

# Client query buffers accumulate new commands. They are limited to a fixed

# amount by default in order to avoid that a protocol desynchronization (for

# instance due to a bug in the client) will lead to unbound memory usage in

# the query buffer. However you can configure it here if you have very special

# needs, such us huge multi/exec requests or alike.

#

# client-query-buffer-limit 1gb

 

# In the Redis protocol, bulk requests, that are, elements representing single

# strings, are normally limited ot 512 mb. However you can change this limit

# here.

#

# proto-max-bulk-len 512mb

 

# Redis calls an internal function to perform many background tasks, like

# closing connections of clients in timeout, purging expired keys that are

# never requested, and so forth.

#

# Not all tasks are performed with the same frequency, but Redis checks for

# tasks to perform according to the specified "hz" value.

#

# By default "hz" is set to 10. Raising the value will use more CPU when

# Redis is idle, but at the same time will make Redis more responsive when

# there are many keys expiring at the same time, and timeouts may be

# handled with more precision.

#

# The range is between 1 and 500, however a value over 100 is usually not

# a good idea. Most users should use the default of 10 and raise this up to

# 100 only in environments where very low latency is required.

hz 10

 

# Normally it is useful to have an HZ value which is proportional to the

# number of clients connected. This is useful in order, for instance, to

# avoid too many clients are processed for each background task invocation

# in order to avoid latency spikes.

#

# Since the default HZ value by default is conservatively set to 10, Redis

# offers, and enables by default, the ability to use an adaptive HZ value

# which will temporary raise when there are many connected clients.

#

# When dynamic HZ is enabled, the actual configured HZ will be used as

# as a baseline, but multiples of the configured HZ value will be actually

# used as needed once more clients are connected. In this way an idle

# instance will use very little CPU time while a busy instance will be

# more responsive.

dynamic-hz yes

 

# When a child rewrites the AOF file, if the following option is enabled

# the file will be fsync-ed every 32 MB of data generated. This is useful

# in order to commit the file to the disk more incrementally and avoid

# big latency spikes.

aof-rewrite-incremental-fsync yes

 

# When redis saves RDB file, if the following option is enabled

# the file will be fsync-ed every 32 MB of data generated. This is useful

# in order to commit the file to the disk more incrementally and avoid

# big latency spikes.

rdb-save-incremental-fsync yes

 

# Redis LFU eviction (see maxmemory setting) can be tuned. However it is a good

# idea to start with the default settings and only change them after investigating

# how to improve the performances and how the keys LFU change over time, which

# is possible to inspect via the OBJECT FREQ command.

#

# There are two tunable parameters in the Redis LFU implementation: the

# counter logarithm factor and the counter decay time. It is important to

# understand what the two parameters mean before changing them.

#

# The LFU counter is just 8 bits per key, it's maximum value is 255, so Redis

# uses a probabilistic increment with logarithmic behavior. Given the value

# of the old counter, when a key is accessed, the counter is incremented in

# this way:

#

# 1. A random number R between 0 and 1 is extracted.

# 2. A probability P is calculated as 1/(old_value*lfu_log_factor+1).

# 3. The counter is incremented only if R < P.

#

# The default lfu-log-factor is 10. This is a table of how the frequency

# counter changes with a different number of accesses with different

# logarithmic factors:

#

# +--------+------------+------------+------------+------------+------------+

# | factor | 100 hits   | 1000 hits  | 100K hits  | 1M hits    | 10M hits   |

# +--------+------------+------------+------------+------------+------------+

# | 0      | 104        | 255        | 255        | 255        | 255        |

# +--------+------------+------------+------------+------------+------------+

# | 1      | 18         | 49         | 255        | 255        | 255        |

# +--------+------------+------------+------------+------------+------------+

# | 10     | 10         | 18         | 142        | 255        | 255        |

# +--------+------------+------------+------------+------------+------------+

# | 100    | 8          | 11         | 49         | 143        | 255        |

# +--------+------------+------------+------------+------------+------------+

#

# NOTE: The above table was obtained by running the following commands:

#

#   redis-benchmark -n 1000000 incr foo

#   redis-cli object freq foo

#

# NOTE 2: The counter initial value is 5 in order to give new objects a chance

# to accumulate hits.

#

# The counter decay time is the time, in minutes, that must elapse in order

# for the key counter to be divided by two (or decremented if it has a value

# less <= 10).

#

# The default value for the lfu-decay-time is 1. A Special value of 0 means to

# decay the counter every time it happens to be scanned.

#

# lfu-log-factor 10

# lfu-decay-time 1

 

########################### ACTIVE DEFRAGMENTATION #######################

#

# WARNING THIS FEATURE IS EXPERIMENTAL. However it was stress tested

# even in production and manually tested by multiple engineers for some

# time.

#

# What is active defragmentation?

# -------------------------------

#

# Active (online) defragmentation allows a Redis server to compact the

# spaces left between small allocations and deallocations of data in memory,

# thus allowing to reclaim back memory.

#

# Fragmentation is a natural process that happens with every allocator (but

# less so with Jemalloc, fortunately) and certain workloads. Normally a server

# restart is needed in order to lower the fragmentation, or at least to flush

# away all the data and create it again. However thanks to this feature

# implemented by Oran Agra for Redis 4.0 this process can happen at runtime

# in an "hot" way, while the server is running.

#

# Basically when the fragmentation is over a certain level (see the

# configuration options below) Redis will start to create new copies of the

# values in contiguous memory regions by exploiting certain specific Jemalloc

# features (in order to understand if an allocation is causing fragmentation

# and to allocate it in a better place), and at the same time, will release the

# old copies of the data. This process, repeated incrementally for all the keys

# will cause the fragmentation to drop back to normal values.

#

# Important things to understand:

#

# 1. This feature is disabled by default, and only works if you compiled Redis

#    to use the copy of Jemalloc we ship with the source code of Redis.

#    This is the default with Linux builds.

#

# 2. You never need to enable this feature if you don't have fragmentation

#    issues.

#

# 3. Once you experience fragmentation, you can enable this feature when

#    needed with the command "CONFIG SET activedefrag yes".

#

# The configuration parameters are able to fine tune the behavior of the

# defragmentation process. If you are not sure about what they mean it is

# a good idea to leave the defaults untouched.

 

# Enabled active defragmentation

# activedefrag yes

 

# Minimum amount of fragmentation waste to start active defrag

# active-defrag-ignore-bytes 100mb

 

# Minimum percentage of fragmentation to start active defrag

# active-defrag-threshold-lower 10

 

# Maximum percentage of fragmentation at which we use maximum effort

# active-defrag-threshold-upper 100

 

# Minimal effort for defrag in CPU percentage

# active-defrag-cycle-min 5

 

# Maximal effort for defrag in CPU percentage

# active-defrag-cycle-max 75

 

# Maximum number of set/hash/zset/list fields that will be processed from

# the main dictionary scan

# active-defrag-max-scan-fields 1000
```

## 2.4:创建容器

```sh
docker run -d \
--name redis \
-p 6379:6379 \
--restart unless-stopped \
-v /home/docker/redis/data:/data \
-v /home/docker/redis/conf/redis.conf:/etc/redis/redis.conf \
redis:6.2.6 \
redis-server /etc/redis/redis.conf
```

# 3:安装Nginx

## 3.1:拉取镜像

docker pull nginx:1.18.0

## 3.2:新建挂载目录

mkdir -p /home/docker/nginx/{conf,log,html}

## 3.3:创建配置文件

创建配置文件

1：可以直接在网上找到一个Nginx的配置这些，然后在启动容器的时候直接挂载目录

2：也可以先直接创建一个容器，不指定数据卷的目录，然后把配置文件复制到自己的Linux上面，再删除之前创建的容器，重新创建这个容器

之前我们使用的是第一个方式，我们这里使用第二个方式



生成内容并复制配置

```sh
# 先生成容器
docker run --name nginx -p 80:80 -d nginx:1.18.0

# 将容器nginx.conf文件复制到宿主机
docker cp nginx:/etc/nginx/nginx.conf /home/docker/nginx/conf/nginx.conf

# 将容器conf.d文件夹下内容复制到宿主机
docker cp nginx:/etc/nginx/conf.d /home/docker/nginx/conf/conf.d

# 将容器中的html文件夹复制到宿主机
docker cp nginx:/usr/share/nginx/html /home/docker/nginx
```

删除前期准备

```
# 直接执行docker rm nginx或者以容器id方式关闭容器

# 找到nginx对应的容器id
docker ps -a

# 关闭该容器
docker stop nginx

# 删除该容器
docker rm nginx

# 删除正在运行的nginx容器
docker rm -f nginx
```

## 3.4:创建容器

```sh
docker run \
-p 80:80 \
--name nginx \
-v /home/docker/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
-v /home/docker/nginx/conf/conf.d:/etc/nginx/conf.d \
-v /home/docker/nginx/log:/var/log/nginx \
-v /home/docker/nginx/html:/usr/share/nginx/html \
-d nginx:1.18.0
```

# 4:安装Gogs

## 4.1:拉取镜像

docker pull gogs/gogs

## 4.2:新建挂载目录

mkdir -p /home/docker/gogs

## 4.3:创建容器

docker run --name=gogs -p 10022:22 -p 10880:3000 -v /home/docker/gogs:/data gogs/gogs

## 4.4:注意事项

![image-20230130205442282](https://zzx-note.oss-cn-beijing.aliyuncs.com/docker/image-20230130205442282.png)

## 4.5:测试

http://192.168.101.66:10880/

# 5:安装RabbitMQ

## 5.1:拉取镜像

docker pull rabbitmq:management

## 5.2:创建容器

挂载的时候可以使用数据卷挂载，也可以直接使用目录挂载，我这里使用了数据卷挂载

```sh
docker run \
 -e RABBITMQ_DEFAULT_USER=ZZX \
 -e RABBITMQ_DEFAULT_PASS=JXLZZX79 \
 -v mq-plugins:/plugins \
 --name rabbitmq \
 -p 5672:5672 \
 -p 15672:15672 \
 -d \
 rabbitmq:management
```

## 5.3:测试

http://192.168.101.66:15672/

## 5.4:查看latest具体版本

我们后面需要安装演示插件，但是我们使用docker images显示的是这样

![image-20230330093210957](https://zzx-note.oss-cn-beijing.aliyuncs.com/docker/image-20230330093210957.png)

我们需要查看具体的版本，怎么做？

使用 docker image inspect rabbitmq:management

![image-20230330093507097](https://zzx-note.oss-cn-beijing.aliyuncs.com/docker/image-20230330093507097.png)

## 5.4:下载延时插件

我们之前是基于Docker安装RabbitMQ，所以下面会使用Docker来安装RabbitMQ插件

RabbitMQ有一个官方的插件社区，地址为：https://www.rabbitmq.com/community-plugins.html

其中包含各种各样的插件，包括我们要使用的DelayExchange插件：

我们下载和RabbitMQ版本对应的插件即可

比如我的是3.9的版本：

https://github.com/rabbitmq/rabbitmq-delayed-message-exchange/releases/tag/3.9.0

## 5.5:上传插件

因为我们是基于Docker安装，所以需要先查看RabbitMQ的插件目录对应的数据卷。如果不是基于Docker的同学，请参考第一章部分，重新创建Docker容器。

我们之前设定的RabbitMQ的数据卷名称为`mq-plugins`，所以我们使用下面命令查看数据卷：

```sh
docker volume inspect mq-plugins
```

可以得到下面结果：

![image-20230330104146423](https://zzx-note.oss-cn-beijing.aliyuncs.com/docker/image-20230330104146423.png)

接下来，将插件上传到这个目录即可



## 5.6.安装插件

最后就是安装了，需要进入MQ容器内部来执行安装。我的容器名为`mq`，所以执行下面命令：

```sh
docker exec -it rabbitmq bash
```

执行时，请将其中的 `-it` 后面的`mq`替换为你自己的容器名.

进入容器内部后，执行下面命令开启插件：

```sh
rabbitmq-plugins enable rabbitmq_delayed_message_exchange
```

结果如下：

![image-20230330104246939](https://zzx-note.oss-cn-beijing.aliyuncs.com/docker/image-20230330104246939.png)







# 6:安装ES

## 6.1:创建网络

因为我们还需要部署kibana容器，因此需要让es和kibana容器互联。这里先创建一个网络

```sh
docker network create es-net
```

## 6.2:加载镜像

这里我们采用elasticsearch的7.12.1版本的镜像，这个镜像体积非常大，接近1G

```
docker pull elasticsearch:7.12.1
```

安装kibana的版本，也是7.12.1的镜像

```
docker pull kibana:7.12.1
```

## 6.3:新建挂载目录

mkdir -p /home/docker/elasticsearch/{data,plugins,logs}

记得授予权限，不然可能报错，有可能不报错

```
chmod 777 data

chmod 777 plugins

chmod 777 logs
```

## 6.4:创建ES容器

```
docker run -d --name es \
    -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" \
    -e "discovery.type=single-node" \
    -e "http.host=0.0.0.0" \
    -v /home/docker/elasticsearch/data:/usr/share/elasticsearch/data \
    -v /home/docker/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
    -v /home/docker/elasticsearch/logs:/usr/share/elasticsearch/logs \
    --privileged \
    --network es-net \
    -p 9200:9200 \
    -p 9300:9300 \
elasticsearch:7.12.1
```

命令解释：

- `-e "cluster.name=es-docker-cluster"`：设置集群名称
- `-e "http.host=0.0.0.0"`：监听的地址，可以外网访问
- `-e "ES_JAVA_OPTS=-Xms512m -Xmx512m"`：内存大小
- `-e "discovery.type=single-node"`：非集群模式
- `-v es-data:/usr/share/elasticsearch/data`：挂载逻辑卷，会自动创建这个数据卷，绑定es的数据目录
- `-v es-logs:/usr/share/elasticsearch/logs`：挂载逻辑卷，绑定es的日志目录
- `-v es-plugins:/usr/share/elasticsearch/plugins`：挂载逻辑卷，绑定es的插件目录
- `--privileged`：授予逻辑卷访问权
- `--network es-net` ：加入一个名为es-net的网络中
- `-p 9200:9200`：端口映射配置

## 6.5:创建Kibana容器

```sh
docker run -d \
--name kibana \
-e ELASTICSEARCH_HOSTS=http://es:9200 \
--network=es-net \
-p 5601:5601  \
kibana:7.12.1
```

```
-network es-net ：加入一个名为es-net的网络中，与elasticsearch在同一个网络中

-e ELASTICSEARCH_HOSTS=http://es:9200：
设置elasticsearch的地址，因为kibana已经与elasticsearch在一个网络
因此可以用容器名直接访问elasticsearch

-p 5601:5601：端口映射配置
```

## 6.6:测试

测试ES服务：http://192.168.101.66:9200 

测试Kibana服务：http://192.168.101.66:5601

## 6.7:安装分词器

我们之前挂载了目录，查看创建容器的语句，存在下面几句

```
-v /home/docker/elasticsearch/data:/usr/share/elasticsearch/data \
-v /home/docker/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
-v /home/docker/elasticsearch/logs:/usr/share/elasticsearch/logs \
```

我们只需要把我们的插件放进我们对应的plugins即可

![image-20230131180223087](https://zzx-note.oss-cn-beijing.aliyuncs.com/docker/image-20230131180223087.png)

![image-20230131180259464](https://zzx-note.oss-cn-beijing.aliyuncs.com/docker/image-20230131180259464.png)

这样就可以使用了

# 7:安装MongDB

## 7.1:拉取镜像

 docker pull mongo:4.4

## 7.2:新建挂载目录

mkdir -p /home/docker/mongodb/data

## 7.3:创建容器

```
docker run -d \
  --name mongodb \
  --privileged \
  -p 27017:27017 \
  -v /home/docker/mongodb/data:/data/db \
  -e MONGO_INITDB_ROOT_USERNAME=ZZX \
  -e MONGO_INITDB_ROOT_PASSWORD=JXLZZX79 \
  mongo:4.4 --auth
```

## 7.4:测试

![image-20230131213828121](https://zzx-note.oss-cn-beijing.aliyuncs.com/docker/image-20230131213828121.png)

# 8:安装Zookeeper

## 8.1:拉取镜像

docker pull zookeeper:3.7.0

## 8.2:新建挂载目录

mkdir -p /home/docker/zookeeper/{data,conf,logs}

## 8.3:创建容器

```
docker run -d \
--name zookeeper \
--privileged=true \
-e TZ="Asia/Shanghai" \
-p 2181:2181  \
-v /home/docker/zookeeper/data:/data \
-v /home/docker/zookeeper/conf:/conf \
-v /home/docker/zookeeper/logs:/datalog \
zookeeper:3.7.0
```

## 8.4:测试连接

cmd窗口下执行：zkCli.cmd -server 192.168.101.66:2181

## 8.5:可视化工具

https://github.com/vran-dev/PrettyZoo/releases

# 9:安装postgresql

## 9.1:拉取镜像

docker pull postgres:14.1

## 9.2:新建挂载目录

mkdir -p /home/docker/postgresql/{data}

## 9.3:创建容器

因为我安装Halo的时候使用了postgresql数据库占用了5432端口

所以我docker的postgresql数据库就用5433端口

```
docker run --name postgresql \
 -e POSTGRES_PASSWORD=JXLZZX79 \
 -p 5433:5432 \
 -v /home/docker/postgresql/data:/var/lib/postgresql/data \
 -d postgres:14.1
```

# 10:禅道

## 10.1:拉取镜像

docker pull easysoft/zentao:12.3.3

## 10.2:新建挂载目录

mkdir -p /home/docker/zentao/{zentaopms,mysqldata}

## 10.3:创建容器

```
docker run --name zentao -p 8848:80  \
-v /opt/zentao/zentaopms:/www/zentaopms \
-v /opt/zentao/mysqldata:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=root \
-d easysoft/zentao:12.3.3
```

禅道的端口默认是80，因为Nginx默认也是80，所以我这里换成了8848端口

# 11:安装Halo

## 11.1:创建目录

mkdir ~/halo && cd ~/halo

## 11.2:创建文件

在halo目录下新建 docker-compose.yaml，内容如下

```yaml
version: "3"

services:
  halo:
    image: halohub/halo:2.2.0
    container_name: halo
    restart: on-failure:3
    depends_on:
      halodb:
        condition: service_healthy
    networks:
      halo_network:
    volumes:
      - ./:/root/.halo2
    ports:
      - "8090:8090"
    command:
      - --spring.r2dbc.url=r2dbc:pool:postgresql://halodb/halo
      - --spring.r2dbc.username=halo
      - --spring.r2dbc.password=JXLZZX79
      - --spring.sql.init.platform=postgresql
      - --halo.external-url=http://101.42.247.138:8090/
      - --halo.security.initializer.superadminusername=ZZX
      - --halo.security.initializer.superadminpassword=JXLZZX79
  halodb:
    image: postgres:latest
    container_name: halodb
    restart: on-failure:3
    networks:
      halo_network:
    volumes:
      - ./db:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    healthcheck:
      test: [ "CMD", "pg_isready" ]
      interval: 10s
      timeout: 5s
      retries: 5
    environment:
      - POSTGRES_PASSWORD=JXLZZX79
      - POSTGRES_USER=halo
      - POSTGRES_DB=halo
      - PGUSER=halo
networks:
  halo_network:
```

## 11.3:构建镜像

在halo目录下执行：docker-compose up -d

## 11.4:配置Nginx

在宝塔面板中安装Nginx

添加站点

![image-20230216101747385](https://zzx-note.oss-cn-beijing.aliyuncs.com/docker/image-20230216101747385.png)

注意：只需要填写域名即可

## 11.5:配置域名

```properties
upstream halo {
  server 101.42.247.138:8090;
}

server
{
    listen 80;
    server_name zzxjxl.online;
    index index.php index.html index.htm default.php default.htm default.html;
    root /www/wwwroot/zzxjxl.online;

    include enable-php-00.conf;

    include /www/server/panel/vhost/rewrite/zzxjxl.online.conf;

    location / {
      proxy_set_header HOST $host;
      proxy_set_header X-Forwarded-Proto $scheme;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_pass http://halo;
    }
    
    location ~ [^/]\.php(/|$) {;
      fastcgi_pass unix:/dev/shm/php-cgi.sock;
      fastcgi_index index.php;
      include fastcgi.conf;
    }
  
    location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|flv|mp4|ico)$ {
      proxy_pass http://halo;
      expires 30d;
      access_log off;
    }
    
    location ~ .*\.(js|css)?$ {
      proxy_pass http://halo;
      expires 7d;
      access_log off;
    }
  
    location ~ /(\.user\.ini|\.ht|\.git|\.svn|\.project|LICENSE|README\.md) {
      deny all;
    }
    
    location /.well-known {
      allow all;
    }
    
    access_log  /www/wwwlogs/zzxjxl.online.log;
    error_log  /www/wwwlogs/zzxjxl.online.error.log;
}
```

主要就是修改域名这一块内容和自己服务器的IP，其它的都不动即可！