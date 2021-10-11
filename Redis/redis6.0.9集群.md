# Redis的Cluster集群搭建(CentOS7环境，Redis6.0.9)



**目录**

 

[一、介绍](https://blog.csdn.net/xlyrh/article/details/110776381#t1)

[二、部署Redis的Cluster集群](https://blog.csdn.net/xlyrh/article/details/110776381#t2)

[1、编译完成Redis文件](https://blog.csdn.net/xlyrh/article/details/110776381#t3)

[2、新建准备好6个文件夹](https://blog.csdn.net/xlyrh/article/details/110776381#t4)

[3、拷贝redis执行文件和conf](https://blog.csdn.net/xlyrh/article/details/110776381#t5)

[4、修改redis.conf文件](https://blog.csdn.net/xlyrh/article/details/110776381#t6)

[5、创建启动文件start_all.sh](https://blog.csdn.net/xlyrh/article/details/110776381#t7)

[ 6、创建停止文件stop_all.sh](https://blog.csdn.net/xlyrh/article/details/110776381#t8)

[7、启动节点](https://blog.csdn.net/xlyrh/article/details/110776381#t9)

[8、创建集群](https://blog.csdn.net/xlyrh/article/details/110776381#t10)

[三、验证集群正常工作](https://blog.csdn.net/xlyrh/article/details/110776381#t11)

[参考文献：](https://blog.csdn.net/xlyrh/article/details/110776381#t12)

------

# 一、介绍

在单机版的Redis中，每个Master之间是没有任何通信的，所以我们一般在Jedis客户端或者Codis这样的代理中做Pre-sharding。按照CAP理论来说，单机版的Redis属于保证CP(Consistency &  Partition-Tolerancy)而牺牲A(Availability)，也就说Redis能够保证所有用户看到相同的数据（一致性，因为Redis不自动冗余数据）和网络通信出问题时，暂时隔离开的子系统能继续运行（分区容忍性，因为Master之间没有直接关系，不需要通信），但是不保证某些结点故障时，所有请求都能被响应（可用性，某个Master结点挂了的话，那么它上面分片的数据就无法访问了）。有了Cluster功能后，Redis从一个单纯的NoSQL内存数据库变成了分布式NoSQL数据库，CAP模型也从CP变成了AP。也就是说，通过自动分片和冗余数据，Redis具有了真正的分布式能力，某个结点挂了的话，因为数据在其他结点上有备份，所以其他结点顶上来就可以继续提供服务，保证了Availability。然而，也正因为这一点，Redis无法保证曾经的强一致性了。这也是CAP理论要求的，三者只能取其二。

# 二、部署Redis的Cluster集群

## 1、编译完成Redis文件

详见上一篇博客文章

## 2、新建准备好6个文件夹

> [root@10 opt]# mkdir redis
>  [root@10 opt]# cd redis
>  [root@10 redis]# mkdir redis0
>  [root@10 redis]# mkdir redis1
>  [root@10 redis]# mkdir redis2
>  [root@10 redis]# mkdir redis3
>  [root@10 redis]# mkdir redis4
>  [root@10 redis]# mkdir redis5

## 3、拷贝redis执行文件和conf

> [root@10 redis0]# cp ../../redis-6.0.9/bin/* .
>
> [root@10 redis0]# cp ../../redis-6.0.9/etc/* .
>
> [root@10 redis0]# cd ../redis1
>  [root@10 redis1]# cp ../redis0/* 
>
> [root@10 redis1]# cd ../redis2
>  [root@10 redis2]# cp ../redis0/* .
>  [root@10 redis2]# cd ../redis3
>  [root@10 redis3]# cp ../redis0/* .
>  [root@10 redis3]# cd ../redis4
>  [root@10 redis4]# cp ../redis0/* .
>  [root@10 redis4]# cd ../redis5
>  [root@10 redis5]# cp ../redis0/* .
>  [root@10 redis5]# ls
>  dump.rdb     redis-benchmark  redis-check-rdb  redis.conf    redis-server
>  mkreleasehdr.sh  redis-check-aof  redis-cli     redis-sentinel  redis-trib.rb

## 4、修改redis.conf文件

修改以下四项内容（红色部分）

\################################## NETWORK #####################################

\# By default, if no "bind" configuration directive is specified, Redis listens
 \# for connections from all available network interfaces on the host machine.
 \# It is possible to listen to just one or multiple selected interfaces using
 \# the "bind" configuration directive, followed by one or more IP addresses.
 \#
 \# Examples:
 \#
 \# bind 192.168.1.100 10.0.0.1
 \# bind 127.0.0.1 ::1
 \#
 \# ~~~ WARNING ~~~ If the computer running Redis is directly exposed to the
 \# internet, binding to all the interfaces is dangerous and will expose the
 \# instance to everybody on the internet. So by default we uncomment the
 \# following bind directive, that will force Redis to listen only on the
 \# IPv4 loopback interface address (this means Redis will only be able to
 \# accept client connections from the same host that it is running on).
 \#
 \# IF YOU ARE SURE YOU WANT YOUR INSTANCE TO LISTEN TO ALL THE INTERFACES
 \# JUST COMMENT OUT THE FOLLOWING LINE.
 \# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 bind 127.0.0.1
bind 192.168.56.101

\# Accept connections on the specified port, default is 6379 (IANA #815344).
 \# If port 0 is specified Redis will not listen on a TCP socket.
 port 6379

 

\################################# GENERAL #####################################

\# By default Redis does not run as a daemon. Use 'yes' if you need it.
 \# Note that Redis will write a pid file in /var/run/redis.pid when daemonized.
 daemonize yes

\################################ REDIS CLUSTER  ###############################

\# Normal Redis instances can't be part of a Redis Cluster; only nodes that are
 \# started as cluster nodes can. In order to start a Redis instance as a
 \# cluster node enable the cluster support uncommenting the following:
 \#
 cluster-enabled yes

注意：同一个IP中6个文件夹里的port端口号要不一样，这里是6379-6484，在实际使用中不同IP的端口号可以设置一样。

## 5、创建启动文件start_all.sh

为了方便管理，启动文件和终止文件都放在redis文件夹下，还有拷贝出来一份redis-cli文件

```bash
cd redis0./redis-server redis.confcd ..cd redis1./redis-server redis.confcd ..cd redis2./redis-server redis.confcd ..cd redis3./redis-server redis.confcd ..cd redis4./redis-server redis.confcd ..cd redis5./redis-server redis.confcd ..
```

##  6、创建停止文件stop_all.sh

```bash
./redis1/redis-cli -p 6379 shutdown./redis1/redis-cli -p 6380 shutdown./redis1/redis-cli -p 6381 shutdown./redis1/redis-cli -p 6382 shutdown./redis1/redis-cli -p 6383 shutdown./redis1/redis-cli -p 6384 shutdown
```

## 7、启动节点

./start_all.sh执行启动文件，执行起来后可以通过ps查看启动情况

> [root@10 redis]# ./start_all.sh
>  [root@10 redis]# ps -ef | grep redis
>  root   14959   1  0 08:18 ?     00:00:07 ./redis-server 192.168.56.101:6379 [cluster]
>  root   14965   1  0 08:18 ?     00:00:07 ./redis-server 192.168.56.101:6380 [cluster]
>  root   14971   1  0 08:18 ?     00:00:07 ./redis-server 192.168.56.101:6381 [cluster]
>  root   14977   1  0 08:18 ?     00:00:07 ./redis-server 192.168.56.101:6382 [cluster]
>  root   14983   1  0 08:18 ?     00:00:07 ./redis-server 192.168.56.101:6383 [cluster]
>  root   14989   1  0 08:18 ?     00:00:07 ./redis-server 192.168.56.101:6384 [cluster]
>  root   18204 10810  0 09:18 pts/1   00:00:00 grep --color=auto redis

## 8、创建集群

执行./redis-cli --cluster create 192.168.56.101:6379  192.168.56.101:6380  192.168.56.101:6381 192.168.56.101:6382 192.168.56.101:6383  192.168.56.101:6384 --cluster-replicas 1命令创建集群

> [root@10 redis]# ./redis-cli --cluster create 192.168.56.101:6379  192.168.56.101:6380 192.168.56.101:6381  192.168.56.101:6382 192.168.56.101:6383 192.168.56.101:6384  --cluster-replicas 1
>  \>>> Performing hash slots allocation on 6 nodes...
>  Master[0] -> Slots 0 - 5460
>  Master[1] -> Slots 5461 - 10922
>  Master[2] -> Slots 10923 - 16383
>  Adding replica 192.168.56.101:6383 to 192.168.56.101:6379
>  Adding replica 192.168.56.101:6384 to 192.168.56.101:6380
>  Adding replica 192.168.56.101:6382 to 192.168.56.101:6381
>  \>>> Trying to optimize slaves allocation for anti-affinity
>  [WARNING] Some slaves are in the same host as their master
>  M: 522f0dca780073ab668bc80ded5a1489e2febe3e 192.168.56.101:6379
>    slots:[0-5460] (5461 slots) master
>  M: f65edec341cdc39ded3ecd5298c92f3192c8afcd 192.168.56.101:6380
>    slots:[5461-10922] (5462 slots) master
>  M: 1b55615b201a68db64db210381d9a4dbaaf6b17f 192.168.56.101:6381
>    slots:[10923-16383] (5461 slots) master
>  S: 3f9ff65caf28e79864f14f1996e8b5ecd136959a 192.168.56.101:6382
>    replicates f65edec341cdc39ded3ecd5298c92f3192c8afcd
>  S: 5a595b9da405a27a097f9756008b47414e2da1f7 192.168.56.101:6383
>    replicates 1b55615b201a68db64db210381d9a4dbaaf6b17f
>  S: 204bc647266c4c369ebe45d4ba0755c28543494e 192.168.56.101:6384
>    replicates 522f0dca780073ab668bc80ded5a1489e2febe3e
>  Can I set the above configuration? (type 'yes' to accept): yes
>  \>>> Nodes configuration updated
>  \>>> Assign a different config epoch to each node
>  \>>> Sending CLUSTER MEET messages to join the cluster
>  Waiting for the cluster to join
>  ..
>  \>>> Performing Cluster Check (using node 192.168.56.101:6379)
>  M: 522f0dca780073ab668bc80ded5a1489e2febe3e 192.168.56.101:6379
>    slots:[0-5460] (5461 slots) master
>    1 additional replica(s)
>  M: f65edec341cdc39ded3ecd5298c92f3192c8afcd 192.168.56.101:6380
>    slots:[5461-10922] (5462 slots) master
>    1 additional replica(s)
>  M: 1b55615b201a68db64db210381d9a4dbaaf6b17f 192.168.56.101:6381
>    slots:[10923-16383] (5461 slots) master
>    1 additional replica(s)
>  S: 204bc647266c4c369ebe45d4ba0755c28543494e 192.168.56.101:6384
>    slots: (0 slots) slave
>    replicates 522f0dca780073ab668bc80ded5a1489e2febe3e
>  S: 3f9ff65caf28e79864f14f1996e8b5ecd136959a 192.168.56.101:6382
>    slots: (0 slots) slave
>    replicates f65edec341cdc39ded3ecd5298c92f3192c8afcd
>  S: 5a595b9da405a27a097f9756008b47414e2da1f7 192.168.56.101:6383
>    slots: (0 slots) slave
>    replicates 1b55615b201a68db64db210381d9a4dbaaf6b17f
>  [OK] All nodes agree about slots configuration.
>  \>>> Check for open slots...
>  \>>> Check slots coverage...
>  [OK] All 16384 slots covered.

注意1：网络上很多都是使用redis-trib.rb创建集群，但是在redis5.0之后取消了ruby脚本  redis-trib.rb的支持（手动命令行添加集群的方式不变），创建集合到redis-cli里，避免了再安装ruby的相关环境。直接使用redis-clit的参数--cluster 来取代。错误信息参考如下：

> [root@10 redis]# ./redis-trib.rb create --replicas 1  192.168.56.101:6379 192.168.56.101:6380 192.168.56.101:6381  192.92.168.56.101:6383 192.168.56.101:6384
>  WARNING: redis-trib.rb is not longer available!
>  You should use redis-cli instead.
>
> All commands and features belonging to redis-trib.rb have been moved
>  to redis-cli.
>  In order to use them you should call redis-cli with the --cluster
>  option followed by the subcommand name, arguments and options.
>
> Use the following syntax:
>  redis-cli --cluster SUBCOMMAND [ARGUMENTS] [OPTIONS]
>
> Example:
>  redis-cli --cluster create 192.168.56.101:6379  192.168.56.101:6380 192.168.56.101:6381 192.168.56.101:6382  192.168.56.56.101:6384 --cluster-replicas 1
>
> To get help about all subcommands, type:
>  redis-cli --cluster help

注意2：创建集群时候提示连接报错如下

> [root@10 redis]# redis-cli --cluster create 192.168.56.101:6379  192.168.56.101:6380 192.168.56.101:6381 192.168.56.10.101:6383  192.168.56.101:6384 --cluster-replicas 1
>  Could not connect to Redis at 192.168.56.101:6379: Connection refused

解决方式：查看步骤4中Redis.conf文件的bind是否有绑定IP，这里是192.168.56.101

注意3：复制的执行文件在创建集群时候报错，错误信息如下：

> [root@10 redis]# ./redis-cli --cluster create 192.168.56.101:6379  192.168.56.101:6380 192.168.56.101:6381 192.168.56.10.101:6383  192.168.56.101:6384 --cluster-replicas 1
>  [ERR] Node  192.168.56.101:6379 is not empty. Either the node already knows other  nodes (check with CLUSTER NODES) or contains some key in database 0.

解决方法：删除每个节点中的appendonly.aof、dump.rdb、node_xxx.conf文件，redis-cli -c -h -p登录每个redis节点，执行以下命令，再创建集群就可以了

> [root@10 redis]# ./redis0/redis-cli -c -h 192.168.56.101 -p 6379
>  192.168.56.101:6379> flushdb
>  OK
>  192.168.56.101:6379> cluster reset
>  OK
>  192.168.56.101:6379> exit

# 三、验证集群正常工作

> [root@10 redis]# ./redis1/redis-cli -c -h 192.168.56.101 -p 6379
>  192.168.56.101:6379> cluster nodes
>  f65edec341cdc39ded3ecd5298c92f3192c8afcd 192.168.56.101:6380@16380 master - 0 1607262213000 2 connected 5461-10922
>  1b55615b201a68db64db210381d9a4dbaaf6b17f 192.168.56.101:6381@16381 master - 0 1607262213000 3 connected 10923-16383
>  204bc647266c4c369ebe45d4ba0755c28543494e 192.168.56.101:6384@16384 slave 522f0dca780073ab668bc80ded5a1489e2febe3e 0 1nnected
>  3f9ff65caf28e79864f14f1996e8b5ecd136959a 192.168.56.101:6382@16382 slave f65edec341cdc39ded3ecd5298c92f3192c8afcd 0 1nnected
>  5a595b9da405a27a097f9756008b47414e2da1f7 192.168.56.101:6383@16383 slave 1b55615b201a68db64db210381d9a4dbaaf6b17f 0 1nnected
>  522f0dca780073ab668bc80ded5a1489e2febe3e 192.168.56.101:6379@16379 myself,master - 0 1607262213000 1 connected 0-5460
>  192.168.56.101:6379> cluster info
>  cluster_state:ok
>  cluster_slots_assigned:16384
>  cluster_slots_ok:16384
>  cluster_slots_pfail:0
>  cluster_slots_fail:0
>  cluster_known_nodes:6
>  cluster_size:3
>  cluster_current_epoch:6
>  cluster_my_epoch:1
>  cluster_stats_messages_ping_sent:1493
>  cluster_stats_messages_pong_sent:1464
>  cluster_stats_messages_sent:2957
>  cluster_stats_messages_ping_received:1459
>  cluster_stats_messages_pong_received:1493
>  cluster_stats_messages_meet_received:5
>  cluster_stats_messages_received:2957
>   

 

# 参考文献：

1、https://blog.csdn.net/truelove12358/article/details/79612954

2、https://www.cnblogs.com/zhoujinyi/p/11606935.html

3、https://blog.csdn.net/miss1181248983/article/details/90056960