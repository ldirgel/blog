#Spark知识拾遗

###模块图

![Spark模块图](https://github.com/ldirgel/blog/blob/master/Figures/spark-stack.png)

这是来自Spark官网(<http://spark.apache.org/>)的模块图，总体分为Spark Core、Spark Streaming、Spark SQL、MLib、GraphX几大模块。

_需要注意的就是写代码的时候得分别**import**才行，因为他们分属源码中不同的包_

###部署图

![Spark部署图](https://github.com/ldirgel/blog/blob/master/Figures/spark-deploy.png)

来自大牛JerryLead的绘图(<https://github.com/JerryLead/SparkInternals/blob/master/markdown/1-Overview.md>)， 很清晰地看到Master、Worker、Executor、Task的关系

###编译Jar包

我是使用sbt的，配置好build.sbt之后(主要是依赖包的配置)，使用sbt package打包，如果要用fat-jar的方式，还需要配置一下sbt-assmeble插件， 并且在build.sbt中配置冲依赖冲突的解决方式。

###RDD常用操作
基本就是map、mappartition、flatmap、filter、distinct、sample、union、top、agg、foreach等

###Cache, Persist or CheckPoint?
这几个都是为了减少数据的重复计算

####Cache/Persist
  + Cache = Persist(MEMORY_ONLY)
  + 只能cache在代码中显式看得到的RDD
  + Cache使用memory，Persist可以指定disk还是memory
  + 会先判断一下是否被CheckPoint了，如果是就直接读，然后判断是不是能存下，存不下就根据FIFO原则先drop**其他RDD**之前被cache的partition
  + 用的时候先去driver查一下，cache了没有，cache在哪，然后从本机或者其他机器拿数据
 
####CheckPoint
  + Job结束后单独启动专门的Job去完成的，所以这个RDD会被计算两次，最好是先把他cache下来，这样第二次就不用重新计算了
  + 使用硬盘
  + CheckPoint之后会清除这个RDD的Dependency，并把他的父RDD设为CPRDD
  + RDDCheckpointData 管理所有被cp的RDD，用的时候去查一下，然后从HDFS读就行了
  
####区别
  1. Lineage
    + Cache会记住之前的computing chain
    + CheckPoint直接落地HDFS，消除了lineage
  2. 生命周期
    + Persist的RDD由blockManager管理，一旦driver运行结束，就没了（blockManager的local文件被删除）
    + CheckPoint的RDD一直存在，下一个driver也能用

###数据源
Spark支持多种数据源：包括HDFS、Hive、ES、MySQL、PgSQL、Cassandra、HBase等

###共享变量
accumulator和broadcast：
+ 前者在driver创建，由executor累加，且只有driver可以读取这个值
+ 后者只广播一次，会在所有executor上存一份，这个值的更改对其他executor不可见

###Spark On Yarn

+ yarn-cluster模式属于托管，把任务提交上去，指定输出路径即可
+ yarn-client模式属于交互，driver在本机，等待运算结果返回
+ spark-submit指令可以通过 --conf指定额外参数(**但代码中设置的conf参数优先级更高**) --jar指定额外依赖包 

    ####一些常用的参数
    + spark.speculation = true
    + spark.executor.cores = 2
    + spark.executor.num = 10
    
    ####内存模型
    Spark On Yarn的内存模型 
    
	![Spark-On-Yarn Memory](https://github.com/ldirgel/blog/blob/master/Figures/Spark-On-Yarn.png)
	+ AM的Container内存由 spark.yarn.am.memory + spark.yarn.am.memoryOverhead 确定
	+ Executor的cache-storage由 (executor.memory + executor.memoryOverhead) 和 spark.storage.memoryFraction 共同确定
	+ Executor的shuffle-storage由 (executor.memory + executor.memoryOverhead) 和 spark.shuffle.memoryFraction 共同确定
	+ Executor的Container内存由 executor.memory + executor.memoryOverhead 确定
	+ Container申请的内存大小必须为 yarn.scheduler.minimum-allocation-mb 的整数倍
    
    ####并行度控制
    repartitoin、coalese
    如何判断？如果任务瞬间完成，说明并行度过高

###Spark SQL
DataFrame可以用来很方便的进行相关处理，常见函数包括aggs、count、max等

###Spark Streaming
SparkStreaming以窗口为最小单位形成RDD，然后在Spark进行处理。

  + 支持Kafka、Flume等多种数据源
  + 需要设置正确的窗口大小和步长
  + 容错
    + Driver：启动时使用getOrCreate
    + Executor：和Spark的容错机制一致
    + Receiver：Spark对Flume的拉式接收器提供临时的数据储存

    ####合理的窗口大小
    先从大一点的开始，逐步缩小
    
    ####增加并行度
    + 使用多个Receiver并将输入union
    + 对输入repartition
    
    ####GC
    对于SparkStreaming来讲，实时性要求较高，不大能接受任务的中断或者暂停，最好使用CMS的GC方式
    
###Spark Mlib
提供了多种机器学习函数：TF-IDF、W2V、Scaling、Normalization、SVM、LR、决策树、随机森林、KMeans、PCA、协同过滤等。





  
