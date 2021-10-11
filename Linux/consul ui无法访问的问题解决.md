# consul ui无法访问的问题解决

在云服务器上启动了consul，但是访问不了8500端口，这里需要在服务器启动的时候，加上-client的指定

./consul agent -dev 这是最开始启动的命令 只能本机访问

当使用下面命令的时候就可以其他机器进行访问了：

./consul agent -dev -client 0.0.0.0 -ui

外部机器就可以访问
