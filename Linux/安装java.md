#### 1、解压JDK到/opt/module目录下

```shell
tar -z(GZIP压缩方式)x（解压）v（看信息）f（制定压缩文件） jdk-8u212-linux-x64.tar.gz -C /opt/module/
```

#### 2、配置JDK环境变量

##### 1、新建/etc/profile.d/myenv.sh文件

```shell
sudo vim /etc/profile.d/my_env.sh
```

###### 添加以下内容

```shell
#JAVA_HOME
export JAVA_HOME=/opt/module/jdk1.8.0_271
export PATH=$PATH:$JAVA_HOME/bin
```



##### 2.保存后退出（:wq）

##### 3.重启shell窗口，让环境变量生效

# 附件

# 使用 /etc/profile.d 而不是 /etc/profile 来配置环境变量 Linux



在 /etc/profile 这个文件中有这么一段 shell, 会在每次启动时自动加载 profile.d 下的每个配置

```bash
if [ -d /etc/profile.d ]; then  for i in /etc/profile.d/*.sh; do    if [ -r $i ]; then      . $i    fi  done  unset ifi
```

### 区别

1、都用来设置环境变量文件

2、/etc/profile.d/ 高度解耦, 比 /etc/profile 好维护，不想要什么变量直接删除 /etc/profile.d/ 下对应的 shell 脚本即可

3、/etc/profile 和 /etc/profile.d 同样是登录（login）级别的变量，当用户重新登录 shell 时会触发。所以效果一致。

4、设置登录级别的变量，重新登录 shell，或者 source /etc/profile。

 

### 需要添加新的环境变量时:

在 /etc/profile.d/ 目录下新建对应的 sh 文件即可，比如新建 java 的：

vim /etc/profile.d/kafka.sh

export KAFKA_HOME=/opt/module/kafka

export PATH=$PATH:$KAFKA_HOME/bin

立即刷新使变量可用

回到上一次目录 source /etc/profile

查看 echo $KAFKA_HOME