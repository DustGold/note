#### 配置JDK环境变量

##### 1、新建/etc/profile.d/myenv.sh文件

```shell
sudo vim /etc/profile.d/my_env.sh
```

###### 添加以下内容

```shell
##HADOOP_HOME
export HADOOP_HOME=/opt/module/hadoop-3.2.2
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
```

##### 2、重启shell

