#Spark知识拾遗

###模块图

![Spark模块图](http://spark.apache.org/images/spark-stack.png)

这是来自Spark官网(<http://spark.apache.org/>)的模块图，总体分为Spark Core、Spark Streaming、Spark SQL、MLib、GraphX几大模块。

_需要注意的就是写代码的时候得分别import才行，因为他们分属源码中不同的包_

###部署图

![Spark部署图](https://github.com/JerryLead/SparkInternals/blob/master/markdown/PNGfigures/deploy.png)

来自大牛JerryLead的绘图(<https://github.com/JerryLead/SparkInternals/blob/master/markdown/1-Overview.md>)， 很清晰地看到Master、Worker、Executor、Task的关系

###编译Jar包

我是使用sbt的，配置好build.sbt之后(主要是依赖包的配置)，使用sbt package打包，如果要用fat-jar的方式，还需要配置一下sbt-assmeble插件， 并且在build.sbt中配置冲依赖冲突的解决方式。

###RDD常用操作
基本就是map、flatmap、filter、distinct、sample、union、top、agg、froeache等

###Cache Persist or CheckPoint？
这几个都是为了减少数据的重复计算，__待补充__

###数据源
Spark支持多种数据源：包括HDFS、Hive、ES、MySQL、PgSQL、Cassandra、HBase等



  
