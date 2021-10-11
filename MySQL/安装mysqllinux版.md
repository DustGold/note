#  MySQL 安装 

#### 1）检查当前系统是否安装过 MySQL

####  [atguigu@hadoop102 ~]$ rpm -qa|grep mariadb mariadb-libs-5.5.56-2.el7.x86_64 

####  //如果存在通过如下命令卸载 

#### [atguigu @hadoop102 ~]$ sudo rpm -e --nodeps  mariadb-libs 

#### 2）将 MySQL 安装包拷贝到/opt/software 目录下

####  [atguigu @hadoop102 software]# ll 总用量 528384 -rw-r--r--. 1 root root 609556480 3 月  21 15:41 mysql-5.7.281.el7.x86_64.rpm-bundle.tar

#### 3）解压 MySQL 安装包 

#### [atguigu @hadoop102 software]# tar -xf mysql-5.7.28-1.el7.x86_64.rpmbundle.tar 

#### 4）在安装目录下执行 rpm 安装 

```shell
sudo rpm -ivh mysql-community-common-5.7.34-1.el7.x86_64.rpm
sudo rpm -ivh mysql-community-libs-5.7.34-1.el7.x86_64.rpm
sudo rpm -ivh mysql-community-libs-compat-5.7.34-1.el7.x86_64.rpm --nodeps --force
sudo rpm -ivh mysql-community-client-5.7.34-1.el7.x86_64.rpm --nodeps --force
sudo rpm -ivh mysql-community-server-5.7.34-1.el7.x86_64.rpm  --nodeps --force

```

----

注意:按照顺序依次执行 
如果 Linux 是最小化安装的，在安装 mysql-community-server-5.7.28-1.el7.x86_64.rpm 时
可能会出现如下错误 [atguigu@hadoop102 software]$ sudo rpm -ivh mysql-community-server5.7.28-1.el7.x86_64.rpm 警告：mysql-community-server-5.7.28-1.el7.x86_64.rpm: 头 V3 DSA/SHA1 Signature, 密钥 ID 5072e1f5: NOKEY 
                           
错误：依赖检测失败：         libaio.so.1()(64bit) 被 mysql-community-server-5.7.28-1.el7.x86_64 需要         libaio.so.1(LIBAIO_0.1)(64bit) 被 mysql-community-server-5.7.281.el7.x86_64 需要         libaio.so.1(LIBAIO_0.4)(64bit) 被 mysql-community-server-5.7.281.el7.x86_64 需要 
通过 yum 安装缺少的依赖,然后重新安装 mysql-community-server-5.7.28-1.el7.x86_64 即
可 
[atguigu@hadoop102 software] yum install -y libaio 

----



#### 5）删除/etc/my.cnf 文件中 datadir 指向的目录下的所有内容,如果有内容的情况下: 
   查看 datadir 的值： [mysqld] datadir=/var/lib/mysql 
   删除/var/lib/mysql 目录下的所有内容: [atguigu @hadoop102 mysql]# cd /var/lib/mysql [atguigu @hadoop102 mysql]# sudo rm -rf ./*    //注意执行命令的位置 

#### 6）初始化数据库

####  [atguigu @hadoop102 opt]$ sudo mysqld --initialize --user=mysql 

#### 7）查看临时生成的 root 用户的密码

####  [atguigu @hadoop102 opt]$ sudo cat /var/log/mysqld.log  

#### 8）启动 MySQL 服务 

#### [atguigu @hadoop102 opt]$ sudo systemctl start mysqld 

#### 9）登录 MySQL 数据库

####  [atguigu @hadoop102 opt]$ mysql -uroot -p Enter password:   输入临时生成的密码 

   登录成功. 
#### 10）必须先修改 root 用户的密码,否则执行其他的操作会报错 
mysql> set password = password("新密码"); 

#### 11）修改 mysql 库下的 user 表中的 root 用户允许任意 ip 连接

 mysql> update mysql.user set host='%' where user='root'; mysql> flush privileges; 



## 常见错误

---

```shell
[root@Hadoop02 mysql]# mysql -uroot -p
mysql: error while loading shared libraries: libncurses.so.5: cannot open shared object file: No such file or directory
```

yum install libncurses*

---

