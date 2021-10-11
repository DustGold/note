# hive各种数据导入方式对count(*)及元数据的影响

## 结论：

|                                                              | count(*)是否正确 | count(*)是否走MR | 是否更改numFiles | 是否更改numRows | 插入数据时是否走MR |
| ------------------------------------------------------------ | ---------------- | ---------------- | ---------------- | --------------- | ------------------ |
| <font color="red">load</font> data [local] inpath '数据的 path' [overwrite] into table  表名[partition (partcol1=val1,…)]; | √                | √                | √                | ×               | ×                  |
| <font color="red">insert</font> [into/overwrite] table 表名 <font color="red">select</font> 列名 from 表名; | √                | ×                | √                | √               | √                  |
| <font color="red">create table</font> if not exists 表名 as select 列名 from 表名; | √                | ×                | √                | √               | √                  |
| <font color="red">create external table</font> if not exists 表名(字段名) row format delimited fields terminated by '\t' <font color="red">location </font>'HDFS上的路径'; | √                | √                | ×                | 没有numRows     | ×                  |
| <font color="red">import table</font> 表名 from 'HDFS上的路径'; | √                | √                | √                | ×               | ×                  |

## 实验：

### ①load

```sql
load data [local] inpath '数据的 path' [overwrite] into table  表名[partition (partcol1=val1,…)];
```

#### 1、创建表

```sql
create table if not exists test_load(
deptno int,
dname string,
loc int
)
row format delimited fields terminated by '\t';
```

![1633955431480](count().assets/1633955431480.png)

#### 2、插入数据

```sql
load data local inpath '/opt/module/data/dept' into table test_load;
```

![1633955474599](count().assets/1633955474599.png)

当然，事先准备好了一份数据在本地

![1633955533970](count().assets/1633955533970.png)

#### 3、检查结果

##### 1.检查count(*)

```sql
select count(*) from test_load; 
```

![](count().assets/1633955663944.png)

##### 2.检查元数据

步骤1、因为我的hive元数据存储在mysql中，所以在mysql中找到我们表的TBL_ID

![1633955823013](count().assets/1633955823013.png)

步骤2、根据test_load的TBL_ID寻找numFiles和numRows

![1633955965860](count().assets/1633955965860.png)

### ②insert ... select ...

```sql
insert [into/overwrite] table 表名 select 列名 from 表名;
```

#### 1、创建表

```sql
create table if not exists test_insert_select(
deptno int,
dname string,
loc int
)
row format delimited fields terminated by '\t';
```

![1633956181820](count().assets/1633956181820.png)

### 2、插入数据

``` sql
insert into table test_insert_select select * from test_load;
```

![1633956491183](count().assets/1633956491183.png)

### 3、检查结果

![1633956582458](count().assets/1633956582458.png)

![1633956639293](count().assets/1633956639293.png)

![1633956660672](count().assets/1633956660672.png)

### ③create table ... as select ...

```sql
create table if not exists 表名 as select 列名 from 表名;
```

#### 1、插入数据

```sql
create table if not exists test_create_as_select as select * from test_load;
```

![1633957048188](count().assets/1633957048188.png)

#### 2、检查结果

![1633957101781](count().assets/1633957101781.png)

![1633957136417](count().assets/1633957136417.png)

![1633957163654](count().assets/1633957163654.png)

### ④create external table ... location ...

```sql
create external table if not exists 表名(字段名) row format delimited fields terminated by '\t' location 'HDFS上的路径';
```

#### 1、hdfs上准备数据

本地数据：

![1633957380221](count().assets/1633957380221.png)

上传到hdfs上:

![1633957453818](count().assets/1633957453818.png)

![1633957467215](count().assets/1633957467215.png)

#### 2、创建表并指定location

```sql
create external table if not exists test_location(
deptno int,
dname string,
loc int
)row format delimited fields terminated by '\t'
location '/test';
```

![1633957627890](count().assets/1633957627890.png)

#### 3、检查结果

![1633957716089](count().assets/1633957716089.png)

![1633957747832](count().assets/1633957747832.png)

![1633957838971](count().assets/1633957838971.png)

<font color="red">无numRows</font>

那numFiles什么时间刷新呢？我们进行下一步测试：

##### 1、向location指定的HDFS位置中再上传文件

![1633958437010](count().assets/1633958437010.png)

##### 2、刷新mysql查看numFiles

![1633958522070](count().assets/1633958522070.png)

此时发现文件数没改变

![1633958577706](count().assets/1633958577706.png)

HDFS文件数没问题

##### 3、运行count(\*)

![1633958664719](count().assets/1633958664719.png)

![1633958694877](count().assets/1633958694877.png)

元数据还是未发生改变

### ⑤import table ... from '...'

```sql
import table 表名 from 'HDFS上的路径';
```

#### 1、首先导出其他表的数据

```sql
export table test_load to '/test_load';
```

![1633959187927](count().assets/1633959187927.png)

![1633959198658](count().assets/1633959198658.png)

#### 2、创建表

```sql
create table if not exists test_import(
deptno int,
dname string,
loc int
)
row format delimited fields terminated by '\t';
```

![1633959285391](count().assets/1633959285391.png)

#### 3、导入数据到表

```sql
import table test_import from '/test_load';
```

#### ![1633959335433](count().assets/1633959335433.png)

#### 4、测试结果

![1633959469836](count().assets/1633959469836.png)

![1633959368465](count().assets/1633959368465.png)

![1633959390635](count().assets/1633959390635.png)

<font color="red">此操作只能在无表或有表无数据下操作，否则会报错，如下所示：</font>

![1633959893112](count().assets/1633959893112.png)

test_import已经导入了test_load数据，此时再导入test_location数据，报错！！！