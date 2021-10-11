# redis错误 ： Not all 16384 slots are covered by nodes.



转载自  [ http://blog.csdn.net/shudaqi2010/article/details/53164439](http://blog.csdn.net/shudaqi2010/article/details/53164439)

 早些时间公司[Redis](http://lib.csdn.net/base/redis)集群环境的某台机子冗机了，同时还导致了部分slot数据分片丢失；

 在用check检查集群运行状态时，遇到错误；

 

 **[root@node01 src]# ./[Redis](http://lib.csdn.net/base/redis)-trib.rb check172.168.63.202:7000**

 


 Connecting to node 172.168.63.202:7000: OK

 Connecting to node 172.168.63.203:7000: OK

 Connecting to node 172.168.63.201:7000: OK

 \>>> Performing Cluster Check(using node 172.168.63.202:7000)

 M: 449de2d2a4b799ceb858501b5b78ab91504c72e0172.168.63.202:7000

  slots: (0 slots) master

   0additional replica(s)

 M: db9d26b1d15889ad2950382f4f32639606f9a94b172.168.63.203:7000

  slots: (0 slots) master

   0additional replica(s)

 M: f90924f71308eb434038fc8a5f481d3661324792172.168.63.201:7000

  slots: (0 slots) master

   0additional replica(s)

 [OK] All nodes agree about slotsconfiguration.

 \>>> Check for open slots...

 \>>> Check slots coverage...

 **[ERR] Not all 16384 slots are covered by nodes.**

 **
** 

 

 

 原因：

 这个往往是由于主node移除了，但是并没有移除node上面的slot，从而导致了slot总数没有达到16384，其实也就是slots分布不正确。所**以在删除节点的时候一定要注意删除的是否是Master主节点。**

 1)、官方是推荐使用[redis](http://lib.csdn.net/base/redis)-trib.rb fix 来修复集群…. …. 通过cluster nodes看到7001这个节点被干掉了… 那么

 ***\*[root@node01 src]#\**  ./redis-trib.rb fix 172.168.63.201:7001**

 


 修复完成后再用check命令检查下是否正确

 **[root@node01 src]# ./redis-trib.rb check172.168.63.202:7000**


 只要输入任意集群中节点即可，会自动检查所有相关节点。可以查看相应的输出看下是否是每个Master都有了slots,如果分布不均匀那可以使用下面的方式重新分配slot:

 

 [root@node01 

src]#  ./redis-trib.rb reshard 172.168.63.201:7001