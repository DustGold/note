日志 : /var/log/taos/

ts timestamp字段每个表必有，主键

testdemo 10条线程 3个字段



![image-20210722105821890](D:\note\TDengine\taos.assets\image-20210722105821890.png)

一个完整的 TDengine 系统是运行在一到多个物理节点上的，逻辑上，它包含数据节点(dnode)、TDengine应用驱动(taosc)以及应用(app)。系统中存在一到多个数据节点，这些数据节点组成一个集群(cluster)。应用通过taosc的API与TDengine集群进行互动。下面对每个逻辑单元进行简要介绍。

**物理节点(pnode):** pnode是一独立运行、拥有自己的计算、存储和网络能力的计算机，可以是安装有OS的物理机、虚拟机或Docker容器。物理节点由其配置的 FQDN(Fully Qualified Domain Name)来标识。TDengine完全依赖FQDN来进行网络通讯，如果不了解FQDN，请看博文[《一篇文章说清楚TDengine的FQDN》](https://www.taosdata.com/blog/2020/09/11/1824.html)。

**数据节点(dnode):** dnode 是 TDengine 服务器侧执行代码 taosd 在物理节点上的一个运行实例，一个工作的系统必须有至少一个数据节点。dnode包含零到多个逻辑的虚拟节点(VNODE)，零或者至多一个逻辑的管理节点(mnode)。dnode在系统中的唯一标识由实例的End Point (EP )决定。EP是dnode所在物理节点的FQDN (Fully Qualified Domain Name)和系统所配置的网络端口号(Port)的组合。通过配置不同的端口，一个物理节点(一台物理机、虚拟机或容器）可以运行多个实例，或有多个数据节点。

**虚拟节点(vnode)**: 为更好的支持数据分片、负载均衡，防止数据过热或倾斜，数据节点被虚拟化成多个虚拟节点(vnode，图中V2, V3, V4等)。每个 vnode 都是一个相对独立的工作单元，是时序数据存储的基本单元，具有独立的运行线程、内存空间与持久化存储的路径。一个 vnode 包含一定数量的表（数据采集点）。当创建一张新表时，系统会检查是否需要创建新的 vnode。一个数据节点上能创建的 vnode 的数量取决于该数据节点所在物理节点的硬件资源。一个 vnode 只属于一个DB，但一个DB可以有多个 vnode。一个 vnode 除存储的时序数据外，也保存有所包含的表的schema、标签值等。一个虚拟节点由所属的数据节点的EP，以及所属的VGroup ID在系统内唯一标识，由管理节点创建并管理。

**管理节点(mnode):** 一个虚拟的逻辑单元，负责所有数据节点运行状态的监控和维护，以及节点之间的负载均衡(图中M)。同时，管理节点也负责元数据(包括用户、数据库、表、静态标签等)的存储和管理，因此也称为 Meta Node。TDengine 集群中可配置多个(开源版最多不超过3个) mnode，它们自动构建成为一个虚拟管理节点组(图中M0, M1, M2)。mnode 间采用 master/slave 的机制进行管理，而且采取强一致方式进行数据同步, 任何数据更新操作只能在 Master 上进行。mnode 集群的创建由系统自动完成，无需人工干预。每个dnode上至多有一个mnode，由所属的数据节点的EP来唯一标识。每个dnode通过内部消息交互自动获取整个集群中所有 mnode 所在的 dnode 的EP。

**虚拟节点组(VGroup):** 不同数据节点上的 vnode 可以组成一个虚拟节点组(vnode group)来保证系统的高可靠。虚拟节点组内采取master/slave的方式进行管理。写操作只能在 master vnode 上进行，系统采用异步复制的方式将数据同步到 slave vnode，这样确保了一份数据在多个物理节点上有拷贝。一个 vgroup 里虚拟节点个数就是数据的副本数。如果一个DB的副本数为N，系统必须有至少N个数据节点。副本数在创建DB时通过参数 replica 可以指定，缺省为1。使用 TDengine 的多副本特性，可以不再需要昂贵的磁盘阵列等存储设备，就可以获得同样的数据高可靠性。虚拟节点组由管理节点创建、管理，并且由管理节点分配一个系统唯一的ID，VGroup ID。如果两个虚拟节点的vnode group ID相同，说明他们属于同一个组，数据互为备份。虚拟节点组里虚拟节点的个数是可以动态改变的，容许只有一个，也就是没有数据复制。VGroup ID是永远不变的，即使一个虚拟节点组被删除，它的ID也不会被收回重复利用。

**TAOSC:** taosc是TDengine给应用提供的驱动程序(driver)，负责处理应用与集群的接口交互，提供C/C++语言原生接口，内嵌于JDBC、C#、Python、Go、Node.js语言连接库里。应用都是通过taosc而不是直接连接集群中的数据节点与整个集群进行交互的。这个模块负责获取并缓存元数据；将插入、查询等请求转发到正确的数据节点；在把结果返回给应用时，还需要负责最后一级的聚合、排序、过滤等操作。对于JDBC, C/C++/C#/Python/Go/Node.js接口而言，这个模块是在应用所处的物理节点上运行。同时，为支持全分布式的RESTful接口，taosc在TDengine集群的每个dnode上都有一运行实例。





https://www.taosdata.com/assets-download/TDengine-server-2.0.20.10-Linux-x64.rpm