#Spark知识拾遗

###模块图

![Spark模块图](http://spark.apache.org/images/spark-stack.png)

这是来自Spark官网(<http://spark.apache.org/>)的模块图，总体分为Spark Core、Spark Streaming、Spark SQL、MLib、GraphX几大模块。

_需要注意的就是写代码的时候得分别**import**才行，因为他们分属源码中不同的包_

###部署图

![Spark部署图](https://github.com/JerryLead/SparkInternals/blob/master/markdown/PNGfigures/deploy.png)

来自大牛JerryLead的绘图(<https://github.com/JerryLead/SparkInternals/blob/master/markdown/1-Overview.md>)， 很清晰地看到Master、Worker、Executor、Task的关系

###编译Jar包

我是使用sbt的，配置好build.sbt之后(主要是依赖包的配置)，使用sbt package打包，如果要用fat-jar的方式，还需要配置一下sbt-assmeble插件， 并且在build.sbt中配置冲依赖冲突的解决方式。

###RDD常用操作
基本就是map、mappartition、flatmap、filter、distinct、sample、union、top、agg、froeache等

###Cache Persist or CheckPoint？
这几个都是为了减少数据的重复计算，__待补充__

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
    Spark On Yarn的内存模型 __待补充__
    
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
    对于SparkStreaming来讲，实时性要求较高，不大能接受任务的终端或者暂停，最好使用CMS的GC方式
    
###Spark Mlib
提供了多种机器学习函数：TF-IDF、W2V、Scaling、Normalization、SVM、LR、决策树、随机森林、KMeans、PCA、协同过滤等。





  
