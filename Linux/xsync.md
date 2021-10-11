

在/usr/local/bin 目录下创建 xsync 文件，向里面添加：

```
#!/bin/sh
# 获取输入参数个数，如果没有参数，直接退出
pcount=$#
if((pcount==0)); then
        echo no args...;
        exit;
fi

# 获取文件名称
p1=$1
fname=`basename $p1`
echo fname=$fname
# 获取上级目录到绝对路径
pdir=`cd -P $(dirname $p1); pwd`
echo pdir=$pdir
# 获取当前用户名称
user=`whoami`
# 循环
for((host=1; host<=2; host++)); do
        echo $pdir/$fname $user@slave$host:$pdir
        echo ==================slave$host==================
        rsync -rvl $pdir/$fname $user@slave$host:$pdir
done
#Note:这里的slave对应自己主机名，需要做相应修改。另外，for循环中的host的边界值由自己的主机编号决定。123456789101112131415161718192021222324
```

最后chmod a+x xsync给文件添加执行权限即可。

使用xsync filename就能将filename分发到集群中的各个节点中。



## 附加

# rsync同步工具和xsync、xcall集群脚本



rsync rsync 远程同步工具，主要用于备份和镜像。具有速度快、避免复制相同内容和支持符号链接的优点。 rsync 和 scp 区别：用 rsync 做文件的复制要比 scp 的速度快，rsync 只对差异文件做更新。scp 是把所有文件都复制过去。 **（1） 查看rsync 使用说明** **man rsync | more** 

**（2） 基本语法** **rsync -rvl  $pdir/$fname   $user@hadoop$host:$pdir** 命令 命令参数 要拷贝的文件路径/名称  目的用户@主机:目的路径选项 -r 递归 -v 显示复制过程  -l 拷贝符号连接

**（3） 案例实操** 把本机/opt/tmp 目录同步到 hadoop03 服务器的 root 用户下的/opt/tmp 目录 **rsync -rvl /opt/tmp root@hadoop03:/opt/**