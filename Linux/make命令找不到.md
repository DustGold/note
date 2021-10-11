# CentOS下安装gcc后仍然提示bash: make: 未找到命令

今天在用linux安装redis的时候，安装了gcc和g++后执行“make”和“make distclean”命令仍然提示“bash: make: 未找到命令”。
 折腾了很久才发现是安装的时候直接使用了“yum install gcc”和“yum install gcc-g++”，系统安装的时候使用的是最小化mini安装，系统没有安装make、vim等常用命令。所以导致找不到命令。只要在使用以下命令安装即可：

```bash
yum -y install gcc automake autoconf libtool make
```

![在这里插入图片描述](C:%5Cnote%5CLinux%5Cmake%E5%91%BD%E4%BB%A4%E6%89%BE%E4%B8%8D%E5%88%B0%5Cimg%5C20201111130013837.png)

