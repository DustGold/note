

- 笔记-基础篇-1(P1-P28)：[https://blog.csdn.net/hancoder/article/details/106922139](https://blog.csdn.net/hancoder/article/details/106922139)
- 笔记-基础篇-2(P28-P100)：[https://blog.csdn.net/hancoder/article/details/107612619](https://blog.csdn.net/hancoder/article/details/107612619)
- 笔记-高级篇(P340)：[https://blog.csdn.net/hancoder/article/details/107612746](https://blog.csdn.net/hancoder/article/details/107612746)
- 笔记-vue：[https://blog.csdn.net/hancoder/article/details/107007605](https://blog.csdn.net/hancoder/article/details/107007605)
- 笔记-elastic search、上架、检索：[https://blog.csdn.net/hancoder/article/details/113922398](https://blog.csdn.net/hancoder/article/details/113922398)
- 笔记-认证服务：[https://blog.csdn.net/hancoder/article/details/114242184](https://blog.csdn.net/hancoder/article/details/114242184)
- 笔记-分布式锁与缓存：[https://blog.csdn.net/hancoder/article/details/114004280](https://blog.csdn.net/hancoder/article/details/114004280)
- 笔记-集群篇：[https://blog.csdn.net/hancoder/article/details/107612802](https://blog.csdn.net/hancoder/article/details/107612802)
- springcloud笔记：[https://blog.csdn.net/hancoder/article/details/109063671](https://blog.csdn.net/hancoder/article/details/109063671)
- 笔记版本说明：2020年提供过笔记文档，但只有P1-P50的内容，2021年整理了P340的内容。请点击标题下面分栏查看系列笔记
- 声明：

  - 可以白嫖，但请勿转载发布，笔记手打不易
  - 本系列笔记不断迭代优化，csdn：hancoder上是最新版内容，10W字都是在csdn免费开放观看的。
  - 离线md笔记文件获取方式见文末。2021-3版本的md笔记打完压缩包共500k（云图床），包括本项目笔记，还有cloud、docker、mybatis-plus、rabbitMQ等个人相关笔记
- sql：[https://github.com/FermHan/gulimall/sql文件](https://github.com/FermHan/gulimall/sql文件)
- 本项目其他笔记见专栏：[https://blog.csdn.net/hancoder/category_10822407.html](https://blog.csdn.net/hancoder/category_10822407.html)

> 请直接ctrl+F搜索内容



## 一、ELASTIC SEARCH

### 0、简介

mysql用作持久化存储，ES用作检索

基本概念：`index库`>`type表`>`document文档`

1) index索引

动词：相当于mysql的insert

名词：相当于mysql的db

2) Type类型

在index中，可以定义一个或多个类型

类似于mysql的table，每一种类型的数据放在一起

3) Document文档

保存在某个index下，某种type的一个数据document，文档是json格式的，document就像是mysql中的某个table里面的内容。每一行对应的列叫属性

![](https://fermhan.oss-cn-qingdao.aliyuncs.com/img/20210219171735.png)

为什么ES搜索快？倒排索引

保存的记录

- 红海行动
- 探索红海行动
- 红海特别行动
- 红海记录片
- 特工红海特别探索

将内容分词就记录到索引中



| 词     | 记录      |
| ------ | --------- |
| 红海   | 1,2,3,4,5 |
| 行动   | 1,2,3     |
| 探索   | 2,5       |
| 特别   | 3,5       |
| 纪录片 | 4,        |
| 特工   | 5         |

检索：

1）、红海**特工**行动？查出后计算相关性得分：3号记录命中了2次，且3号本身才有3个单词，2/3，所以3号最匹配
2）、红海行动？

关系型数据库中两个数据表示是独立的，即使他们里面有相同名称的列也不影响使用，但ES中不是这样的。elasticsearch是基于Lucene开发的搜索引擎，而ES中不同type下名称相同的filed最终在Lucene中的处理方式是一样的。

- 两个不同type下的两个user_name，在ES同一个索引下其实被认为是同一个filed，你必须在两个不同的type中定义相同的filed映射。否则，不同type中的相同字段名称就会在处理中出现冲突的情况，导致Lucene处理效率下降。
- 去掉type就是为了提高ES处理数据的效率。
- Elasticsearch 7.xURL中的type参数为可选。比如，索引一个文档不再要求提供文档类型。
- Elasticsearch 8.x不再支持URL中的type参数。
  解决：将索引从多类型迁移到单类型，每种类型文档一个独立索引  

### 1、安装elastic search

 dokcer中安装elastic search

（1）下载ealastic search（存储和检索）和kibana（可视化检索）

```shell
docker pull elasticsearch:7.4.2
docker pull kibana:7.4.2
版本要统一
```

（2）配置

```shell
# 将docker里的目录挂载到linux的/mydata目录中
# 修改/mydata就可以改掉docker里的
mkdir -p /mydata/elasticsearch/config
mkdir -p /mydata/elasticsearch/data

# es可以被远程任何机器访问
echo "http.host: 0.0.0.0" >/mydata/elasticsearch/config/elasticsearch.yml

# 递归更改权限，es需要访问
chmod -R 777 /mydata/elasticsearch/
```



（3）启动Elastic search

```shell
# 9200是用户交互端口 9300是集群心跳端口
# -e指定是单阶段运行
# -e指定占用的内存大小，生产时可以设置32G
docker run --name elasticsearch -p 9200:9200 -p 9300:9300 \
-e  "discovery.type=single-node" \
-e ES_JAVA_OPTS="-Xms64m -Xmx512m" \
-v /mydata/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
-v /mydata/elasticsearch/data:/usr/share/elasticsearch/data \
-v  /mydata/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
-d elasticsearch:7.4.2 


# 设置开机启动elasticsearch
docker update elasticsearch --restart=always
```

> 因为容器里的文件映射到了外面，所以删除容器和新建容器数据还在

> 第一次查docker ps启动了，第二次查的时候发现关闭了，docker logs elasticsearch
>
> http://192.168.56.10:9200
>
> 数据挂载到外面，但是访问权限不足
>
> 把/mydata/elasticsearch下文件夹的权限设置好，上面已经设置过了



> 遇到了更新阿里源也下载不下来kibana镜像的情况，先在别的网络下载下来后传到vagrant中
>
> ```bash
> docker save -o kibana.tar kibana:7.4.2 
> 
> docker load -i kibana.tar 
> 
> # 如何通过其他工具链接ssh
> 
> 修改/etc/ssh/sshd_config
> 修改 PasswordAuthentication yes
> 
> systemctl restart sshd.service  或 service sshd restart
> 
> # 连接192.168.56.10:22端口成功，用户名root，密码vagrant
> 
> 也可以通过vagrant ssh-config查看ip和端口，此时是127.0.0.1:2222
> ```
>
> 在安装离线docker镜像的时候还提示内存不足，看了下是因为外部挂载的内存也算在了vagrant中，即使外部删了很多文件，vagrant中df -h硬盘占用率也不下降。我在外部删完文件后在内部又rm -rf XXX 强行接触占用

（4）启动kibana：

```shell
# kibana指定了了ES交互端口9200  # 5600位kibana主页端口
docker run --name kibana -e ELASTICSEARCH_HOSTS=http://192.168.56.10:9200 -p 5601:5601 -d kibana:7.4.2


# 设置开机启动kibana
docker update kibana  --restart=always
```

（5）测试

查看elasticsearch版本信息： http://192.168.56.10:9200

```json
{
    "name": "66718a266132",
    "cluster_name": "elasticsearch",
    "cluster_uuid": "xhDnsLynQ3WyRdYmQk5xhQ",
    "version": {
        "number": "7.4.2",
        "build_flavor": "default",
        "build_type": "docker",
        "build_hash": "2f90bbf7b93631e52bafb59b3b049cb44ec25e96",
        "build_date": "2019-10-28T20:40:44.881551Z",
        "build_snapshot": false,
        "lucene_version": "8.2.0",
        "minimum_wire_compatibility_version": "6.8.0",
        "minimum_index_compatibility_version": "6.0.0-beta1"
    },
    "tagline": "You Know, for Search"
}
```



显示elasticsearch 节点信息http://192.168.56.10:9200/_cat/nodes 

```json
127.0.0.1 14 99 25 0.29 0.40 0.22 dilm * 66718a266132

66718a266132代表上面的结点
*代表是主节点
```

##### kibana

访问Kibana： http://192.168.56.10:5601/app/kibana 





### 2、初步检索

#### 1）检索es信息

（1）`GET  /_cat/nodes`：查看所有节点

 如：http://192.168.56.10:9200/_cat/nodes

> 可以直接浏览器输入上面的url，也可以在kibana中输入`GET /_cat/nodes`

```
127.0.0.1 12 97 3 0.00 0.01 0.05 dilm * 66718a266132

66718a266132代表结点
*代表是主节点
```

（2）`GET  /_cat/health`：查看es健康状况

如： http://192.168.56.10:9200/_cat/health 

```
1613741055 13:24:15 elasticsearch green 1 1 0 0 0 0 0 0 - 100.0%
```

注：green表示健康值正常

（3）`GET  /_cat/master`：查看主节点

如： http://192.168.56.10:9200/_cat/master 

```
089F76WwSaiJcO6Crk7MpA 127.0.0.1 127.0.0.1 66718a266132

主节点唯一编号
虚拟机地址
```

（4）`GET/_cat/indicies`：查看所有索引 ，等价于mysql数据库的show databases;

如：http://192.168.56.10:9200/_cat/indices 

```json
green  open .kibana_task_manager_1   DhtDmKrsRDOUHPJm1EFVqQ 1 0 2 3 40.8kb 40.8kb
green  open .apm-agent-configuration vxzRbo9sQ1SvMtGkx6aAHQ 1 0 0 0   230b   230b
green  open .kibana_1                rdJ5pejQSKWjKxRtx-EIkQ 1 0 5 1 18.2kb 18.2kb

这3个索引是kibana创建的
```

####  2）新增文档

保存一个数据，保存在哪个索引的哪个类型下（哪张数据库哪张表下），保存时用唯一标识指定

```bash
# # 在customer索引下的external类型下保存1号数据
PUT customer/external/1

# POSTMAN输入
http://192.168.56.10:9200/customer/external/1

{
 "name":"John Doe"
}
```



##### ==PUT和POST区别==

- POST新增。如果不指定id，**会自动生成id**。指定id就会修改这个数据，并新增版本号；
  - 可以不指定id，不指定id时永远为创建
  - 指定不存在的id为创建
  - 指定存在的id为更新，而版本号会根据内容变没变而觉得版本号递增与否
  - 
- PUT可以新增也可以修改。**PUT必须指定id**；由于PUT需要指定id，我们一般用来做修改操作，不指定id会报错。
  - 必须指定id
  - 版本号总会增加
- 怎么记：put和java里map.put一样必须指定key-value。而post相当于mysql insert

> seq_no和version的区别：
>
> - 每个文档的版本号"`_version`" 起始值都为1 每次对当前文档成功操作后都加1
> - 而序列号"`_seq_no`"则可以看做是索引的信息 在第一次为索引插入数据时为0，**每对索引内数据操作成功一次`sqlNO`加1**， 并且文档会记录是第几次操作使它成为现在的情况的
>
> 可以参考https://www.cnblogs.com/Taeso/p/13363136.html

下面是在postman中的测试数据：

![](https://fermhan.oss-cn-qingdao.aliyuncs.com/img/20210219213722.png)



创建数据成功后，显示201 created表示插入记录成功。

```json
返回数据：
带有下划线开头的，称为元数据，反映了当前的基本信息。  
{
    "_index": "customer", 表明该数据在哪个数据库下；
    "_type": "external", 表明该数据在哪个类型下；
    "_id": "1",  表明被保存数据的id；
    "_version": 1,  被保存数据的版本
    "result": "created", 这里是创建了一条数据，如果重新put一条数据，则该状态会变为updated，并且版本号也会发生变化。
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 0,
    "_primary_term": 1
}
```



下面选用POST方式：

添加数据的时候，**不指定ID**，会自动的生成id，并且类型是新增：

```json
{
    "_index": "customer",
    "_type": "external",
    "_id": "5MIjvncBKdY1wAQm-wNZ",
    "_version": 1,
    "result": "created",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 11,
    "_primary_term": 6
}
```

再次使用POST插入数据，**不指定ID**，仍然是新增的：

```json
{
    "_index": "customer",
    "_type": "external",
    "_id": "5cIkvncBKdY1wAQmcQNk",
    "_version": 1,
    "result": "created",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 12,
    "_primary_term": 6
}
```

添加数据的时候，**指定ID**，会使用该id，并且类型是新增：

```json
{
    "_index": "customer",
    "_type": "external",
    "_id": "2",
    "_version": 1,
    "result": "created",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 13,
    "_primary_term": 6
}
```

再次使用POST插入数据，**指定同样的ID**，类型为updated

```json
{
    "_index": "customer",
    "_type": "external",
    "_id": "2",
    "_version": 2,
    "result": "updated",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 14,
    "_primary_term": 6
}
```

#### 3）查看文档

GET /customer/external/1

 http://192.168.56.10:9200/customer/external/1 

```json
{
    "_index": "customer",
    "_type": "external",
    "_id": "1",
    "_version": 10,
    "_seq_no": 18,//并发控制字段，每次更新都会+1，用来做乐观锁
    "_primary_term": 6,//同上，主分片重新分配，如重启，就会变化
    "found": true,
    "_source": {
        "name": "John Doe"
    }
}
```

> 乐观锁用法：通过“`if_seq_no=1&if_primary_term=1 `”，当序列号匹配的时候，才进行修改，否则不修改。

实例：将id=1的数据更新为name=1，然后再次更新为name=2，起始`1_seq_no=18，_primary_term=6`

 （1）将name更新为1

PUT  http://192.168.56.10:9200/customer/external/1?if_seq_no=18&if_primary_term=6

<img src="https://fermhan.oss-cn-qingdao.aliyuncs.com/img/20210220145320.png" style="zoom:80%;" />

 （2）将name更新为2，更新过程中使用seq_no=18

PUT  http://192.168.56.10:9200/customer/external/1?if_seq_no=18&if_primary_term=6

结果为：

```json
{
    "error": {
        "root_cause": [
            {
                "type": "version_conflict_engine_exception",
                "reason": "[1]: version conflict, required seqNo [18], primary term [6]. current document has seqNo [19] and primary term [6]",
                "index_uuid": "mG9XiCQISPmfBAmL1BPqIw",
                "shard": "0",
                "index": "customer"
            }
        ],
        "type": "version_conflict_engine_exception",
        "reason": "[1]: version conflict, required seqNo [18], primary term [6]. current document has seqNo [19] and primary term [6]",
        "index_uuid": "mG9XiCQISPmfBAmL1BPqIw",
        "shard": "0",
        "index": "customer"
    },
    "status": 409
}
```

出现更新错误。



（3）查询新的数据

 GET http://192.168.56.10:9200/customer/external/1 

```json
{
    "_index": "customer",
    "_type": "external",
    "_id": "1",
    "_version": 11,
    "_seq_no": 19,
    "_primary_term": 6,
    "found": true,
    "_source": {
        "name": "1"
    }
}
```

能够看到_seq_no变为19

（4）再次更新，更新成功

PUT http://192.168.56.10:9200/customer/external/1?if_seq_no=19&if_primary_term=1 

#### 4）更新文档_update

```json
POST customer/externel/1/_update
{
    "doc":{
        "name":"111"
    }
}
或者
POST customer/externel/1
{
    "doc":{
        "name":"222"
    }
}
或者
PUT customer/externel/1
{
    "doc":{
        "name":"222"
    }
}
```

 不同：带有update情况下

- ==POST操作会对比源文档数据==，如果相同不会有什么操作，文档version不增加。
- PUT操作总会重新保存并增加version版本

POST时带`_update`对比元数据如果一样就不进行任何操作。

看场景：

- 对于大并发更新，不带update
- 对于大并发查询偶尔更新，带update；对比更新，重新计算分配规则



（1）POST更新文档，带有_update

http://192.168.56.10:9200/customer/external/1/_update 

![](https://fermhan.oss-cn-qingdao.aliyuncs.com/img/20210220153320.png)

![](https://fermhan.oss-cn-qingdao.aliyuncs.com/img/20210220153512.png)



如果再次执行更新，则不执行任何操作，序列号也不发生变化

```json
返回
{
    "_index": "customer",
    "_type": "external",
    "_id": "1",
    "_version": 12,
    "result": "noop", // 无操作
    "_shards": {
        "total": 0,
        "successful": 0,
        "failed": 0
    },
    "_seq_no": 20,
    "_primary_term": 6
}
```

POST更新方式，会对比原来的数据，和原来的相同，则不执行任何操作（version和_seq_no）都不变。

 （2）POST更新文档，不带_update

在更新过程中，重复执行更新操作，数据也能够更新成功，不会和原来的数据进行对比。

```json
{
    "_index": "customer",
    "_type": "external",
    "_id": "1",
    "_version": 13,
    "result": "updated",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 21,
    "_primary_term": 6
}
```



#### 5）删除文档或索引

```
DELETE customer/external/1
DELETE customer
```

注：elasticsearch并没有提供删除类型的操作，只提供了删除索引和文档的操作。



实例：删除id=1的数据，删除后继续查询

DELETE http://192.168.56.10:9200/customer/external/1

```json
{
    "_index": "customer",
    "_type": "external",
    "_id": "1",
    "_version": 14,
    "result": "deleted",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 22,
    "_primary_term": 6
}
```

再次执行DELETE http://192.168.56.10:9200/customer/external/1

```json
{
    "_index": "customer",
    "_type": "external",
    "_id": "1",
    "_version": 15,
    "result": "not_found",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 23,
    "_primary_term": 6
}
```

GET http://192.168.56.10:9200/customer/external/1

```json
{
    "_index": "customer",
    "_type": "external",
    "_id": "1",
    "found": false
}
```

##### 删除索引

实例：删除整个costomer索引数据

删除前，所有的索引http://192.168.56.10:9200/_cat/indices

```
green  open .kibana_task_manager_1   DhtDmKrsRDOUHPJm1EFVqQ 1 0 2 0 31.3kb 31.3kb
green  open .apm-agent-configuration vxzRbo9sQ1SvMtGkx6aAHQ 1 0 0 0   283b   283b
green  open .kibana_1                rdJ5pejQSKWjKxRtx-EIkQ 1 0 8 3 28.8kb 28.8kb
yellow open customer                 mG9XiCQISPmfBAmL1BPqIw 1 1 9 1  8.6kb  8.6kb
```

删除“ customer ”索引

DELTE http://192.168.56.10:9200/customer

```json
响应
{
    "acknowledged": true
}
```

删除后，所有的索引http://192.168.56.10:9200/_cat/indices

```
green open .kibana_task_manager_1   DhtDmKrsRDOUHPJm1EFVqQ 1 0 2 0 31.3kb 31.3kb
green open .apm-agent-configuration vxzRbo9sQ1SvMtGkx6aAHQ 1 0 0 0   283b   283b
green open .kibana_1                rdJ5pejQSKWjKxRtx-EIkQ 1 0 8 3 28.8kb 28.8kb
```



#### 6）ES的批量操作——bulk

 匹配导入数据

POST http://192.168.56.10:9200/customer/external/_bulk

```json
两行为一个整体
{"index":{"_id":"1"}}
{"name":"a"}
{"index":{"_id":"2"}}
{"name":"b"}
注意格式json和text均不可，要去kibana里Dev Tools
```

语法格式：

```json
{action:{metadata}}\n
{request body  }\n

{action:{metadata}}\n
{request body  }\n
```

这里的批量操作，当发生某一条执行发生失败时，其他的数据仍然能够接着执行，也就是说彼此之间是独立的。

bulk api以此按顺序执行所有的action（动作）。如果一个单个的动作因任何原因失败，它将继续处理它后面剩余的动作。当bulk api返回时，它将提供每个动作的状态（与发送的顺序相同），所以您可以检查是否一个指定的动作是否失败了。

实例1: 执行多条数据


```json
POST /customer/external/_bulk
{"index":{"_id":"1"}}
{"name":"John Doe"}
{"index":{"_id":"2"}}
{"name":"John Doe"}
```

执行结果

```json
#! Deprecation: [types removal] Specifying types in bulk requests is deprecated.
{
  "took" : 318,  花费了多少ms
  "errors" : false, 没有发生任何错误
  "items" : [ 每个数据的结果
    {
      "index" : { 保存
        "_index" : "customer", 索引
        "_type" : "external", 类型
        "_id" : "1", 文档
        "_version" : 1, 版本
        "result" : "created", 创建
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 0,
        "_primary_term" : 1,
        "status" : 201 新建完成
      }
    },
    {
      "index" : { 第二条记录
        "_index" : "customer",
        "_type" : "external",
        "_id" : "2",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 1,
        "_primary_term" : 1,
        "status" : 201
      }
    }
  ]
}

```



实例2：对于整个索引执行批量操作

```json
POST /_bulk
{"delete":{"_index":"website","_type":"blog","_id":"123"}}
{"create":{"_index":"website","_type":"blog","_id":"123"}}
{"title":"my first blog post"}
{"index":{"_index":"website","_type":"blog"}}
{"title":"my second blog post"}
{"update":{"_index":"website","_type":"blog","_id":"123"}}
{"doc":{"title":"my updated blog post"}}
```

运行结果：

```json
#! Deprecation: [types removal] Specifying types in bulk requests is deprecated.
{
  "took" : 304,
  "errors" : false,
  "items" : [
    {
      "delete" : { 删除
        "_index" : "website",
        "_type" : "blog",
        "_id" : "123",
        "_version" : 1,
        "result" : "not_found", 没有该记录
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 0,
        "_primary_term" : 1,
        "status" : 404 没有该
      }
    },
    {
      "create" : {  创建
        "_index" : "website",
        "_type" : "blog",
        "_id" : "123",
        "_version" : 2,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 1,
        "_primary_term" : 1,
        "status" : 201
      }
    },
    {
      "index" : {  保存
        "_index" : "website",
        "_type" : "blog",
        "_id" : "5sKNvncBKdY1wAQmeQNo",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 2,
        "_primary_term" : 1,
        "status" : 201
      }
    },
    {
      "update" : { 更新
        "_index" : "website",
        "_type" : "blog",
        "_id" : "123",
        "_version" : 3,
        "result" : "updated",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 3,
        "_primary_term" : 1,
        "status" : 200
      }
    }
  ]
}
```



#### 7）样本测试数据

准备了一份顾客银行账户信息的虚构的JSON文档样本。每个文档都有下列的schema（模式）。

```json
{
	"account_number": 1,
	"balance": 39225,
	"firstname": "Amber",
	"lastname": "Duke",
	"age": 32,
	"gender": "M",
	"address": "880 Holmes Lane",
	"employer": "Pyrami",
	"email": "amberduke@pyrami.com",
	"city": "Brogan",
	"state": "IL"
}
```

 https://github.com/elastic/elasticsearch/blob/master/docs/src/test/resources/accounts.json ，导入测试数据，

```
POST bank/account/_bulk
上面的数据
```

```json
http://192.168.56.10:9200/_cat/indices
刚导入了1000条
yellow open bank                     99m64ElxRuiH46wV7RjXZA 1 1 1000 0 427.8kb 427.8kb
```



## 二、进阶检索

#### 3.1）search检索文档

ES支持两种基本方式检索；

* 通过REST request uri 发送搜索参数 （uri +检索参数）；
* 通过REST request body 来发送它们（uri+请求体）；

信息检索

API： https://www.elastic.co/guide/en/elasticsearch/reference/7.x/getting-started-search.html

```sh
请求参数方式检索
GET bank/_search?q=*&sort=account_number:asc
说明：
q=* # 查询所有
sort # 排序字段
asc #升序


检索bank下所有信息，包括type和docs
GET bank/_search
```

返回内容：

- `took` – 花费多少ms搜索
- `timed_out` – 是否超时 
- `_shards` – 多少分片被搜索了，以及多少成功/失败的搜索分片
- `max_score` –文档相关性最高得分 
- `hits.total.value` - 多少匹配文档被找到
- `hits.sort` - 结果的排序key（列），没有的话按照score排序
- `hits._score` - 相关得分 (not applicable when using `match_all`)

```json
GET bank/_search?q=*&sort=account_number:asc

检索了1000条数据，但是根据相关性算法，只返回10条
```

uri+请求体进行检索

```json
GET /bank/_search
{
  "query": { "match_all": {} },
  "sort": [
    { "account_number": "asc" },
    { "balance":"desc"}
  ]
}
```

POSTMAN中get不能携带请求体，我们变为post也是一样的，我们post一个jsob风格的查询请求体到_search

需要了解，一旦搜索的结果被返回，es就完成了这次请求，不能切不会维护任何服务端的资源或者结果的cursor游标

#### 3.2）DSL领域特定语言

> 这节教我们如何写复杂查询

Elasticsearch提供了一个可以执行查询的Json风格的DSL(domain-specific language领域特定语言)。这个被称为Query DSL，该查询语言非常全面。

##### （1）基本语法格式

一个查询语句的典型结构

```json
如果针对于某个字段，那么它的结构如下：
{
  QUERY_NAME:{   # 使用的功能
     FIELD_NAME:{  #  功能参数
       ARGUMENT:VALUE,
       ARGUMENT:VALUE,...
      }   
   }
}
```

```json
示例  使用时不要加#注释内容
GET bank/_search
{
  "query": {  #  查询的字段
    "match_all": {}
  },
  "from": 0,  # 从第几条文档开始查
  "size": 5,
  "_source":["balance"],
  "sort": [
    {
      "account_number": {  # 返回结果按哪个列排序
        "order": "desc"  # 降序
      }
    }
  ]
}
_source为要返回的字段
```

query定义如何查询；

- match_all查询类型【代表查询所有的索引】，es中可以在query中组合非常多的查询类型完成复杂查询；
- 除了query参数之外，我们可也传递其他的参数以改变查询结果，如sort，size；
- from+size限定，完成分页功能；
- sort排序，多字段排序，会在前序字段相等时后续字段内部排序，否则以前序为准；

##### （2）from返回部分字段

```json
GET bank/_search
{
  "query": {
    "match_all": {}
  },
  "from": 0,
  "size": 5,
  "sort": [
    {
      "account_number": {
        "order": "desc"
      }
    }
  ],
  "_source": ["balance","firstname"]
  
}
```

查询结果：

```json
{
  "took" : 18,  #   花了18ms
  "timed_out" : false,  # 没有超时
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1000,  # 命令1000条
      "relation" : "eq"   
    },
    "max_score" : null,
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "999",  # 第一条数据id是999
        "_score" : null,  # 得分信息
        "_source" : {
          "firstname" : "Dorothy",
          "balance" : 6087
        },
        "sort" : [  #  排序字段的值
          999
        ]
      },
      省略。。。
```



##### （3）query/match匹配查询

如果是非字符串，会进行精确匹配。如果是字符串，会进行全文检索

* 基本类型（非字符串），精确控制

```json
GET bank/_search
{
  "query": {
    "match": {
      "account_number": "20"
    }
  }
}
```

match返回account_number=20的数据。

查询结果：

```json
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,  // 得到一条
      "relation" : "eq"
    },
    "max_score" : 1.0,  # 最大得分
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "20",
        "_score" : 1.0,
        "_source" : {  # 该条文档信息
          "account_number" : 20,
          "balance" : 16418,
          "firstname" : "Elinor",
          "lastname" : "Ratliff",
          "age" : 36,
          "gender" : "M",
          "address" : "282 Kings Place",
          "employer" : "Scentric",
          "email" : "elinorratliff@scentric.com",
          "city" : "Ribera",
          "state" : "WA"
        }
      }
    ]
  }
}

```

* 字符串，全文检索

```json
GET bank/_search
{
  "query": {
    "match": {
      "address": "kings"
    }
  }
}
```

全文检索，最终会按照评分进行排序，会对检索条件进行分词匹配。

查询结果：

```json
{
  "took" : 30,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 5.990829,
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "20",
        "_score" : 5.990829,
        "_source" : {
          "account_number" : 20,
          "balance" : 16418,
          "firstname" : "Elinor",
          "lastname" : "Ratliff",
          "age" : 36,
          "gender" : "M",
          "address" : "282 Kings Place",
          "employer" : "Scentric",
          "email" : "elinorratliff@scentric.com",
          "city" : "Ribera",
          "state" : "WA"
        }
      },
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "722",
        "_score" : 5.990829,
        "_source" : {
          "account_number" : 722,
          "balance" : 27256,
          "firstname" : "Roberts",
          "lastname" : "Beasley",
          "age" : 34,
          "gender" : "F",
          "address" : "305 Kings Hwy",
          "employer" : "Quintity",
          "email" : "robertsbeasley@quintity.com",
          "city" : "Hayden",
          "state" : "PA"
        }
      }
    ]
  }
}

```



##### （4） `query/match_phrase` [不拆分匹配]

将需要匹配的值当成一整个单词（不分词）进行检索

- `match_phrase`：不拆分字符串进行检索
- `字段.keyword`：必须全匹配上才检索成功

> 前面的是包含mill或road就查出来，我们现在要**都包含**才查出

```json
GET bank/_search
{
  "query": {
    "match_phrase": {
      "address": "mill road"   #  就是说不要匹配只有mill或只有road的，要匹配mill road一整个子串
    }
  }
}
```

查处address中包含mill road的所有记录，并给出相关性得分

查看结果：

```json
{
  "took" : 32,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 8.926605,
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "970",
        "_score" : 8.926605,
        "_source" : {
          "account_number" : 970,
          "balance" : 19648,
          "firstname" : "Forbes",
          "lastname" : "Wallace",
          "age" : 28,
          "gender" : "M",
          "address" : "990 Mill Road", # "mill road"
          "employer" : "Pheast",
          "email" : "forbeswallace@pheast.com",
          "city" : "Lopezo",
          "state" : "AK"
        }
      }
    ]
  }
}

```

match_phrase和match的区别，观察如下实例：

```json
GET bank/_search
{
  "query": {
    "match_phrase": {
      "address": "990 Mill"
    }
  }
}
```

查询结果：

```json
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1, # 1
      "relation" : "eq"
    },
    "max_score" : 10.806405,
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "970",
        "_score" : 10.806405,
        "_source" : {
          "account_number" : 970,
          "balance" : 19648,
          "firstname" : "Forbes",
          "lastname" : "Wallace",
          "age" : 28,
          "gender" : "M",
          "address" : "990 Mill Road",  # "990 Mill"
          "employer" : "Pheast",
          "email" : "forbeswallace@pheast.com",
          "city" : "Lopezo",
          "state" : "AK"
        }
      }
    ]
  }
}

```



使用match的`keyword`

```json
GET bank/_search
{
  "query": {
    "match": {
      "address.keyword": "990 Mill"  # 字段后面加上 .keyword
    }
  }
}
```

查询结果，一条也未匹配到

```json
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 0, # 因为要求完全equal，所以匹配不到
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  }
}

```



修改匹配条件为“990 Mill Road”

```json
GET bank/_search
{
  "query": {
    "match": {
      "address.keyword": "990 Mill Road"  # 正好有这条文档，所以能匹配到
    }
  }
}
```

查询出一条数据

```json
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1, # 1
      "relation" : "eq"
    },
    "max_score" : 6.5032897,
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "970",
        "_score" : 6.5032897,
        "_source" : {
          "account_number" : 970,
          "balance" : 19648,
          "firstname" : "Forbes",
          "lastname" : "Wallace",
          "age" : 28,
          "gender" : "M",
          "address" : "990 Mill Road",  # equal
          "employer" : "Pheast",
          "email" : "forbeswallace@pheast.com",
          "city" : "Lopezo",
          "state" : "AK"
        }
      }
    ]
  }
}
```



文本字段的匹配，使用keyword，匹配的条件就是要显示字段的全部值，要进行精确匹配的。

match_phrase是做短语匹配，只要文本中包含匹配条件，就能匹配到。  

##### （5）query/multi_math【多字段匹配】

**state或者address中包含mill**，并且在查询过程中，会对于查询条件进行分词。

```json
GET bank/_search
{
  "query": {
    "multi_match": {  # 前面的match仅指定了一个字段。
      "query": "mill",
      "fields": [ # state和address有mill子串  不要求都有
        "state",
        "address"
      ]
    }
  }
}
```

查询结果：

```json
{
  "took" : 28,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 4,
      "relation" : "eq"
    },
    "max_score" : 5.4032025,
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "970",
        "_score" : 5.4032025,
        "_source" : {
          "account_number" : 970,
          "balance" : 19648,
          "firstname" : "Forbes",
          "lastname" : "Wallace",
          "age" : 28,
          "gender" : "M",
          "address" : "990 Mill Road",  # 有mill
          "employer" : "Pheast",
          "email" : "forbeswallace@pheast.com",
          "city" : "Lopezo",
          "state" : "AK"  # 没有mill
        }
      },
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "136",
        "_score" : 5.4032025,
        "_source" : {
          "account_number" : 136,
          "balance" : 45801,
          "firstname" : "Winnie",
          "lastname" : "Holland",
          "age" : 38,
          "gender" : "M",
          "address" : "198 Mill Lane", # mill
          "employer" : "Neteria",
          "email" : "winnieholland@neteria.com",
          "city" : "Urie",
          "state" : "IL"  # 没有mill
        }
      },
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "345",
        "_score" : 5.4032025,
        "_source" : {
          "account_number" : 345,
          "balance" : 9812,
          "firstname" : "Parker",
          "lastname" : "Hines",
          "age" : 38,
          "gender" : "M",
          "address" : "715 Mill Avenue",  # 
          "employer" : "Baluba",
          "email" : "parkerhines@baluba.com",
          "city" : "Blackgum",
          "state" : "KY"  # 没有mill
        }
      },
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "472",
        "_score" : 5.4032025,
        "_source" : {
          "account_number" : 472,
          "balance" : 25571,
          "firstname" : "Lee",
          "lastname" : "Long",
          "age" : 32,
          "gender" : "F",
          "address" : "288 Mill Street", #
          "employer" : "Comverges",
          "email" : "leelong@comverges.com",
          "city" : "Movico",
          "state" : "MT" # 没有mill
        }
      }
    ]
  }
}

```



##### （6）query/bool/must复合查询

复合语句可以合并，任何其他查询语句，包括符合语句。这也就意味着，复合语句之间可以互相嵌套，可以表达非常复杂的逻辑。

- must：必须达到must所列举的所有条件
- must_not：必须不匹配must_not所列举的所有条件。
- should：应该满足should所列举的条件。满足条件最好，不满足也可以，**满足得分更高**

实例：查询gender=m，并且address=mill的数据

```json
GET bank/_search
{
   "query":{
        "bool":{  # 
             "must":[ # 必须有这些字段
              {"match":{"address":"mill"}},
              {"match":{"gender":"M"}}
             ]
         }
    }
}
```

查询结果：

```json
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : 6.0824604,
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "970",
        "_score" : 6.0824604,
        "_source" : {
          "account_number" : 970,
          "balance" : 19648,
          "firstname" : "Forbes",
          "lastname" : "Wallace",
          "age" : 28,
          "gender" : "M",  # M
          "address" : "990 Mill Road", # mill
          "employer" : "Pheast",
          "email" : "forbeswallace@pheast.com",
          "city" : "Lopezo",
          "state" : "AK"
        }
      },
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "136",
        "_score" : 6.0824604,
        "_source" : {
          "account_number" : 136,
          "balance" : 45801,
          "firstname" : "Winnie",
          "lastname" : "Holland",
          "age" : 38,
          "gender" : "M", # 
          "address" : "198 Mill Lane", # 
          "employer" : "Neteria",
          "email" : "winnieholland@neteria.com",
          "city" : "Urie",
          "state" : "IL"
        }
      },
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "345",
        "_score" : 6.0824604,
        "_source" : {
          "account_number" : 345,
          "balance" : 9812,
          "firstname" : "Parker",
          "lastname" : "Hines",
          "age" : 38,
          "gender" : "M",  # 
          "address" : "715 Mill Avenue",  # 
          "employer" : "Baluba",
          "email" : "parkerhines@baluba.com",
          "city" : "Blackgum",
          "state" : "KY"
        }
      }
    ]
  }
}
```



**must_not：必须不是指定的情况**

实例：查询gender=m，并且address=mill的数据，但是age不等于38的

```json
GET bank/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "gender": "M" }},
        { "match": {"address": "mill"}}
      ],
      "must_not": [  # 不可以是指定值
        { "match": { "age": "38" }}
      ]
   }
}
```

查询结果：

```json
{
  "took" : 4,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 6.0824604,
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "970",
        "_score" : 6.0824604,
        "_source" : {
          "account_number" : 970,
          "balance" : 19648,
          "firstname" : "Forbes",
          "lastname" : "Wallace",
          "age" : 28, # 不是38
          "gender" : "M", #
          "address" : "990 Mill Road", #
          "employer" : "Pheast",
          "email" : "forbeswallace@pheast.com",
          "city" : "Lopezo",
          "state" : "AK" 
        }
      }
    ]
  }
}
```



**should：应该达到should列举的条件，如果到达会增加相关文档的评分，并不会改变查询的结果。如果query中只有should且只有一种匹配规则，那么should的条件就会被作为默认匹配条件二区改变查询结果。**

实例：匹配lastName应该等于Wallace的数据

```json
GET bank/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "gender": "M"
          }
        },
        {
          "match": {
            "address": "mill"
          }
        }
      ],
      "must_not": [
        {
          "match": {
            "age": "18"
          }
        }
      ],
      "should": [
        {
          "match": {
            "lastname": "Wallace"
          }
        }
      ]
    }
  }
}
```

查询结果：

```json
{
  "took" : 5,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : 12.585751,
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "970",
        "_score" : 12.585751,
        "_source" : {
          "account_number" : 970,
          "balance" : 19648,
          "firstname" : "Forbes",
          "lastname" : "Wallace",  # 因为匹配了should，所以得分第一
          "age" : 28, # 不是18
          "gender" : "M",  # 
          "address" : "990 Mill Road",  # 
          "employer" : "Pheast",
          "email" : "forbeswallace@pheast.com",
          "city" : "Lopezo",
          "state" : "AK"
        }
      },
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "136",
        "_score" : 6.0824604,
        "_source" : {
          "account_number" : 136,
          "balance" : 45801,
          "firstname" : "Winnie",
          "lastname" : "Holland",
          "age" : 38,
          "gender" : "M",
          "address" : "198 Mill Lane",
          "employer" : "Neteria",
          "email" : "winnieholland@neteria.com",
          "city" : "Urie",
          "state" : "IL"
        }
      },
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "345",
        "_score" : 6.0824604,
        "_source" : {
          "account_number" : 345,
          "balance" : 9812,
          "firstname" : "Parker",
          "lastname" : "Hines",
          "age" : 38,
          "gender" : "M",
          "address" : "715 Mill Avenue",
          "employer" : "Baluba",
          "email" : "parkerhines@baluba.com",
          "city" : "Blackgum",
          "state" : "KY"
        }
      }
    ]
  }
}

```

能够看到相关度越高，得分也越高。

##### （7）query/filter【结果过滤】

- must 贡献得分
- should 贡献得分
- must_not 不贡献得分
- filter 不贡献得分

> 上面的must和should影响相关性得分，而must_not仅仅是一个filter ，不贡献得分
>
> must改为filter就使must不贡献得分
>
> 如果只有filter条件的话，我们会发现得分都是0
>
> 一个key多个值可以用terms

并不是所有的查询都需要产生分数，特别是哪些仅用于filtering过滤的文档。为了不计算分数，elasticsearch会自动检查场景并且优化查询的执行。

不参与评分更快

```json
GET bank/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": {"address": "mill" } }
      ],
      "filter": {  # query.bool.filter
        "range": {
          "balance": {  # 哪个字段
            "gte": "10000",
            "lte": "20000"
          }
        }
      }
    }
  }
}

```

这里先是查询所有匹配address=mill的文档，然后再根据10000<=balance<=20000进行过滤查询结果

查询结果：

```json
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 5.4032025,
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "970",
        "_score" : 5.4032025,
        "_source" : {
          "account_number" : 970,
          "balance" : 19648,  # 1W到2W之间
          "firstname" : "Forbes",
          "lastname" : "Wallace",
          "age" : 28,
          "gender" : "M",
          "address" : "990 Mill Road", # 
          "employer" : "Pheast",
          "email" : "forbeswallace@pheast.com",
          "city" : "Lopezo",
          "state" : "AK"
        }
      }
    ]
  }
}

```



Each `must`, `should`, and `must_not` element in a Boolean query is referred to as a query clause. How well a document meets the criteria in each `must` or `should` clause contributes to the document’s *relevance score*. The higher the score, the better the document matches your search criteria. By default, Elasticsearch returns documents ranked by these relevance scores.

 在boolean查询中，`must`, `should` 和`must_not` 元素都被称为查询子句 。 文档是否符合每个“must”或“should”子句中的标准，决定了文档的“相关性得分”。  得分越高，文档越符合您的搜索条件。  默认情况下，Elasticsearch返回根据这些相关性得分排序的文档。 

The criteria in a `must_not` clause is treated as a *filter*. It affects whether or not the document is included in the results, but does not contribute to how documents are scored. You can also explicitly specify arbitrary filters to include or exclude documents based on structured data.

`“must_not”子句中的条件被视为“过滤器”。` 它影响文档是否包含在结果中，  但不影响文档的评分方式。  还可以显式地指定任意过滤器来包含或排除基于结构化数据的文档。 



filter在使用过程中，并不会计算相关性得分：

```json
GET bank/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "address": "mill"
          }
        }
      ],
      "filter": {
        "range": {
          "balance": {
            "gte": "10000",
            "lte": "20000"
          }
        }
      }
    }
  }
}
```

查询结果：

```json
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 213,
      "relation" : "eq"
    },
    "max_score" : 0.0,
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "20",
        "_score" : 0.0,
        "_source" : {
          "account_number" : 20,
          "balance" : 16418,
          "firstname" : "Elinor",
          "lastname" : "Ratliff",
          "age" : 36,
          "gender" : "M",
          "address" : "282 Kings Place",
          "employer" : "Scentric",
          "email" : "elinorratliff@scentric.com",
          "city" : "Ribera",
          "state" : "WA"
        }
      },
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "37",
        "_score" : 0.0,
        "_source" : {
          "account_number" : 37,
          "balance" : 18612,
          "firstname" : "Mcgee",
          "lastname" : "Mooney",
          "age" : 39,
          "gender" : "M",
          "address" : "826 Fillmore Place",
          "employer" : "Reversus",
          "email" : "mcgeemooney@reversus.com",
          "city" : "Tooleville",
          "state" : "OK"
        }
      },
        省略。。。
```

**能看到所有文档的 "_score" : 0.0。**

##### （8）query/term

和match一样。匹配某个属性的值。

- 全文检索字段用match，
- 其他**非text字段**匹配用term。

不要使用term来进行文本字段查询

es默认存储text值时用分词分析，所以要搜索text值，使用match

https://www.elastic.co/guide/en/elasticsearch/reference/7.6/query-dsl-term-query.html 

- 字段.keyword：要一一匹配到
- match_phrase：子串包含即可

使用term匹配查询

```json
GET bank/_search
{
  "query": {
    "term": {
      "address": "mill Road"
    }
  }
}
```

查询结果：

```json
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 0, # 没有
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  }
}

```

一条也没有匹配到

而更换为match匹配时，能够匹配到32个文档

```json
{
  "took" : 5,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 32,
      "relation" : "eq"
    },
    "max_score" : 8.926605,
    "hits" : [
```

也就是说，**全文检索字段用match，其他非text字段匹配用term**。

##### （9）aggs/agg1（聚合）

> 前面介绍了存储、检索，但还没介绍分析

聚合提供了从数据中分组和提取数据的能力。最简单的聚合方法大致等于SQL `Group by`和SQL`聚合函数`。

在elasticsearch中，==执行搜索返回this（命中结果），并且同时返回聚合结果==，把以响应中的所有hits（命中结果）分隔开的能力。这是非常强大且有效的，你可以执行查询和多个聚合，并且在一次使用中得到各自的（任何一个的）返回结果，使用一次简洁和简化的API啦避免网络往返。

aggs：执行聚合。聚合语法如下：

```json
"aggs":{ # 聚合
    "aggs_name":{ # 这次聚合的名字，方便展示在结果集中
        "AGG_TYPE":{} # 聚合的类型(avg,term,terms)
     }
}
```

- terms：看值的可能性分布，会合并锁查字段，给出计数即可
- avg：看值的分布平均

例：**搜索address中包含mill的所有人的年龄分布以及平均年龄，但不显示这些人的详情**


```json
# 分别为包含mill、，平均年龄、
GET bank/_search
{
  "query": { # 查询出包含mill的
    "match": {
      "address": "Mill"
    }
  },
  "aggs": { #基于查询聚合
    "ageAgg": {  # 聚合的名字，随便起
      "terms": { # 看值的可能性分布
        "field": "age",
        "size": 10
      }
    },
    "ageAvg": { 
      "avg": { # 看age值的平均
        "field": "age"
      }
    },
    "balanceAvg": {
      "avg": { # 看balance的平均
        "field": "balance"
      }
    }
  },
  "size": 0  # 不看详情
}
```

查询结果：

```json
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 4, // 命中4条
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "ageAgg" : { // 第一个聚合的结果
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : 38,  # age为38的有2条
          "doc_count" : 2
        },
        {
          "key" : 28,
          "doc_count" : 1
        },
        {
          "key" : 32,
          "doc_count" : 1
        }
      ]
    },
    "ageAvg" : { // 第二个聚合的结果
      "value" : 34.0  # balance字段的平均值是34
    },
    "balanceAvg" : {
      "value" : 25208.0
    }
  }
}

```

###### aggs/aggName/aggs/aggName子聚合

复杂：
按照年龄聚合，并且求这些年龄段的这些人的平均薪资

> 写到一个聚合里是基于上个聚合进行子聚合。
>
> 下面求每个age分布的平均balance

```json
GET bank/_search
{
  "query": {
    "match_all": {}
  },
  "aggs": {
    "ageAgg": {
      "terms": { # 看分布
        "field": "age",
        "size": 100
      },
      "aggs": { # 与terms并列
        "ageAvg": { #平均
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  },
  "size": 0
}
```

输出结果：

```json
{
  "took" : 49,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1000,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "ageAgg" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : 31,
          "doc_count" : 61,
          "ageAvg" : {
            "value" : 28312.918032786885
          }
        },
        {
          "key" : 39,
          "doc_count" : 60,
          "ageAvg" : {
            "value" : 25269.583333333332
          }
        },
        {
          "key" : 26,
          "doc_count" : 59,
          "ageAvg" : {
            "value" : 23194.813559322032
          }
        },
        {
          "key" : 32,
          "doc_count" : 52,
          "ageAvg" : {
            "value" : 23951.346153846152
          }
        },
        {
          "key" : 35,
          "doc_count" : 52,
          "ageAvg" : {
            "value" : 22136.69230769231
          }
        },
        {
          "key" : 36,
          "doc_count" : 52,
          "ageAvg" : {
            "value" : 22174.71153846154
          }
        },
        {
          "key" : 22,
          "doc_count" : 51,
          "ageAvg" : {
            "value" : 24731.07843137255
          }
        },
        {
          "key" : 28,
          "doc_count" : 51,
          "ageAvg" : {
            "value" : 28273.882352941175
          }
        },
        {
          "key" : 33,
          "doc_count" : 50,
          "ageAvg" : {
            "value" : 25093.94
          }
        },
        {
          "key" : 34,
          "doc_count" : 49,
          "ageAvg" : {
            "value" : 26809.95918367347
          }
        },
        {
          "key" : 30,
          "doc_count" : 47,
          "ageAvg" : {
            "value" : 22841.106382978724
          }
        },
        {
          "key" : 21,
          "doc_count" : 46,
          "ageAvg" : {
            "value" : 26981.434782608696
          }
        },
        {
          "key" : 40,
          "doc_count" : 45,
          "ageAvg" : {
            "value" : 27183.17777777778
          }
        },
        {
          "key" : 20,
          "doc_count" : 44,
          "ageAvg" : {
            "value" : 27741.227272727272
          }
        },
        {
          "key" : 23,
          "doc_count" : 42,
          "ageAvg" : {
            "value" : 27314.214285714286
          }
        },
        {
          "key" : 24,
          "doc_count" : 42,
          "ageAvg" : {
            "value" : 28519.04761904762
          }
        },
        {
          "key" : 25,
          "doc_count" : 42,
          "ageAvg" : {
            "value" : 27445.214285714286
          }
        },
        {
          "key" : 37,
          "doc_count" : 42,
          "ageAvg" : {
            "value" : 27022.261904761905
          }
        },
        {
          "key" : 27,
          "doc_count" : 39,
          "ageAvg" : {
            "value" : 21471.871794871793
          }
        },
        {
          "key" : 38,
          "doc_count" : 39,
          "ageAvg" : {
            "value" : 26187.17948717949
          }
        },
        {
          "key" : 29,
          "doc_count" : 35,
          "ageAvg" : {
            "value" : 29483.14285714286
          }
        }
      ]
    }
  }
}
```

复杂子聚合：查出所有年龄分布，并且这些**年龄段**中M的平均薪资和F的平均薪资以及这个年龄段的总体平均薪资

```json
GET bank/_search
{
  "query": {
    "match_all": {}
  },
  "aggs": {
    "ageAgg": {
      "terms": {  #  看age分布
        "field": "age",
        "size": 100
      },
      "aggs": { # 子聚合
        "genderAgg": {
          "terms": { # 看gender分布
            "field": "gender.keyword" # 注意这里，文本字段应该用.keyword
          },
          "aggs": { # 子聚合
            "balanceAvg": {
              "avg": { # 男性的平均
                "field": "balance"
              }
            }
          }
        },
        "ageBalanceAvg": {
          "avg": { #age分布的平均（男女）
            "field": "balance"
          }
        }
      }
    }
  },
  "size": 0
}
```

输出结果：

```json
{
  "took" : 119,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1000,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "ageAgg" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : 31,
          "doc_count" : 61,
          "genderAgg" : {
            "doc_count_error_upper_bound" : 0,
            "sum_other_doc_count" : 0,
            "buckets" : [
              {
                "key" : "M",
                "doc_count" : 35,
                "balanceAvg" : {
                  "value" : 29565.628571428573
                }
              },
              {
                "key" : "F",
                "doc_count" : 26,
                "balanceAvg" : {
                  "value" : 26626.576923076922
                }
              }
            ]
          },
          "ageBalanceAvg" : {
            "value" : 28312.918032786885
          }
        }
      ]
        .......//省略其他
    }
  }
}

```

##### nested对象聚合

属性是"type": "nested",因为是内部的属性进行检索

数组类型的对象会被扁平化处理（对象的每个属性会分别存储到一起）

```json
user.name=["aaa","bbb"]
user.addr=["ccc","ddd"]

这种存储方式，可能会发生如下错误：
错误检索到{aaa,ddd}，这个组合是不存在的
```

数组的扁平化处理会使检索能检索到本身不存在的，为了解决这个问题，就采用了嵌入式属性，数组里是对象时用嵌入式属性（不是对象无需用嵌入式属性）

nested阅读：https://blog.csdn.net/weixin_40341116/article/details/80778599

使用聚合：https://blog.csdn.net/kabike/article/details/101460578

```json
GET articles/_search
{
  "size": 0, 
  "aggs": {
    "nested": { # 
      "nested": { #
        "path": "payment"
      },
      "aggs": {
        "amount_avg": {
          "avg": {
            "field": "payment.amount"
          }
        }
      }
    }
  }
}
```

## 三、Mapping字段映射

映射定义文档如何被存储和检索的

##### （1）字段类型

https://www.elastic.co/guide/en/elasticsearch/reference/7.x/mapping-types.html

- 核心类型
- 复合类型
- 地理类型
- 特定类型

核心数据类型

（1）字符串

- `text` ⽤于全⽂索引，搜索时会自动使用分词器进⾏分词再匹配
- `keyword` 不分词，搜索时需要匹配完整的值

（2）数值型

- 整型： byte，short，integer，long
- 浮点型： float, half_float, scaled_float，double 

（3）日期类型：date

（4）范围型

integer_range， long_range， float_range，double_range，date_range

gt是大于，lt是小于，e是equals等于。

age_limit的区间包含了此值的文档都算是匹配。

（5）布尔

- boolean  

（6）二进制

- binary  会把值当做经过 base64 编码的字符串，默认不存储，且不可搜索

复杂数据类型

（1）对象

- object一个对象中可以嵌套对象。

（2）数组

- Array

嵌套类型

- nested 用于json对象数组

![image-20200502161339291](https://fermhan.oss-cn-qingdao.aliyuncs.com/guli/image-20200502161339291.png)



##### （2）映射

Mapping(映射)是用来定义一个文档（document），以及它所包含的属性（field）是如何存储和索引的。比如：使用maping来定义：

* 哪些字符串属性应该被看做全文本属性（full text fields）；

* 哪些属性包含数字，日期或地理位置；

* 文档中的所有属性是否都嫩被索引（all 配置）；

* 日期的格式；

* 自定义映射规则来执行动态添加属性；

* 查看mapping信息：GET bank/_mapping
  

  
```json
  {
    "bank" : {
      "mappings" : {
        "properties" : {
          "account_number" : {
            "type" : "long" # long类型
          },
          "address" : {
            "type" : "text", # 文本类型，会进行全文检索，进行分词
            "fields" : {
              "keyword" : { # addrss.keyword
                "type" : "keyword",  # 该字段必须全部匹配到
                "ignore_above" : 256
              }
            }
          },
          "age" : {
            "type" : "long"
          },
          "balance" : {
            "type" : "long"
          },
          "city" : {
            "type" : "text",
            "fields" : {
              "keyword" : {
                "type" : "keyword",
                "ignore_above" : 256
              }
            }
          },
          "email" : {
            "type" : "text",
            "fields" : {
              "keyword" : {
                "type" : "keyword",
                "ignore_above" : 256
              }
            }
          },
          "employer" : {
            "type" : "text",
            "fields" : {
              "keyword" : {
                "type" : "keyword",
                "ignore_above" : 256
              }
            }
          },
          "firstname" : {
            "type" : "text",
            "fields" : {
              "keyword" : {
                "type" : "keyword",
                "ignore_above" : 256
              }
            }
          },
          "gender" : {
            "type" : "text",
            "fields" : {
              "keyword" : {
                "type" : "keyword",
                "ignore_above" : 256
              }
            }
          },
          "lastname" : {
            "type" : "text",
            "fields" : {
              "keyword" : {
                "type" : "keyword",
                "ignore_above" : 256
              }
            }
          },
          "state" : {
            "type" : "text",
            "fields" : {
              "keyword" : {
                "type" : "keyword",
                "ignore_above" : 256
              }
            }
          }
        }
      }
    }
  }
```




* 修改mapping信息

![image-20200502170924399](https://fermhan.oss-cn-qingdao.aliyuncs.com/guli/image-20200502170924399.png)





##### （3）新版本改变

ElasticSearch7-去掉type概念

1. 关系型数据库中两个数据表示是独立的，即使他们里面有相同名称的列也不影响使用，但ES中不是这样的。elasticsearch是基于Lucene开发的搜索引擎，而ES中不同type下名称相同的filed最终在Lucene中的处理方式是一样的。

   - 两个不同type下的两个user_name，在ES同一个索引下其实被认为是同一个filed，你必须在两个不同的type中定义相同的filed映射。否则，不同type中的相同字段名称就会在处理中出现冲突的情况，导致Lucene处理效率下降。
   - 去掉type就是为了提高ES处理数据的效率。

2. Elasticsearch 7.x URL中的type参数为可选。比如，索引一个文档不再要求提供文档类型。

3. Elasticsearch 8.x **不再支持URL中的type参数**。

4. 解决：
   将索引从多类型迁移到单类型，每种类型文档一个独立索引

   将已存在的索引下的类型数据，全部迁移到指定位置即可。详见数据迁移

>
>
>**Elasticsearch 7.x**
>
>- Specifying types in requests is deprecated. For instance, indexing a document no longer requires a document `type`. The new index APIs are `PUT {index}/_doc/{id}` in case of explicit ids and `POST {index}/_doc` for auto-generated ids. Note that in 7.0, `_doc` is a permanent part of the path, and represents the endpoint name rather than the document type.
>- The `include_type_name` parameter in the index creation, index template, and mapping APIs will default to `false`. Setting the parameter at all will result in a deprecation warning.
>- The `_default_` mapping type is removed.
>
>**Elasticsearch 8.x**
>
>- Specifying types in requests is no longer supported.
>- The `include_type_name` parameter is removed.

###### 创建映射`PUT /my_index`

> 第一次存储数据的时候es就猜出了映射
>
> 第一次存储数据前可以指定映射

创建索引并指定映射

```json
PUT /my_index
{
  "mappings": {
    "properties": {
      "age": {
        "type": "integer"
      },
      "email": {
        "type": "keyword" # 指定为keyword
      },
      "name": {
        "type": "text" # 全文检索。保存时候分词，检索时候进行分词匹配
      }
    }
  }
}
```

 输出：

```json
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "my_index"
}
```

###### 查看映射`GET /my_index`

```json
GET /my_index
```

输出结果：

```json
{
  "my_index" : {
    "aliases" : { },
    "mappings" : {
      "properties" : {
        "age" : {
          "type" : "integer"
        },
        "email" : {
          "type" : "keyword"
        },
        "employee-id" : {
          "type" : "keyword",
          "index" : false
        },
        "name" : {
          "type" : "text"
        }
      }
    },
    "settings" : {
      "index" : {
        "creation_date" : "1588410780774",
        "number_of_shards" : "1",
        "number_of_replicas" : "1",
        "uuid" : "ua0lXhtkQCOmn7Kh3iUu0w",
        "version" : {
          "created" : "7060299"
        },
        "provided_name" : "my_index"
      }
    }
  }
}
```





###### 添加新的字段映射`PUT /my_index/_mapping`

```json
PUT /my_index/_mapping
{
  "properties": {
    "employee-id": {
      "type": "keyword",
      "index": false # 字段不能被检索。检索
    }
  }
}
```

这里的 "index": false，表明新增的字段不能被检索，只是一个冗余字段。

###### 不能更新映射

对于已经存在的字段映射，我们不能更新。更新必须创建新的索引，进行数据迁移。

###### 数据迁移

先创建new_twitter的正确映射。

然后使用如下方式进行数据迁移。

```json
6.0以后写法
POST reindex
{
  "source":{
      "index":"twitter"
   },
  "dest":{
      "index":"new_twitters"
   }
}


老版本写法
POST reindex
{
  "source":{
      "index":"twitter",
      "twitter":"twitter"
   },
  "dest":{
      "index":"new_twitters"
   }
}
```

更多详情见： https://www.elastic.co/guide/en/elasticsearch/reference/7.6/docs-reindex.html 



案例：原来类型为account，新版本没有类型了，所以我们把他去掉

GET /bank/_search

```json
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1000,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "account",//原来类型为account，新版本没有类型了，所以我们把他去掉
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "account_number" : 1,
          "balance" : 39225,
          "firstname" : "Amber",
          "lastname" : "Duke",
          "age" : 32,
          "gender" : "M",
          "address" : "880 Holmes Lane",
          "employer" : "Pyrami",
          "email" : "amberduke@pyrami.com",
          "city" : "Brogan",
          "state" : "IL"
        }
      },
      ...
```



```
GET /bank/_search
查出
"age":{"type":"long"}
```

想要将年龄修改为integer

先创建新的索引

```json
PUT /newbank
{
  "mappings": {
    "properties": {
      "account_number": {
        "type": "long"
      },
      "address": {
        "type": "text"
      },
      "age": {
        "type": "integer"
      },
      "balance": {
        "type": "long"
      },
      "city": {
        "type": "keyword"
      },
      "email": {
        "type": "keyword"
      },
      "employer": {
        "type": "keyword"
      },
      "firstname": {
        "type": "text"
      },
      "gender": {
        "type": "keyword"
      },
      "lastname": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      },
      "state": {
        "type": "keyword"
      }
    }
  }
}
```

查看“newbank”的映射：

GET /newbank/_mapping

```json
能够看到age的映射类型被修改为了integer.
"age":{"type":"integer"}
```

将bank中的数据迁移到newbank中

```json
POST _reindex
{
  "source": {
    "index": "bank",
    "type": "account"
  },
  "dest": {
    "index": "newbank"
  }
}
```

运行输出：

```json
#! Deprecation: [types removal] Specifying types in reindex requests is deprecated.
{
  "took" : 768,
  "timed_out" : false,
  "total" : 1000,
  "updated" : 0,
  "created" : 1000,
  "deleted" : 0,
  "batches" : 1,
  "version_conflicts" : 0,
  "noops" : 0,
  "retries" : {
    "bulk" : 0,
    "search" : 0
  },
  "throttled_millis" : 0,
  "requests_per_second" : -1.0,
  "throttled_until_millis" : 0,
  "failures" : [ ]
}
```



查看newbank中的数据

```json
GET /newbank/_search

输出
  "hits" : {
    "total" : {
      "value" : 1000,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "newbank",
        "_type" : "_doc", # 没有了类型
```

## 四、分词

一个tokenizer（分词器）接收一个字符流，将之分割为独立的`tokens`（**词元**，通常是独立的单词），然后输出tokens流。

例如：whitespace tokenizer遇到空白字符时分割文本。它会将文本`"Quick brown fox!"`分割为`[Quick,brown,fox!]`

该tokenizer（分词器）还负责记录各个terms(词条)的顺序或position位置（用于phrase短语和word proximity词近邻查询），以及term（词条）所代表的原始word（单词）的start（起始）和end（结束）的character offsets（字符串偏移量）（用于高亮显示搜索的内容）。

elasticsearch提供了很多**内置的分词器**（标准分词器），可以用来构建custom analyzers（自定义分词器）。

关于分词器： https://www.elastic.co/guide/en/elasticsearch/reference/7.6/analysis.html 



```json
POST _analyze
{
  "analyzer": "standard",
  "text": "The 2 Brown-Foxes bone."
}
```

执行结果：

```json
{
  "tokens" : [
    {
      "token" : "the",
      "start_offset" : 0,
      "end_offset" : 3,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "2",
      "start_offset" : 4,
      "end_offset" : 5,
      "type" : "<NUM>",
      "position" : 1
    },
    {
      "token" : "brown",
      "start_offset" : 6,
      "end_offset" : 11,
      "type" : "<ALPHANUM>",
      "position" : 2
    },
    {
      "token" : "foxes",
      "start_offset" : 12,
      "end_offset" : 17,
      "type" : "<ALPHANUM>",
      "position" : 3
    },
    {
      "token" : "bone",
      "start_offset" : 18,
      "end_offset" : 22,
      "type" : "<ALPHANUM>",
      "position" : 4
    }
  ]
}
```

对于中文，我们需要安装额外的分词器

#### 1 安装`ik分词器`

所有的语言分词，默认使用的都是“Standard Analyzer”，但是这些分词器针对于中文的分词，并不友好。为此需要安装中文的分词器。

注意：不能用默认elasticsearch-plugin install xxx.zip 进行自动安装
https://github.com/medcl/elasticsearch-analysis-ik/releases



在前面安装的elasticsearch时，我们已经将elasticsearch容器的“`/usr/share/elasticsearch/plugins`”目录，映射到宿主机的“ `/mydata/elasticsearch/plugins`”目录下，所以比较方便的做法就是下载“`/elasticsearch-analysis-ik-7.4.2.zip`”文件，然后**解压到该文件夹**下即可。安装完毕后，需要重启elasticsearch容器。

 

如果不嫌麻烦，还可以采用如下的方式。

###### 1）查看elasticsearch版本号：

```shell
[vagrant@localhost ~]$ curl http://localhost:9200
{
  "name" : "66718a266132",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "xhDnsLynQ3WyRdYmQk5xhQ",
  "version" : {
    "number" : "7.4.2",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "2f90bbf7b93631e52bafb59b3b049cb44ec25e96",
    "build_date" : "2019-10-28T20:40:44.881551Z",
    "build_snapshot" : false,
    "lucene_version" : "8.2.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```



###### 2）进入es容器内部plugin目录

* docker exec -it 容器id /bin/bash

```shell
[vagrant@localhost ~]$ sudo docker exec -it elasticsearch /bin/bash

[root@66718a266132 elasticsearch]# pwd
/usr/share/elasticsearch
[root@66718a266132 elasticsearch]# pwd
/usr/share/elasticsearch
[root@66718a266132 elasticsearch]# yum install wget
#下载ik7.4.2
[root@66718a266132 elasticsearch]# wget https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.4.2/elasticsearch-analysis-ik-7.4.2.zip
```

* unzip 下载的文件

```shell
[root@66718a266132 elasticsearch]# unzip elasticsearch-analysis-ik-7.4.2.zip -d ik

#移动到plugins目录下
[root@66718a266132 elasticsearch]# mv ik plugins/
chmod -R 777 plugins/ik

docker restart elasticsearch
```

* rm -rf *.zip

```
[root@66718a266132 elasticsearch]# rm -rf elasticsearch-analysis-ik-7.6.2.zip 
```

> 怎么ssh vagrant可以看第一篇笔记

确认是否安装好了分词器

#### 2 测试分词器

使用默认分词器

```json
GET _analyze
{
   "text":"我是中国人"
}
```

请观察执行结果：

```json
{
  "tokens" : [
    {
      "token" : "我",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "<IDEOGRAPHIC>",
      "position" : 0
    },
    {
      "token" : "是",
      "start_offset" : 1,
      "end_offset" : 2,
      "type" : "<IDEOGRAPHIC>",
      "position" : 1
    },
    {
      "token" : "中",
      "start_offset" : 2,
      "end_offset" : 3,
      "type" : "<IDEOGRAPHIC>",
      "position" : 2
    },
    {
      "token" : "国",
      "start_offset" : 3,
      "end_offset" : 4,
      "type" : "<IDEOGRAPHIC>",
      "position" : 3
    },
    {
      "token" : "人",
      "start_offset" : 4,
      "end_offset" : 5,
      "type" : "<IDEOGRAPHIC>",
      "position" : 4
    }
  ]
}

```



```json
GET _analyze
{
   "analyzer": "ik_smart", 
   "text":"我是中国人"
}
```

输出结果：

```json
{
  "tokens" : [
    {
      "token" : "我",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "CN_CHAR",
      "position" : 0
    },
    {
      "token" : "是",
      "start_offset" : 1,
      "end_offset" : 2,
      "type" : "CN_CHAR",
      "position" : 1
    },
    {
      "token" : "中国人",
      "start_offset" : 2,
      "end_offset" : 5,
      "type" : "CN_WORD",
      "position" : 2
    }
  ]
}

```



```json
GET _analyze
{
   "analyzer": "ik_max_word", 
   "text":"我是中国人"
}
```

输出结果：

```json
{
  "tokens" : [
    {
      "token" : "我",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "CN_CHAR",
      "position" : 0
    },
    {
      "token" : "是",
      "start_offset" : 1,
      "end_offset" : 2,
      "type" : "CN_CHAR",
      "position" : 1
    },
    {
      "token" : "中国人",
      "start_offset" : 2,
      "end_offset" : 5,
      "type" : "CN_WORD",
      "position" : 2
    },
    {
      "token" : "中国",
      "start_offset" : 2,
      "end_offset" : 4,
      "type" : "CN_WORD",
      "position" : 3
    },
    {
      "token" : "国人",
      "start_offset" : 3,
      "end_offset" : 5,
      "type" : "CN_WORD",
      "position" : 4
    }
  ]
}
```

> 调整vagrant内存为4G

#### 3 自定义词库

> 比如我们要把尚硅谷算作一个词

* 修改/usr/share/elasticsearch/plugins/ik/config中的IKAnalyzer.cfg.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
	<comment>IK Analyzer 扩展配置</comment>
	<!--用户可以在这里配置自己的扩展字典 -->
	<entry key="ext_dict"></entry>
	 <!--用户可以在这里配置自己的扩展停止词字典-->
	<entry key="ext_stopwords"></entry>
	<!--用户可以在这里配置远程扩展字典 -->
	<entry key="remote_ext_dict">http://192.168.56.10/es/fenci.txt</entry> 
	<!--用户可以在这里配置远程扩展停止词字典-->
	<!-- <entry key="remote_ext_stopwords">words_location</entry> -->
</properties>
```



修改完成后，需要重启elasticsearch容器，否则修改不生效。docker restart elasticsearch

更新完成后，es只会对于新增的数据用更新分词。历史数据是不会重新分词的。如果想要历史数据重新分词，需要执行：

```shell
POST my_index/_update_by_query?conflicts=proceed
```

> 安装笔记1里的安装nginx安装好nginx
>
> ```bash
> mkdir /mydata/nginx/html/es
> cd /mydata/nginx/html/es
> vim fenci.txt
> 输入尚硅谷
> ```
>
> 测试http://192.168.56.10/es/fenci.txt



，然后创建“fenci.txt”文件，内容如下：

```shell
echo "樱桃萨其马，带你甜蜜入夏" > /mydata/nginx/html/fenci.txt 
```

测试效果：

```json
GET _analyze
{
   "analyzer": "ik_max_word", 
   "text":"樱桃萨其马，带你甜蜜入夏"
}
```

输出结果：

```json
{
  "tokens" : [
    {
      "token" : "樱桃",
      "start_offset" : 0,
      "end_offset" : 2,
      "type" : "CN_WORD",
      "position" : 0
    },
    {
      "token" : "萨其马",
      "start_offset" : 2,
      "end_offset" : 5,
      "type" : "CN_WORD",
      "position" : 1
    },
    {
      "token" : "带你",
      "start_offset" : 6,
      "end_offset" : 8,
      "type" : "CN_WORD",
      "position" : 2
    },
    {
      "token" : "甜蜜",
      "start_offset" : 8,
      "end_offset" : 10,
      "type" : "CN_WORD",
      "position" : 3
    },
    {
      "token" : "入夏",
      "start_offset" : 10,
      "end_offset" : 12,
      "type" : "CN_WORD",
      "position" : 4
    }
  ]
}

```





## 五、elasticsearch-Rest-Client

java操作es有两种方式

#### 1）9300: TCP

 * spring-data-elasticsearch:transport-api.jar;
   * springboot版本不同，ransport-api.jar不同，不能适配es版本
   * 7.x已经不建议使用，8以后就要废弃

#### 2）9200: HTTP

有诸多包

 * jestClient: 非官方，更新慢；
 * RestTemplate：模拟HTTP请求，ES很多操作需要自己封装，麻烦；
 * HttpClient：同上；
* `Elasticsearch-Rest-Client`：官方RestClient，封装了ES操作，API层次分明，上手简单；

最终选择Elasticsearch-Rest-Client（elasticsearch-rest-high-level-client）

https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high.html 





#### 5、附录：安装Nginx

* 随便启动一个nginx实例，只是为了复制出配置

  ```shell
  docker run -p80:80 --name nginx -d nginx:1.10   
  ```

* 将容器内的配置文件拷贝到/mydata/nginx/conf/ 下

  ```shell
  mkdir -p /mydata/nginx/html
  mkdir -p /mydata/nginx/logs
  mkdir -p /mydata/nginx/conf
  docker container cp nginx:/etc/nginx/*  /mydata/nginx/conf/ 
  #由于拷贝完成后会在config中存在一个nginx文件夹，所以需要将它的内容移动到conf中
  mv /mydata/nginx/conf/nginx/* /mydata/nginx/conf/
  rm -rf /mydata/nginx/conf/nginx
  ```

* 终止原容器：

  ```shell
  docker stop nginx
  ```

* 执行命令删除原容器：

  ```shell
  docker rm nginx
  ```

* 创建新的Nginx，执行以下命令

  ```shell
  docker run -p 80:80 --name nginx \
   -v /mydata/nginx/html:/usr/share/nginx/html \
   -v /mydata/nginx/logs:/var/log/nginx \
   -v /mydata/nginx/conf/:/etc/nginx \
   -d nginx:1.10
  ```

* 设置开机启动nginx

  ```
  docker update nginx --restart=always
  ```

  

* 创建“/mydata/nginx/html/index.html”文件，测试是否能够正常访问

  ```
  echo '<h2>hello nginx!</h2>' >index.html
  ```

  访问：http://nginx所在主机的IP:80/index.html



## 六、SpringBoot整合ElasticSearch

创建项目gulimall-search

选择依赖web，但不要在里面选择es

### 1、导入依赖

这里的版本要和所按照的ELK版本匹配。

```xml
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-high-level-client</artifactId>
    <version>7.4.2</version>
</dependency>
```



在spring-boot-dependencies中所依赖的ES版本位6.8.5，要改掉

```xml
<properties>
    <java.version>1.8</java.version>
    <elasticsearch.version>7.4.2</elasticsearch.version>
</properties>
```

请求测试项，比如es添加了安全访问规则，访问es需要添加一个安全头，就可以通过requestOptions设置

官方建议把requestOptions创建成单实例

```java
@Configuration
public class GuliESConfig {

    public static final RequestOptions COMMON_OPTIONS;

    static {
        RequestOptions.Builder builder = RequestOptions.DEFAULT.toBuilder();

        COMMON_OPTIONS = builder.build();
    }

    @Bean
    public RestHighLevelClient esRestClient() {

        RestClientBuilder builder = null;
        // 可以指定多个es
        builder = RestClient.builder(new HttpHost(host, 9200, "http"));

        RestHighLevelClient client = new RestHighLevelClient(builder);
        return client;
    }
}

```

此外还有多种方法

### 2、测试

#### 1）保存数据

 https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-document-index.html 

保存方式分为同步和异步，异步方式多了个listener回调

```java
@Test
public void indexData() throws IOException {
    
    // 设置索引
    IndexRequest indexRequest = new IndexRequest ("users");
    indexRequest.id("1");

    User user = new User();
    user.setUserName("张三");
    user.setAge(20);
    user.setGender("男");
    String jsonString = JSON.toJSONString(user);
    
    //设置要保存的内容，指定数据和类型
    indexRequest.source(jsonString, XContentType.JSON);
    
    //执行创建索引和保存数据
    IndexResponse index = client.index(indexRequest, GulimallElasticSearchConfig.COMMON_OPTIONS);

    System.out.println(index);

}
```



#### 2）获取数据

 https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-search.html 

```java
@Test
    public void find() throws IOException {
        // 1 创建检索请求
        SearchRequest searchRequest = new SearchRequest();
        searchRequest.indices("bank");
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
        // 构造检索条件
//        sourceBuilder.query();
//        sourceBuilder.from();
//        sourceBuilder.size();
//        sourceBuilder.aggregation();
        sourceBuilder.query(QueryBuilders.matchQuery("address","mill"));
        System.out.println(sourceBuilder.toString());

        searchRequest.source(sourceBuilder);

        // 2 执行检索
        SearchResponse response = client.search(searchRequest, GuliESConfig.COMMON_OPTIONS);
        // 3 分析响应结果
        System.out.println(response.toString());
    }
```

```json
{
    "took":198,
    "timed_out":false,
    "_shards": {"total":1,"successful":1,"skipped":0,"failed":0},
    "hits":{
        "total":{"value":4,"relation":"eq"},
        "max_score":5.4032025,
        "hits":[
            {"_index":"bank",
             "_type":"account",
             "_id":"970",
             "_score":5.4032025,
             "_source":{"account_number":970,"balance":19648,
                        "firstname":"Forbes","lastname":"Wallace","age":28,
                        "gender":"M","address":"990 Mill Road","employer":"Pheast",
                        "email":"forbeswallace@pheast.com","city":"Lopezo","state":"AK"}
            },
            {"_index":"bank","_type":"account","_id":"136",
             "_score":5.4032025,
             "_source":{"account_number":136,"balance":45801,"firstname":"Winnie",
                        "lastname":"Holland","age":38,"gender":"M","address":"198 Mill Lane",
                        "employer":"Neteria","email":"winnieholland@neteria.com","city":"Urie","state":"IL"
                       }
            },
            {"_index":"bank","_type":"account","_id":"345",
             "_score":5.4032025,
             "_source":{"account_number":345,"balance":9812,"firstname":"Parker",
                        "lastname":"Hines","age":38,"gender":"M",
                        "address":"715 Mill Avenue","employer":"Baluba","email":"parkerhines@baluba.com",
                        "city":"Blackgum","state":"KY"
                       }
            },
            {"_index":"bank",
             "_type":"account","_id":"472",
             "_score":5.4032025,
             "_source":{"account_number":472,"balance":25571,"firstname":"Lee","lastname":"Long",
                        "age":32,"gender":"F","address":"288 Mill Street","employer":"Comverges",
                        "email":"leelong@comverges.com","city":"Movico","state":"MT"
                       }
            }
        ]
    }
}

```



```java
 @Test
    public void find() throws IOException {
        // 1 创建检索请求
        SearchRequest searchRequest = new SearchRequest();
        searchRequest.indices("bank");
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
        // 构造检索条件
//        sourceBuilder.query();
//        sourceBuilder.from();
//        sourceBuilder.size();
//        sourceBuilder.aggregation();
        sourceBuilder.query(QueryBuilders.matchQuery("address","mill"));
        //AggregationBuilders工具类构建AggregationBuilder
        // 构建第一个聚合条件:按照年龄的值分布
        TermsAggregationBuilder agg1 = AggregationBuilders.terms("agg1").field("age").size(10);// 聚合名称
// 参数为AggregationBuilder
        sourceBuilder.aggregation(agg1);
        // 构建第二个聚合条件:平均薪资
        AvgAggregationBuilder agg2 = AggregationBuilders.avg("agg2").field("balance");
        sourceBuilder.aggregation(agg2);

        System.out.println("检索条件"+sourceBuilder.toString());

        searchRequest.source(sourceBuilder);

        // 2 执行检索
        SearchResponse response = client.search(searchRequest, GuliESConfig.COMMON_OPTIONS);
        // 3 分析响应结果
        System.out.println(response.toString());
    }
```

#### 转换bean

```java
// 3.1 获取java bean
SearchHits hits = response.getHits();
SearchHit[] hits1 = hits.getHits();
for (SearchHit hit : hits1) {
    hit.getId();
    hit.getIndex();
    String sourceAsString = hit.getSourceAsString();
    Account account = JSON.parseObject(sourceAsString, Account.class);
    System.out.println(account);

}
```

```java

Account(accountNumber=970, balance=19648, firstname=Forbes, lastname=Wallace, age=28, gender=M, address=990 Mill Road, employer=Pheast, email=forbeswallace@pheast.com, city=Lopezo, state=AK)
Account(accountNumber=136, balance=45801, firstname=Winnie, lastname=Holland, age=38, gender=M, address=198 Mill Lane, employer=Neteria, email=winnieholland@neteria.com, city=Urie, state=IL)
Account(accountNumber=345, balance=9812, firstname=Parker, lastname=Hines, age=38, gender=M, address=715 Mill Avenue, employer=Baluba, email=parkerhines@baluba.com, city=Blackgum, state=KY)
Account(accountNumber=472, balance=25571, firstname=Lee, lastname=Long, age=32, gender=F, address=288 Mill Street, employer=Comverges, email=leelong@comverges.com, city=Movico, state=MT)
```

####  Buckets分析信息

```java

// 3.2 获取检索到的分析信息
Aggregations aggregations = response.getAggregations();
Terms agg21 = aggregations.get("agg2");
for (Terms.Bucket bucket : agg21.getBuckets()) {
    String keyAsString = bucket.getKeyAsString();
    System.out.println(keyAsString);
}
```



**搜索address中包含mill的所有人的年龄分布以及平均年龄，平均薪资**


```json
GET bank/_search
{
  "query": {
    "match": {
      "address": "Mill"
    }
  },
  "aggs": {
    "ageAgg": {
      "terms": {
        "field": "age",
        "size": 10
      }
    },
    "ageAvg": {
      "avg": {
        "field": "age"
      }
    },
    "balanceAvg": {
      "avg": {
        "field": "balance"
      }
    }
  }
}
```

## 七、product-es准备

P128

ES在内存中，所以在检索中优于mysql。ES也支持集群，数据分片存储。

需求：

- 上架的商品才可以在网站展示。
- 上架的商品需要可以被检索。

#### 分析sku在es中如何存储

商品mapping

分析：商品上架在es中是存sku还是spu？

- 1）、检索的时候输入名字，是需要按照sku的title进行全文检索的
- 2）、检素使用商品规格，规格是spu的公共属性，每个spu是一样的
- 3）、按照分类id进去的都是直接列出spu的，还可以切换。
- 4〕、我们如果将sku的全量信息保存到es中（包括spu属性〕就太多字段了

方案1：

```json
{
    skuId:1
    spuId:11
    skyTitile:华为xx
    price:999
    saleCount:99
    attr:[
        {尺寸:5},
        {CPU:高通945},
        {分辨率:全高清}
	]
缺点：如果每个sku都存储规格参数(如尺寸)，会有冗余存储，因为每个spu对应的sku的规格参数都一样
```

方案2：

```json
sku索引
{
    spuId:1
    skuId:11
}
attr索引
{
    skuId:11
    attr:[
        {尺寸:5},
        {CPU:高通945},
        {分辨率:全高清}
	]
}
先找到4000个符合要求的spu，再根据4000个spu查询对应的属性，封装了4000个id，long 8B*4000=32000B=32KB
1K个人检索，就是32MB


结论：如果将规格参数单独建立索引，会出现检索时出现大量数据传输的问题，会引起网络网络
```

因此选用方案1，以空间换时间

#### ==建立product索引==

最终选用的数据模型：

- { "type": "keyword" },  # 保持数据精度问题，可以检索，但不分词
- "analyzer": "ik_smart"  # 中文分词器
- "index": false,  # 不可被检索，不生成index
- "doc_values": false # 默认为true，不可被聚合，es就不会维护一些聚合的信息

```json
PUT product
{
    "mappings":{
        "properties": {
            "skuId":{ "type": "long" },
            "spuId":{ "type": "keyword" },  # 不可分词
            "skuTitle": {
                "type": "text",
                "analyzer": "ik_smart"  # 中文分词器
            },
            "skuPrice": { "type": "keyword" },  # 保证精度问题
            "skuImg"  : { "type": "keyword" },  # 视频中有false
            "saleCount":{ "type":"long" },
            "hasStock": { "type": "boolean" },
            "hotScore": { "type": "long"  },
            "brandId":  { "type": "long" },
            "catalogId": { "type": "long"  },
            "brandName": {"type": "keyword"}, # 视频中有false
            "brandImg":{
                "type": "keyword",
                "index": false,  # 不可被检索，不生成index，只用做页面使用
                "doc_values": false # 不可被聚合，默认为true
            },
            "catalogName": {"type": "keyword" }, # 视频里有false
            "attrs": {
                "type": "nested",
                "properties": {
                    "attrId": {"type": "long"  },
                    "attrName": {
                        "type": "keyword",
                        "index": false,
                        "doc_values": false
                    },
                    "attrValue": {"type": "keyword" }
                }
            }
        }
    }
}
```

> 如果检索不到商品，自己用postman测试一下，可能有的字段需要更改，你也可以把没必要的"keyword"去掉

冗余存储的字段：不用来检索，也不用来分析，节省空间

> 库存是bool。
>
> 检索品牌id，但是不检索品牌名字、图片
>
> 用skuTitle检索

#### nested嵌入式对象

属性是"type": "nested",因为是内部的属性进行检索

数组类型的对象会被扁平化处理（对象的每个属性会分别存储到一起）

```json
user.name=["aaa","bbb"]
user.addr=["ccc","ddd"]

这种存储方式，可能会发生如下错误：
错误检索到{aaa,ddd}，这个组合是不存在的
```

数组的扁平化处理会使检索能检索到本身不存在的，为了解决这个问题，就采用了嵌入式属性，数组里是对象时用嵌入式属性（不是对象无需用嵌入式属性）

nested阅读：https://blog.csdn.net/weixin_40341116/article/details/80778599

使用聚合：https://blog.csdn.net/kabike/article/details/101460578

## 八、商品上架

按skuId上架

POST  /product/spuinfo/{spuId}/up

```java
@GetMapping("/skuId/{id}")
public R getSkuInfoBySkuId(@PathVariable("id") Long skuId){

    SpuInfoEntity entity = spuInfoService.getSpuInfoBySkuId(skuId);
    return R.ok().setData(entity);
}
```

>  product里组装好，search里上架

#### 上架实体类

商品上架需要在es中保存spu信息并更新spu的状态信息，由于`SpuInfoEntity`与索引的数据模型并不对应，所以我们要建立专门的vo进行数据传输

```java
@Data
public class SkuEsModel { //common中
    private Long skuId;
    private Long spuId;
    private String skuTitle;
    private BigDecimal skuPrice;
    private String skuImg;
    private Long saleCount;
    private boolean hasStock;
    private Long hotScore;
    private Long brandId;
    private Long catalogId;
    private String brandName;
    private String brandImg;
    private String catalogName;
    private List<Attr> attrs;

    @Data
    public static class Attr{
        private Long attrId;
        private String attrName;
        private String attrValue;
    }
}
```

#### 库存量查询

上架要确保还有库存

1)在ware微服务里添加"查询sku是否有库存"的controller

```java
// sku的规格参数相同，因此我们要将查询规格参数提前，只查询一次
/**
     * 查询sku是否有库存
     * 返回skuId 和 stock库存量
     */
@PostMapping("/hasStock")
public R getSkuHasStock(@RequestBody List<Long> SkuIds){
    List<SkuHasStockVo> vos = wareSkuService.getSkuHasStock(SkuIds);
    return R.ok().setData(vos);
}

```

然后用feign调用

2)设置R的时候最后设置成泛型的

3)收集成map的时候，`toMap()`参数为两个方法，如`SkyHasStockVo::getSkyId,  item->item.getHasStock()`

4) 将封装好的SkuInfoEntity，调用search的feign，保存到es中

下面代码为更具sku的各种信息保存到es中

```java
/**
	 * 上架商品
	 */
@PostMapping("/product") // ElasticSaveController
public R productStatusUp(@RequestBody List<SkuEsModel> skuEsModels){

    boolean status;
    try {
        status = productSaveService.productStatusUp(skuEsModels);
    } catch (IOException e) {
        log.error("ElasticSaveController商品上架错误: {}", e);
        return R.error(BizCodeEnum.PRODUCT_UP_EXCEPTION.getCode(), BizCodeEnum.PRODUCT_UP_EXCEPTION.getMsg());
    }
    if(!status){
        return R.ok();
    }
    return R.error(BizCodeEnum.PRODUCT_UP_EXCEPTION.getCode(), BizCodeEnum.PRODUCT_UP_EXCEPTION.getMsg());
}


public boolean productStatusUp(List<SkuEsModel> skuEsModels) throws IOException {
    // 1.给ES建立一个索引 product
    BulkRequest bulkRequest = new BulkRequest();
    // 2.构造保存请求
    for (SkuEsModel esModel : skuEsModels) {
        // 设置索引
        IndexRequest indexRequest = new IndexRequest(EsConstant.PRODUCT_INDEX);
        // 设置索引id
        indexRequest.id(esModel.getSkuId().toString());
        String jsonString = JSON.toJSONString(esModel);
        indexRequest.source(jsonString, XContentType.JSON);
        // add
        bulkRequest.add(indexRequest);
    }
    // bulk批量保存
    BulkResponse bulk = client.bulk(bulkRequest, GuliESConfig.COMMON_OPTIONS);
    // TODO 是否拥有错误
    boolean hasFailures = bulk.hasFailures();
    if(hasFailures){
        List<String> collect = Arrays.stream(bulk.getItems()).map(item -> item.getId()).collect(Collectors.toList());
        log.error("商品上架错误：{}",collect);
    }
    return hasFailures;
}
```

5）上架失败返回R.error(错误码,消息)

此时再定义一个错误码枚举。

在接收端获取他返回的状态码

6）上架后再让数据库中变为上架状态

7）mybatis为了能兼容接收null类型，要把long改为Long

debug时很容易远程调用异常，因为超时了



#### 根据spuId封装上架数据

前面我们写了把sku信息放到es中，但是这些信息需要我们封装，前端只是传过来了一个spuId

```java
// SpuInfoServiceImpl 
public void upSpuForSearch(Long spuId) {
        //1、查出当前spuId对应的所有sku信息,品牌的名字
        List<SkuInfoEntity> skuInfoEntities=skuInfoService.getSkusBySpuId(spuId);
        //TODO 4、根据spu查出当前sku的所有可以被用来检索的规格属性
        List<ProductAttrValueEntity> productAttrValueEntities = productAttrValueService.list(new QueryWrapper<ProductAttrValueEntity>().eq("spu_id", spuId));
        List<Long> attrIds = productAttrValueEntities.stream().map(attr -> {
            return attr.getAttrId();
        }).collect(Collectors.toList());
        List<Long> searchIds=attrService.selectSearchAttrIds(attrIds);
        Set<Long> ids = new HashSet<>(searchIds);
        List<SkuEsModel.Attr> searchAttrs = productAttrValueEntities.stream().filter(entity -> {
            return ids.contains(entity.getAttrId());
        }).map(entity -> {
            SkuEsModel.Attr attr = new SkuEsModel.Attr();
            BeanUtils.copyProperties(entity, attr);
            return attr;
        }).collect(Collectors.toList());


        //TODO 1、发送远程调用，库存系统查询是否有库存
        Map<Long, Boolean> stockMap = null;
        try {
            List<Long> longList = skuInfoEntities.stream().map(SkuInfoEntity::getSkuId).collect(Collectors.toList());
            List<SkuHasStockVo> skuHasStocks = wareFeignService.getSkuHasStocks(longList);
            stockMap = skuHasStocks.stream().collect(Collectors.toMap(SkuHasStockVo::getSkuId, SkuHasStockVo::getHasStock));
        }catch (Exception e){
            log.error("远程调用库存服务失败,原因{}",e);
        }

        //2、封装每个sku的信息
        Map<Long, Boolean> finalStockMap = stockMap;
        List<SkuEsModel> skuEsModels = skuInfoEntities.stream().map(sku -> {
            SkuEsModel skuEsModel = new SkuEsModel();
            BeanUtils.copyProperties(sku, skuEsModel);
            skuEsModel.setSkuPrice(sku.getPrice());
            skuEsModel.setSkuImg(sku.getSkuDefaultImg());
            //TODO 2、热度评分。0
            skuEsModel.setHotScore(0L);
            //TODO 3、查询品牌和分类的名字信息
            BrandEntity brandEntity = brandService.getById(sku.getBrandId());
            skuEsModel.setBrandName(brandEntity.getName());
            skuEsModel.setBrandImg(brandEntity.getLogo());
            CategoryEntity categoryEntity = categoryService.getById(sku.getCatalogId());
            skuEsModel.setCatalogName(categoryEntity.getName());
            //设置可搜索属性
            skuEsModel.setAttrs(searchAttrs);
            //设置是否有库存
            skuEsModel.setHasStock(finalStockMap==null?false:finalStockMap.get(sku.getSkuId()));
            return skuEsModel;
        }).collect(Collectors.toList());

    
        //TODO 5、将数据发给es进行保存：gulimall-search
        R r = searchFeignService.saveProductAsIndices(skuEsModels);
        if (r.getCode()==0){
            this.baseMapper.upSpuStatus(spuId, ProductConstant.ProductStatusEnum.SPU_UP.getCode());
        }else {
            log.error("商品远程es保存失败");
        }
    }
```

### gulimall-search



依赖：thymeleaf

修改源文档index.html中的路径，加上/static前缀，交由nginx响应

修改hosts  search.gulimall.com

修改nginx的配置文件 *.gulimall.com;  要注意这种配置方式不包含gulimall.com

```
 server_name gulimall.com  *.gulimall.com;
```

修改index.html成list.html。添加对应controller



#### 上架controller

在product封装好了数据，远程调用search服务，接收的controller：

```java
/*** 上架商品*/
@PostMapping("/product") // ElasticSaveController
public R productStatusUp(@RequestBody List<SkuEsModel> skuEsModels){

    boolean status;
    try {
        status = productSaveService.productStatusUp(skuEsModels);
    } catch (IOException e) {
        log.error("ElasticSaveController商品上架错误: {}", e);
        return R.error(BizCodeEnum.PRODUCT_UP_EXCEPTION.getCode(), BizCodeEnum.PRODUCT_UP_EXCEPTION.getMsg());
    }
    if(!status){
        return R.ok();
    }
    return R.error(BizCodeEnum.PRODUCT_UP_EXCEPTION.getCode(), BizCodeEnum.PRODUCT_UP_EXCEPTION.getMsg());
}
```

```java
public class ProductSaveServiceImpl implements ProductSaveService {

	@Resource
	private RestHighLevelClient client;

	/**
	 * 将数据保存到ES
	 * 用bulk代替index，进行批量保存
	 * BulkRequest bulkRequest, RequestOptions options
	 */
	@Override // ProductSaveServiceImpl
	public boolean productStatusUp(List<SkuEsModel> skuEsModels) throws IOException {
		// 1.给ES建立一个索引 product
		BulkRequest bulkRequest = new BulkRequest();
		// 2.构造保存请求
		for (SkuEsModel esModel : skuEsModels) {
			// 设置es索引
			IndexRequest indexRequest = new IndexRequest(EsConstant.PRODUCT_INDEX);
			// 设置索引id
			indexRequest.id(esModel.getSkuId().toString());
			// json格式
			String jsonString = JSON.toJSONString(esModel);
			indexRequest.source(jsonString, XContentType.JSON);
			// 添加到文档
			bulkRequest.add(indexRequest);
		}
		// bulk批量保存
		BulkResponse bulk = client.bulk(bulkRequest, GuliESConfig.COMMON_OPTIONS);
		// TODO 是否拥有错误
		boolean hasFailures = bulk.hasFailures();
		if(hasFailures){
			List<String> collect = Arrays.stream(bulk.getItems()).map(item -> item.getId()).collect(Collectors.toList());
			log.error("商品上架错误：{}",collect);
		}
		return hasFailures;
	}
}
```



## 九、商品检索



#### 1、检索参数VO与url

创建SearchParam用于检索VO

* 全文检索：skuTitle-》keyword
* 排序：saleCount（销量）、hotScore（热度分）、skuPrice（价格）
* 过滤：hasStock、skuPrice区间、brandId、catalog3Id、attrs
* 聚合：attrs

```java
keyword=小米&
sort=saleCount_desc/asc&
hasStock=0/1&
skuPrice=400_1900&
brandId=1&
catalog3Id=1&
attrs=1_3G:4G:5G&
attrs=2_骁龙845&
attrs=4_高清屏
```

```java
/**
封装页面所有可能传递过来的关键字
 * catalog3Id=225&keyword=华为&sort=saleCount_asc&hasStock=0/1&brandId=25&brandId=30
 */
@Data
public class SearchParam {

    // 页面传递过来的全文匹配关键字
    private String keyword;

    /** 三级分类id*/
    private Long catalog3Id;
    //排序条件：sort=price/salecount/hotscore_desc/asc
    private String sort;
    // 仅显示有货
    private Integer hasStock;

    /*** 价格区间 */
    private String skuPrice;

    /*** 品牌id 可以多选 */
    private List<Long> brandId;

    /*** 按照属性进行筛选 */
    private List<String> attrs;

    /*** 页码*/
    private Integer pageNum = 1;

    /*** 原生所有查询属性*/
    private String _queryString;
}
```

##### 检索结果VO

查询得到商品、总记录数、总页码
品牌list用于在品牌栏显示，分类list用于在分类栏显示

其他栏每栏用AttrVo表示

- 不仅要根据关键字从es中检索到商品
- 还要通过**聚合**生成**品牌**等信息，方便**分类栏**显示

```java
/**
 * <p>Title: SearchResponse</p>
 * Description：包含页面需要的所有信息
 */
@Data
public class SearchResult {

    /** * 查询到的所有商品信息*/
    private List<SkuEsModel> products;

    /*** 当前页码*/
    private Integer pageNum;
    /** 总记录数*/
    private Long total;
    /** * 总页码*/
    private Integer totalPages;

    /** 当前查询到的结果, 所有涉及到的品牌*/
    private List<BrandVo> brands;
    /*** 当前查询到的结果, 所有涉及到的分类*/
    private List<CatalogVo> catalogs;
	/** * 当前查询的结果 所有涉及到所有属性*/
    private List<AttrVo> attrs;

	/** 导航页   页码遍历结果集(分页)  */
	private List<Integer> pageNavs;
//	================以上是返回给页面的所有信息================

    /** 导航数据*/
    private List<NavVo> navs = new ArrayList<>();

    /** 便于判断当前id是否被使用*/
    private List<Long> attrIds = new ArrayList<>();

    @Data
    public static class NavVo {
        private String name;
        private String navValue;
        private String link;
    }

    @Data
    public static class BrandVo {

        private Long brandId;
        private String brandName;
        private String brandImg;
    }

    @Data
    public static class CatalogVo {
        private Long catalogId;
        private String catalogName;
    }

    @Data
    public static class AttrVo {

        private Long attrId;
        private String attrName;
        private List<String> attrValue;
    }
}

```



### 2、ES语句DSL



此处先写出如何检索指定的商品，如检索"华为"关键字

- 嵌入式的属性 
- highlight：设置该值后，返回的时候就包装过了
- 查出结果后，附属栏也要对应变化
- 嵌入式的聚合时候也要注意

> 使用时将我的注释去掉
>
> ```json
> "attrs": { # 聚合名字
>     "type": "nested",  # nested
>     "properties": {
>         "attrId": {"type": "long"  },
>         "attrName": {
>             "type": "keyword",
>             "index": false,
>             "doc_values": false
>         },
>         "attrValue": {"type": "keyword" }
>     }
> }
> }
> ```
>

比如我们要根据一些信息检索出符合条件的文档

```json
GET gulimall_product/_search
{
  "query": {
    "bool": {
      "must": [ {"match": {  "skuTitle": "华为" }} ], # 检索出华为
      "filter": [ # 过滤
        { "term": { "catalogId": "225" } },
        { "terms": {"brandId": [ "2"] } }, 
        { "term": { "hasStock": "false"} },
        {
          "range": {
            "skuPrice": { # 价格1K~7K
              "gte": 1000,
              "lte": 7000
            }
          }
        },
        {
          "nested": {
            "path": "attrs", # 聚合名字
            "query": {
              "bool": {
                "must": [
                  {
                    "term": { "attrs.attrId": { "value": "6"} }
                  }
                ]
              }
            }
          }
        }
      ]
    }
  },
  "sort": [ {"skuPrice": {"order": "desc" } } ],
  "from": 0,
  "size": 5,
  "highlight": {  
    "fields": {"skuTitle": {}}, # 高亮的字段
    "pre_tags": "<b style='color:red'>",  # 前缀
    "post_tags": "</b>"
  },
  "aggs": { # 查完后聚合
    "brandAgg": {
      "terms": {
        "field": "brandId",
        "size": 10
      },
      "aggs": { # 子聚合
        "brandNameAgg": {  # 每个商品id的品牌
          "terms": {
            "field": "brandName",
            "size": 10
          }
        },
      
        "brandImgAgg": {
          "terms": {
            "field": "brandImg",
            "size": 10
          }
        }
        
      }
    },
    "catalogAgg":{
      "terms": {
        "field": "catalogId",
        "size": 10
      },
      "aggs": {
        "catalogNameAgg": {
          "terms": {
            "field": "catalogName",
            "size": 10
          }
        }
      }
    },
    "attrs":{
      "nested": {"path": "attrs" },
      "aggs": {
        "attrIdAgg": {
          "terms": {
            "field": "attrs.attrId",
            "size": 10
          },
          "aggs": {
            "attrNameAgg": {
              "terms": {
                "field": "attrs.attrName",
                "size": 10
              }
            }
          }
        }
      }
    }
  }
}
```

### 3、检索业务层

- 请求带来的参数是SearchParam
- 传给es的参数是SearchRequest
- es返回结果是SearchResponse
- 把结果封装为SearchResult

#### 1)  controller

主要逻辑在service层进行，service层将封装好的`SearchParam`组建查询条件，再将返回后的结果封装成`SearchResult`

```java
@GetMapping(value = {"/search.html","/"})
public String getSearchPage(SearchParam searchParam, // 检索参数，
                            Model model, HttpServletRequest request) {
    searchParam.set_queryString(request.getQueryString());//_queryString是个字段
    SearchResult result=searchService.getSearchResult(searchParam);
    model.addAttribute("result", result);
    return "search";
}
```

DSL转java主要逻辑：

```JAVA
// service
public SearchResult getSearchResult(SearchParam searchParam) {//根据带来的请求内容封装
    SearchResult searchResult= null;
    // 通过请求参数构建查询请求
    SearchRequest request = bulidSearchRequest(searchParam);
    try {
        SearchResponse searchResponse = restHighLevelClient.search(request, 
                                                                   GulimallElasticSearchConfig.COMMON_OPTIONS);
        // 将es响应数据封装成结果
        searchResult = bulidSearchResult(searchParam,searchResponse);
    } catch (IOException e) {
        e.printStackTrace();
    }
    return searchResult;
}
```



##### DSL转java

```java
private SearchRequest bulidSearchRequest(SearchParam searchParam) {
    // 用于构建DSL语句 
    SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
    //1. 构建bool query
    BoolQueryBuilder boolQueryBuilder = new BoolQueryBuilder();
    //1.1 bool must
    if (!StringUtils.isEmpty(searchParam.getKeyword())) {
        boolQueryBuilder.must(QueryBuilders.matchQuery("skuTitle", searchParam.getKeyword()));
    }

    //1.2 bool filter
    //1.2.1 catalog
    if (searchParam.getCatalog3Id()!=null){
        boolQueryBuilder.filter(QueryBuilders.termQuery("catalogId", searchParam.getCatalog3Id()));
    }
    //1.2.2 brand
    if (searchParam.getBrandId()!=null&&searchParam.getBrandId().size()>0) {
        boolQueryBuilder.filter(QueryBuilders.termsQuery("brandId",searchParam.getBrandId()));
    }
    //1.2.3 hasStock
    if (searchParam.getHasStock() != null) {
        boolQueryBuilder.filter(QueryBuilders.termQuery("hasStock", searchParam.getHasStock() == 1));
    }
    //1.2.4 priceRange
    RangeQueryBuilder rangeQueryBuilder = QueryBuilders.rangeQuery("skuPrice");
    if (!StringUtils.isEmpty(searchParam.getSkuPrice())) {
        String[] prices = searchParam.getSkuPrice().split("_");
        if (prices.length == 1) {
            if (searchParam.getSkuPrice().startsWith("_")) {
                rangeQueryBuilder.lte(Integer.parseInt(prices[0]));
            }else {
                rangeQueryBuilder.gte(Integer.parseInt(prices[0]));
            }
        } else if (prices.length == 2) {
            //_6000会截取成["","6000"]
            if (!prices[0].isEmpty()) {
                rangeQueryBuilder.gte(Integer.parseInt(prices[0]));
            }
            rangeQueryBuilder.lte(Integer.parseInt(prices[1]));
        }
        boolQueryBuilder.filter(rangeQueryBuilder);
    }
    //1.2.5 attrs-nested
    //attrs=1_5寸:8寸&2_16G:8G
    List<String> attrs = searchParam.getAttrs();
    BoolQueryBuilder queryBuilder = new BoolQueryBuilder();
    if (attrs!=null&&attrs.size() > 0) {
        attrs.forEach(attr->{
            String[] attrSplit = attr.split("_");
            queryBuilder.must(QueryBuilders.termQuery("attrs.attrId", attrSplit[0]));
            String[] attrValues = attrSplit[1].split(":");
            queryBuilder.must(QueryBuilders.termsQuery("attrs.attrValue", attrValues));
        });
    }
    NestedQueryBuilder nestedQueryBuilder = QueryBuilders.nestedQuery("attrs", queryBuilder, ScoreMode.None);
    boolQueryBuilder.filter(nestedQueryBuilder);
    //1.X bool query构建完成
    searchSourceBuilder.query(boolQueryBuilder);

    //2. sort  eg:sort=saleCount_desc/asc
    if (!StringUtils.isEmpty(searchParam.getSort())) {
        String[] sortSplit = searchParam.getSort().split("_");
        searchSourceBuilder.sort(sortSplit[0], sortSplit[1].equalsIgnoreCase("asc") ? SortOrder.ASC : SortOrder.DESC);
    }

    //3. 分页 // 是检测结果分页
    searchSourceBuilder.from((searchParam.getPageNum() - 1) * EsConstant.PRODUCT_PAGESIZE);
    searchSourceBuilder.size(EsConstant.PRODUCT_PAGESIZE);

    //4. 高亮highlight
    if (!StringUtils.isEmpty(searchParam.getKeyword())) {
        HighlightBuilder highlightBuilder = new HighlightBuilder();
        highlightBuilder.field("skuTitle");
        highlightBuilder.preTags("<b style='color:red'>");
        highlightBuilder.postTags("</b>");
        searchSourceBuilder.highlighter(highlightBuilder);
    }

    //5. 聚合
    //5.1 按照brand聚合
    TermsAggregationBuilder brandAgg = AggregationBuilders.terms("brandAgg").field("brandId");
    TermsAggregationBuilder brandNameAgg = AggregationBuilders.terms("brandNameAgg").field("brandName");
    TermsAggregationBuilder brandImgAgg = AggregationBuilders.terms("brandImgAgg").field("brandImg");
    brandAgg.subAggregation(brandNameAgg);
    brandAgg.subAggregation(brandImgAgg);
    searchSourceBuilder.aggregation(brandAgg);

    //5.2 按照catalog聚合
    TermsAggregationBuilder catalogAgg = AggregationBuilders.terms("catalogAgg").field("catalogId");
    // 子聚合
    TermsAggregationBuilder catalogNameAgg = AggregationBuilders.terms("catalogNameAgg").field("catalogName");
    catalogAgg.subAggregation(catalogNameAgg);
    searchSourceBuilder.aggregation(catalogAgg);

    //5.3 按照attrs聚合
    NestedAggregationBuilder nestedAggregationBuilder = new NestedAggregationBuilder("attrs", "attrs");
    //按照attrId聚合     //按照attrId聚合之后再按照attrName和attrValue聚合
    TermsAggregationBuilder attrIdAgg    = AggregationBuilders.terms("attrIdAgg"   ).field("attrs.attrId");
    TermsAggregationBuilder attrNameAgg  = AggregationBuilders.terms("attrNameAgg" ).field("attrs.attrName");
    TermsAggregationBuilder attrValueAgg = AggregationBuilders.terms("attrValueAgg").field("attrs.attrValue");
    attrIdAgg.subAggregation(attrNameAgg);
    attrIdAgg.subAggregation(attrValueAgg);

    nestedAggregationBuilder.subAggregation(attrIdAgg);
    searchSourceBuilder.aggregation(nestedAggregationBuilder);

    log.debug("构建的DSL语句 {}",searchSourceBuilder.toString());

    SearchRequest request = new SearchRequest(new String[]{EsConstant.PRODUCT_INDEX}, searchSourceBuilder);
    return request;
}
```

#### 2) 接收检索响应

- 得到检索到的商品
- 还有聚合信息

```java
private SearchResult bulidSearchResult(SearchParam searchParam, SearchResponse searchResponse) {
    SearchResult result = new SearchResult();
    
    SearchHits hits = searchResponse.getHits();
    //1. 封装查询到的商品信息
    if (hits.getHits()!=null&&hits.getHits().length>0){
        List<SkuEsModel> skuEsModels = new ArrayList<>();
        for (SearchHit hit : hits) {
            String sourceAsString = hit.getSourceAsString();
            SkuEsModel skuEsModel = JSON.parseObject(sourceAsString, SkuEsModel.class);
            //设置高亮属性
            if (!StringUtils.isEmpty(searchParam.getKeyword())) {
                HighlightField skuTitle = hit.getHighlightFields().get("skuTitle");
                String highLight = skuTitle.getFragments()[0].string();
                skuEsModel.setSkuTitle(highLight);
            }
            skuEsModels.add(skuEsModel);
        }
        result.setProduct(skuEsModels);
    }

    //2. 封装分页信息
    //2.1 当前页码
    result.setPageNum(searchParam.getPageNum());
    //2.2 总记录数
    long total = hits.getTotalHits().value;
    result.setTotal(total);
    //2.3 总页码
    Integer totalPages = (int)total % EsConstant.PRODUCT_PAGESIZE == 0 ?
            (int)total / EsConstant.PRODUCT_PAGESIZE : (int)total / EsConstant.PRODUCT_PAGESIZE + 1;
    result.setTotalPages(totalPages);
    List<Integer> pageNavs = new ArrayList<>();
    for (int i = 1; i <= totalPages; i++) {
        pageNavs.add(i);
    }
    result.setPageNavs(pageNavs);

    //3. 查询结果涉及到的品牌
    List<SearchResult.BrandVo> brandVos = new ArrayList<>();
    Aggregations aggregations = searchResponse.getAggregations();
    //ParsedLongTerms用于接收terms聚合的结果，并且可以把key转化为Long类型的数据
    ParsedLongTerms brandAgg = aggregations.get("brandAgg");
    for (Terms.Bucket bucket : brandAgg.getBuckets()) {
        //3.1 得到品牌id
        Long brandId = bucket.getKeyAsNumber().longValue();

        Aggregations subBrandAggs = bucket.getAggregations();
        //3.2 得到品牌图片
        ParsedStringTerms brandImgAgg=subBrandAggs.get("brandImgAgg");
        String brandImg = brandImgAgg.getBuckets().get(0).getKeyAsString();
        //3.3 得到品牌名字
        Terms brandNameAgg=subBrandAggs.get("brandNameAgg");
        String brandName = brandNameAgg.getBuckets().get(0).getKeyAsString();
        SearchResult.BrandVo brandVo = new SearchResult.BrandVo(brandId, brandName, brandImg);
        brandVos.add(brandVo);
    }
    result.setBrands(brandVos);

    //4. 查询涉及到的所有分类
    List<SearchResult.CatalogVo> catalogVos = new ArrayList<>();
    ParsedLongTerms catalogAgg = aggregations.get("catalogAgg");
    for (Terms.Bucket bucket : catalogAgg.getBuckets()) {
        //4.1 获取分类id
        Long catalogId = bucket.getKeyAsNumber().longValue();
        Aggregations subcatalogAggs = bucket.getAggregations();
        //4.2 获取分类名
        ParsedStringTerms catalogNameAgg=subcatalogAggs.get("catalogNameAgg");
        String catalogName = catalogNameAgg.getBuckets().get(0).getKeyAsString();
        SearchResult.CatalogVo catalogVo = new SearchResult.CatalogVo(catalogId, catalogName);
        catalogVos.add(catalogVo);
    }
    result.setCatalogs(catalogVos);

    //5 查询涉及到的所有属性
    List<SearchResult.AttrVo> attrVos = new ArrayList<>();
    //ParsedNested用于接收内置属性的聚合
    ParsedNested parsedNested=aggregations.get("attrs");
    ParsedLongTerms attrIdAgg=parsedNested.getAggregations().get("attrIdAgg");
    for (Terms.Bucket bucket : attrIdAgg.getBuckets()) {
        //5.1 查询属性id
        Long attrId = bucket.getKeyAsNumber().longValue();

        Aggregations subAttrAgg = bucket.getAggregations();
        //5.2 查询属性名
        ParsedStringTerms attrNameAgg=subAttrAgg.get("attrNameAgg");
        String attrName = attrNameAgg.getBuckets().get(0).getKeyAsString();
        //5.3 查询属性值
        ParsedStringTerms attrValueAgg = subAttrAgg.get("attrValueAgg");
        List<String> attrValues = new ArrayList<>();
        for (Terms.Bucket attrValueAggBucket : attrValueAgg.getBuckets()) {
            String attrValue = attrValueAggBucket.getKeyAsString();
            attrValues.add(attrValue);
            List<SearchResult.NavVo> navVos = new ArrayList<>();
        }
        SearchResult.AttrVo attrVo = new SearchResult.AttrVo(attrId, attrName, attrValues);
        attrVos.add(attrVo);
    }
    result.setAttrs(attrVos);

    // 6. 构建面包屑导航
    List<String> attrs = searchParam.getAttrs();
    if (attrs != null && attrs.size() > 0) {
        List<SearchResult.NavVo> navVos = attrs.stream().map(attr -> {
            String[] split = attr.split("_");
            SearchResult.NavVo navVo = new SearchResult.NavVo();
            //6.1 设置属性值
            navVo.setNavValue(split[1]);
            //6.2 查询并设置属性名
            try {
                R r = productFeignService.info(Long.parseLong(split[0]));
                if (r.getCode() == 0) {
                    AttrResponseVo attrResponseVo = JSON.parseObject(JSON.toJSONString(r.get("attr")), new TypeReference<AttrResponseVo>() {
                    });
                    navVo.setNavName(attrResponseVo.getAttrName());
                }
            } catch (Exception e) {
                log.error("远程调用商品服务查询属性失败", e);
            }
            //6.3 设置面包屑跳转链接
            String queryString = searchParam.get_queryString();
            String replace = queryString.replace("&attrs=" + attr, "").replace("attrs=" + attr+"&", "").replace("attrs=" + attr, "");
            navVo.setLink("http://search.gulimall.com/search.html" + (replace.isEmpty()?"":"?"+replace));
            return navVo;
        }).collect(Collectors.toList());
        result.setNavs(navVos);
    }
    return result;
}
```

P182完

## 十、渲染检索页面

#### 1)  基本数据渲染

将商品的基本属性渲染出来

```html
<div class="rig_tab">
    <!-- 遍历各个商品-->
    <div th:each="product : ${result.getProduct()}">
        <div class="ico">
            <i class="iconfont icon-weiguanzhu"></i>
            <a href="/static/search/#">关注</a>
        </div>
        <p class="da">
            <a th:href="|http://item.gulimall.com/${product.skuId}.html|" >
                <!--图片 -->
                <img   class="dim" th:src="${product.skuImg}">
            </a>
        </p>
        <ul class="tab_im">
            <li><a href="/static/search/#" title="黑色">
                <img th:src="${product.skuImg}"></a></li>
        </ul>
        <p class="tab_R">
              <!-- 价格 -->
            <span th:text="'￥' + ${product.skuPrice}">¥5199.00</span>
        </p>
        <p class="tab_JE">
            <!-- 标题 -->
            <!-- 使用utext标签,使检索时高亮不会被转义-->
            <a href="/static/search/#" th:utext="${product.skuTitle}">
                Apple iPhone 7 Plus (A1661) 32G 黑色 移动联通电信4G手机
            </a>
        </p>
        <p class="tab_PI">已有<span>11万+</span>热门评价
            <a href="/static/search/#">二手有售</a>
        </p>
        <p class="tab_CP"><a href="/static/search/#" title="谷粒商城Apple产品专营店">谷粒商城Apple产品...</a>
            <a href='#' title="联系供应商进行咨询">
                <img src="/static/search/img/xcxc.png">
            </a>
        </p>
        <div class="tab_FO">
            <div class="FO_one">
                <p>自营
                    <span>谷粒商城自营,品质保证</span>
                </p>
                <p>满赠
                    <span>该商品参加满赠活动</span>
                </p>
            </div>
        </div>
    </div>
</div>
```

#### 2) 筛选条件渲染

将结果的品牌、分类、商品属性进行遍历显示，并且点击某个属性值时 可以通过拼接url进行跳转

```html
<div class="JD_nav_logo">
    <!--品牌-->
    <div class="JD_nav_wrap">
        <div class="sl_key">
            <span>品牌：</span>
        </div>
        <div class="sl_value">
            <div class="sl_value_logo">
                <ul>
                    <li th:each="brand: ${result.getBrands()}">
                        <!--替换url-->
                        <a href="#"  th:href="${'javascript:searchProducts(&quot;brandId&quot;,'+brand.brandId+')'}">
                            <img src="/static/search/img/598033b4nd6055897.jpg" alt="" th:src="${brand.brandImg}">
                            <div th:text="${brand.brandName}">
                                华为(HUAWEI)
                            </div>
                        </a>
                    </li>
                </ul>
            </div>
        </div>
        <div class="sl_ext">
            <a href="#">
                更多
                <i style='background: url("image/search.ele.png")no-repeat 3px 7px'></i>
                <b style='background: url("image/search.ele.png")no-repeat 3px -44px'></b>
            </a>
            <a href="#">
                多选
                <i>+</i>
                <span>+</span>
            </a>
        </div>
    </div>
    <!--分类-->
    <div class="JD_pre" th:each="catalog: ${result.getCatalogs()}">
        <div class="sl_key">
            <span>分类：</span>
        </div>
        <div class="sl_value">
            <ul>
                <li><a href="#" th:text="${catalog.getCatalogName()}" th:href="${'javascript:searchProducts(&quot;catalogId&quot;,'+catalog.catalogId+')'}">0-安卓（Android）</a></li>
            </ul>
        </div>
    </div>
    <!--价格-->
    <div class="JD_pre">
        <div class="sl_key">
            <span>价格：</span>
        </div>
        <div class="sl_value">
            <ul>
                <li><a href="#">0-499</a></li>
                <li><a href="#">500-999</a></li>
                <li><a href="#">1000-1699</a></li>
                <li><a href="#">1700-2799</a></li>
                <li><a href="#">2800-4499</a></li>
                <li><a href="#">4500-11999</a></li>
                <li><a href="#">12000以上</a></li>
                <li class="sl_value_li">
                    <input type="text">
                    <p>-</p>
                    <input type="text">
                    <a href="#">确定</a>
                </li>
            </ul>
        </div>
    </div>
    <!--商品属性-->
    <div class="JD_pre" th:each="attr: ${result.getAttrs()}" >
        <div class="sl_key">
            <span th:text="${attr.getAttrName()}">系统：</span>
        </div>
        <div class="sl_value">
            <ul>
                <li th:each="val: ${attr.getAttrValue()}">
                    <a href="#"  th:text="${val}"
                       th:href="${'javascript:searchProducts(&quot;attrs&quot;,&quot;'+attr.attrId+'_'+val+'&quot;)'}">0-安卓（Android）</a></li>
            </ul>
        </div>
    </div>
</div>
```

```javascript
function searchProducts(name, value) {
    //原來的页面
    location.href = replaceParamVal(location.href,name,value,true)
};

   /**
     * @param url 目前的url
     * @param paramName 需要替换的参数属性名
     * @param replaceVal 需要替换的参数的新属性值
     * @param forceAdd 该参数是否可以重复查询(attrs=1_3G:4G:5G&attrs=2_骁龙845&attrs=4_高清屏)
     * @returns {string} 替换或添加后的url
     */
function replaceParamVal(url, paramName, replaceVal,forceAdd) {
    var oUrl = url.toString();
    var nUrl;
    if (oUrl.indexOf(paramName) != -1) {
        if( forceAdd && oUrl.indexOf(paramName+"="+replaceVal)==-1) {
            if (oUrl.indexOf("?") != -1) {
                nUrl = oUrl + "&" + paramName + "=" + replaceVal;
            } else {
                nUrl = oUrl + "?" + paramName + "=" + replaceVal;
            }
        } else {
            var re = eval('/(' + paramName + '=)([^&]*)/gi');
            nUrl = oUrl.replace(re, paramName + '=' + replaceVal);
        }
    } else {
        if (oUrl.indexOf("?") != -1) {
            nUrl = oUrl + "&" + paramName + "=" + replaceVal;
        } else {
            nUrl = oUrl + "?" + paramName + "=" + replaceVal;
        }
    }
    return nUrl;
};
```

#### 3) 分页数据渲染

将页码绑定至属性pn，当点击某页码时，通过获取pn值进行url拼接跳转页面

```html
<div class="filter_page">
    <div class="page_wrap">
        <span class="page_span1">
               <!-- 不是第一页时显示上一页 -->
            <a class="page_a" href="#" th:if="${result.pageNum>1}" th:attr="pn=${result.getPageNum()-1}">
                 上一页
            </a>
             <!-- 将各个页码遍历显示，并将当前页码绑定至属性pn -->
            <a href="#" class="page_a"
               th:each="page: ${result.pageNavs}"
               th:text="${page}"
               th:style="${page==result.pageNum?'border: 0;color:#ee2222;background: #fff':''}"
               th:attr="pn=${page}"
            >1</a>
              <!-- 不是最后一页时显示下一页 -->
            <a href="#" class="page_a" th:if="${result.pageNum<result.totalPages}" th:attr="pn=${result.getPageNum()+1}">
                下一页 >
            </a>
        </span>
        <span class="page_span2">
            <em>共<b th:text="${result.totalPages}">169</b>页&nbsp;&nbsp;到第</em>
            <input type="number" value="1" class="page_input">
            <em>页</em>
            <a href="#">确定</a>
        </span>
    </div>
</div>
```

```javascript
$(".page_a").click(function () {
    var pn=$(this).attr("pn");
    location.href=replaceParamVal(location.href,"pageNum",pn,false);
    console.log(replaceParamVal(location.href,"pageNum",pn,false))
})
```

#### 4) 页面排序和价格区间



页面排序功能需要保证，点击某个按钮时，样式会变红，并且其他的样式保持最初的样子；

点击某个排序时首先按升序显示，再次点击再变为降序，并且还会显示上升或下降箭头

页面排序跳转的思路是通过点击某个按钮时会向其`class`属性添加/去除`desc`，并根据属性值进行url拼接

```html
<div class="filter_top">
    <div class="filter_top_left" th:with="p = ${param.sort}, priceRange = ${param.skuPrice}">
        <!-- 通过判断当前class是否有desc来进行样式的渲染和箭头的显示-->
        <a sort="hotScore"
           th:class="${(!#strings.isEmpty(p) && #strings.startsWith(p,'hotScore') && #strings.endsWith(p,'desc')) ? 'sort_a desc' : 'sort_a'}"
           th:attr="style=${(#strings.isEmpty(p) || #strings.startsWith(p,'hotScore')) ?
               'color: #fff; border-color: #e4393c; background: #e4393c;':'color: #333; border-color: #ccc; background: #fff;' }">
            综合排序[[${(!#strings.isEmpty(p) && #strings.startsWith(p,'hotScore') &&
            #strings.endsWith(p,'desc')) ?'↓':'↑' }]]</a>
        <a sort="saleCount"
           th:class="${(!#strings.isEmpty(p) && #strings.startsWith(p,'saleCount') && #strings.endsWith(p,'desc')) ? 'sort_a desc' : 'sort_a'}"
           th:attr="style=${(!#strings.isEmpty(p) && #strings.startsWith(p,'saleCount')) ?
               'color: #fff; border-color: #e4393c; background: #e4393c;':'color: #333; border-color: #ccc; background: #fff;' }">
            销量[[${(!#strings.isEmpty(p) && #strings.startsWith(p,'saleCount') &&
            #strings.endsWith(p,'desc'))?'↓':'↑'  }]]</a>
        <a sort="skuPrice"
           th:class="${(!#strings.isEmpty(p) && #strings.startsWith(p,'skuPrice') && #strings.endsWith(p,'desc')) ? 'sort_a desc' : 'sort_a'}"
           th:attr="style=${(!#strings.isEmpty(p) && #strings.startsWith(p,'skuPrice')) ?
               'color: #fff; border-color: #e4393c; background: #e4393c;':'color: #333; border-color: #ccc; background: #fff;' }">
            价格[[${(!#strings.isEmpty(p) && #strings.startsWith(p,'skuPrice') &&
            #strings.endsWith(p,'desc'))?'↓':'↑'  }]]</a>
        <a sort="hotScore" class="sort_a">评论分</a>
        <a sort="hotScore" class="sort_a">上架时间</a>
        <!--价格区间搜索-->
        <input id="skuPriceFrom" type="number"
               th:value="${#strings.isEmpty(priceRange)?'':#strings.substringBefore(priceRange,'_')}"
               style="width: 100px; margin-left: 30px">
        -
        <input id="skuPriceTo" type="number"
               th:value="${#strings.isEmpty(priceRange)?'':#strings.substringAfter(priceRange,'_')}"
               style="width: 100px">
        <button id="skuPriceSearchBtn">确定</button>
    </div>
    <div class="filter_top_right">
        <span class="fp-text">
           <b>1</b><em>/</em><i>169</i>
       </span>
        <a href="#" class="prev"><</a>
        <a href="#" class="next"> > </a>
    </div>
</div>
```

```javascript
$(".sort_a").click(function () {
    	//添加、剔除desc
        $(this).toggleClass("desc");
    	//获取sort属性值并进行url跳转
        let sort = $(this).attr("sort");
        sort = $(this).hasClass("desc") ? sort + "_desc" : sort + "_asc";
        location.href = replaceParamVal(location.href, "sort", sort,false);
        return false;
    });
```

价格区间搜索函数

```javascript
$("#skuPriceSearchBtn").click(function () {
    var skuPriceFrom = $("#skuPriceFrom").val();
    var skuPriceTo = $("#skuPriceTo").val();
    location.href = replaceParamVal(location.href, "skuPrice", skuPriceFrom + "_" + skuPriceTo, false);
})
```

#### 5) 面包屑导航

在封装结果时，将查询的属性值进行封装

```java
   // 6. 构建面包屑导航
        List<String> attrs = searchParam.getAttrs();
        if (attrs != null && attrs.size() > 0) {
            List<SearchResult.NavVo> navVos = attrs.stream().map(attr -> {
                String[] split = attr.split("_");
                SearchResult.NavVo navVo = new SearchResult.NavVo();
                //6.1 设置属性值
                navVo.setNavValue(split[1]);
                //6.2 查询并设置属性名
                try {
                    R r = productFeignService.info(Long.parseLong(split[0]));
                    if (r.getCode() == 0) {
                        AttrResponseVo attrResponseVo = JSON.parseObject(JSON.toJSONString(r.get("attr")), new TypeReference<AttrResponseVo>() {
                        });
                        navVo.setNavName(attrResponseVo.getAttrName());
                    }
                } catch (Exception e) {
                    log.error("远程调用商品服务查询属性失败", e);
                }
                //6.3 设置面包屑跳转链接(当点击该链接时剔除点击属性)
                String queryString = searchParam.get_queryString();
                String replace = queryString.replace("&attrs=" + attr, "").replace("attrs=" + attr+"&", "").replace("attrs=" + attr, "");
                navVo.setLink("http://search.gulimall.com/search.html" + (replace.isEmpty()?"":"?"+replace));
                return navVo;
            }).collect(Collectors.toList());
            result.setNavs(navVos);
        }
```

页面渲染

```html
<div class="JD_ipone_one c">
    <!-- 遍历面包屑功能 -->
    <a th:href="${nav.link}" th:each="nav:${result.navs}"><span th:text="${nav.navName}"></span>：<span th:text="${nav.navValue}"></span> x</a>
</div>
```

search.gulimall.com/search.html/keyword=华为&pageNum=2&attrs=6_2019

![](https://fermhan.oss-cn-qingdao.aliyuncs.com/img/20210226163544.png)



#### 6) 条件筛选联动

就是将品牌和分类也封装进面包屑数据中，并且在页面进行th:if的判断，当url有该属性的查询条件时就不进行显示了

### 笔记不易：

笔记均为markdown格式，图片也是云图，10多篇笔记20W字，压缩包仅500k，推荐使用typora阅读。

![](https://fermhan.oss-cn-qingdao.aliyuncs.com/img/20210301013846.png)

如果帮到了你，留下赞吧，谢谢支持



笔记-高级篇：[https://blog.csdn.net/hancoder/article/details/107612746](https://blog.csdn.net/hancoder/article/details/107612746)