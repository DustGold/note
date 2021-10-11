# AppStream Error

## 问题描述

![1605947398771](C:%5Cnote%5Cdocker%5CAPPStream%E4%B8%8B%E8%BD%BD%E9%97%AE%E9%A2%98%5Cimg%5C1605947398771.png)

## 解决方法

1、检查是否可以连接外网  ping www.baidu.com  # 可以 ping 通百度

2、systemctl stop firewalld.service             # 停止防火墙 


 3、修改软件源

1. vim CentOS-Base.repo
2. vim CentOS-AppStream.repo
3. vim CentOS-Extras.repo

将文件中三文件中的 mirrorlist注释，将baseurl 修改为阿里源
 为 baseurl=https://mirrors.aliyun.com/centos/$releasever/BaseOS/$basearch/os/  

完成了以上三步，还是报同样的错：

> Error: Failed to download metadata for repo 'AppStream'
>
> Error: Failed to download metadata for repo 'AppStream'
>
> The command '/bin/sh -c yum -y install vim' returned a non-zero code: 1

4、下载阿里源：

> wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
>
> [root@localhost ~]# wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-8.repo

5、运行 yum makecache 生成缓存（或 yum clean all&&yum makecache ）

6、重启docker服务【 重要 】

​    systemctl restart docker

7、再执行命令（执行成功 ）：

> [root@localhost mydocker]# docker build -f /mydocker/Dockerfile2 -t mycentos:1.3 . 

![img](C:%5Cnote%5Cdocker%5CAPPStream%E4%B8%8B%E8%BD%BD%E9%97%AE%E9%A2%98%5Cimg%5C20200504114645182.png)

 

相关命令：

查询yum源  yum repolist

## 附录

CentOS 8更改了软件包的安装程序，取消了 yum 的配置方法，改而使用了dnf 作为安装程序。虽然改变了软件包的安装方式，但是  dnf 还是能兼容使用 yum  的配置文件的和命令的使用方法的。不过我并不知道这个兼容配置会持续多久和国内的镜像(这里使用的是阿里云镜像)路径是否会做修改，所以才在标题添加了临时标志。

这里也就不过多讲解了，直接上文件：

```
Copy# file: /etc/yum.repos.d/CentOS-AppStream.repo

[AppStream]
name=CentOS-$releasever - AppStream
baseurl=http://mirrors.aliyun.com/centos/$releasever/AppStream/$basearch/os/
gpgcheck=1
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
Copy# file: /etc/yum.repos.d/CentOS-Base.repo

[BaseOS]
name=CentOS-$releasever - Base
baseurl=http://mirrors.aliyun.com/centos/$releasever/BaseOS/$basearch/os/
gpgcheck=1
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
Copy# file: /etc/yum.repos.d/CentOS-Epel.repo

[epel]
name=CentOS-$releasever - Epel
baseurl=http://mirrors.aliyun.com/epel/8/Everything/$basearch
enabled=1
gpgcheck=0
```



