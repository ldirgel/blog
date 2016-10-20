#Hive知识拾遗

###什么是Hive？
Hive是位于用户和HDFS上的分布式数据中间的重要一层
+ 从用户角度讲，Hive提供类似SQL的查询、统计接口
+ 从执行层次看，Hive把用户的类似SQL的查询、统计行为转化为MapReduce或者Spark任务执行
+ 从储存层次看，Hive管理HDFS上的数据，虚拟组织成DB、Table这种结构

###Metastore
负责储存Hive所管理的数据的元数据

	*什么是元(meta)数据？就是一些表模式(schema)，表位置之类的*
Metastore其实分为两个部分
+ MetaStore Server
+ MetaStore DB

下面是三种配置图
+ Embedded模式：Server和DB在Hive Service进程内 -> 不支持多用户连接
+ Local模式：Server在Hive Server内，DB是独立的进程
+ Remote模式：Server与Hive Server独立

###Data
元数据的目的还是管理数据，Hive提供托管表和外部表两种方式来管理数据
+ 托管表：Hive会把数据移动到自己的warehouse目录，drop table时删除数据和元数据
+ 外部表：Hive不移动数据，drop table时也只删除元数据

###数据储存
Hive所管理的表可以指定底层的存储方式为面向行的SequenceFile或者面向列的RC、ORC、Parquet等
+ 面向行的储存对于整行读取和随机查询效率比较高
+ 面向列的方式适用于列数很多但是查询只查某些列的情况，而且由于同一列的类型一致，可以大大提高压缩比
	+ Parquet最开始就设计成支持PB，Thrift嵌套列查询，不支持update也不支持传统数据库的ACID
	+ ORC源于RC，支持update，支持事务，但是想写PB之类的嵌套列查询很麻烦

###Hive SQL
Hive SQL虽然与SQL有区别，但是基本上是差不多的。基本的Select，join，groupby等都支持的，但是不支持子查询。

HiveSQL中值得一说的是窗口函数，比如分组内的排序、部分汇总，都可以使用窗口函数，关键词是sum over partition by 

###Hive SQL的简单优化
这里只简单的提几点，更详细的优化方式可以在网上找到
+ Join时小表放左边（因为MR普通Join缓存左表，遍历右表）
+ 合理使用mapjoin
+ 减小join输入表的大小（先where，groupby再Join）
+ 对表进行一级甚至二级分区
	
###数据倾斜
HiveSQL涉及的数据倾斜主要分为2块
+ Group by时的数据倾斜
	+ 加大reduce个数
	+ 设定groupby.skewindata = true（通过两次MR来解决数据倾斜）
	+ hive.map.aggr 决定是不是在map先combine一次，可以有两个参数来判断要不要先combine
		+ hive.groupby.mapaggr.checkinterval = 100000，hive.map.aggr.hash.min.reduction = 0.5
+ Join时的数据倾斜
	+ 检查是不是Key有Null
	+ set hive.optimize.skewjoin = true（这个参数可以对join的key进行拆分）





  
	
	
