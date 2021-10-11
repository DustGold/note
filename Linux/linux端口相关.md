# Linux 查看端口占用情况

netstat -anp|grep 端口号

lsof -i:端口号

netstat -tlnp | grep 进程号



netstat -anp | grep 进程号   （功能描述：查看该进程网络信息）

netstat –nlp | grep 端口号   （功能描述：查看网络端口号占用情况）

