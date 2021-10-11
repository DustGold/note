Nacos首次启动报错

    版本信息
    报错信息
        db.num is null
            解决办法
        Unable to start embedded Tomcat
            解决办法

版本信息

nacos-server-2.0.0

    1

报错信息

在本地安装目录D:\devSoft\nacos\bin 下，cmd下执行 startup.cmd ，报错信息一共以下两个：
ps：我是在Windows10 系统下，如果是Linux系统，运行对应的sh命令即可。

    db.num is null

在这里插入图片描述
解决办法

	1、mysql新建库：nacos，字符集：utf8 ，排序规则：utf8_general_ci
	
	2、nacos\conf\ nacos-mysql.sql文件里的sql脚本执行到本机数据库的nacos库中
	
	3、nacos\conf\ application.properties 配置文件修改配置
	
	1
	2
	3
	4
	5

在这里插入图片描述

    Unable to start embedded Tomcat

解决办法

在这里插入图片描述
由于nacos默认启动是以集群方式启动，而本次我是以学习为主，所以就需要以单机方式启动，启动命令如下：

startup.cmd -m standalone

    1

另外：直接使用startup.cmd 命令启动默认是以集群方式启动，在开始运行的时候会打印如下信息。


