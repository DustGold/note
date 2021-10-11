# 两种解决ERROR: Attempting to operate on hdfs namenode as root的方法        

描述：hadoop-3.1.0启动hadoop集群时还有可能可能会报如下错误

```shell
[root@localhost sbin]# start-all.sh
Starting namenodes on [hadoop]
ERROR: Attempting to operate on hdfs namenode as root
ERROR: but there is no HDFS_NAMENODE_USER defined. Aborting operation.
Starting datanodes
ERROR: Attempting to operate on hdfs datanode as root
ERROR: but there is no HDFS_DATANODE_USER defined. Aborting operation.
Starting secondary namenodes [hadoop]
ERROR: Attempting to operate on hdfs secondarynamenode as root
ERROR: but there is no HDFS_SECONDARYNAMENODE_USER defined. Aborting operation.
2018-07-16 05:45:04,628 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Starting resourcemanager
ERROR: Attempting to operate on yarn resourcemanager as root
ERROR: but there is no YARN_RESOURCEMANAGER_USER defined. Aborting operation.
Starting nodemanagers
ERROR: Attempting to operate on yarn nodemanager as root
ERROR: but there is no YARN_NODEMANAGER_USER defined. Aborting operation.
```

## 解决方案一：

输入如下命令，在环境变量中添加下面的配置

> vi /etc/profile

然后向里面加入如下的内容

```shell
export HDFS_NAMENODE_USER=root
export HDFS_DATANODE_USER=root
export HDFS_SECONDARYNAMENODE_USER=root
export YARN_RESOURCEMANAGER_USER=root
export YARN_NODEMANAGER_USER=root


export HDFS_NAMENODE_USER=peng
export HDFS_DATANODE_USER=peng
export HDFS_SECONDARYNAMENODE_USER=peng
export YARN_RESOURCEMANAGER_USER=peng
export YARN_NODEMANAGER_USER=peng
```

输入如下命令使改动生效

> source /etc/profile

## 解决方案二：

将start-dfs.sh，stop-dfs.sh(在hadoop安装目录的sbin里)两个文件顶部添加以下参数

```shell
HDFS_DATANODE_USER=root
HADOOP_SECURE_DN_USER=hdfs
HDFS_NAMENODE_USER=root
HDFS_SECONDARYNAMENODE_USER=root
```

将start-yarn.sh，stop-yarn.sh(在hadoop安装目录的sbin里)两个文件顶部添加以下参数

```shell
YARN_RESOURCEMANAGER_USER=root
HADOOP_SECURE_DN_USER=yarn
YARN_NODEMANAGER_USER=root
```

参考文章：
 https://blog.csdn.net/u013725455/article/details/70147331
 https://www.cnblogs.com/zhengna/p/9316424.html

**祝大家hadoop之旅顺利**