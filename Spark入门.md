Spark和mapreduce有哪些区别。请用具体例子说明

MapReduce是hadoop中的一个计算框架，具体核心是将编程抽象为map和reduce两个方法，程序员只需要编写map和reduce两个方法的具体代码就可以完成一个分布式计算操作，大大的简化了开发的难度，使开发难度减小。同时MapReduce程序是基于分布式集群运行，所以可以处理大量的数据。

  正是因为MapReduce把具体编程过程抽象为map和reduce两个过程，使程序开发难度降低，其实也限制了它的表达能力，使得MapReduce表达能力有限。
每一个map阶段结束后都需要将map的结果写入到hdfs上或则本地磁盘中，每一个reduce阶段都需要从远程拷贝map阶段输出的结果，作为reduce阶段的输入数据，所以磁盘IO开销较大。
因为MapReduce启动耗时也不小，并且每次map和reduce过程都要设计到对磁盘的读写，所以MapReduce程序的延迟性较高。

因为每次的map和reduce阶段都需要进行落盘，每次运行map或reduce过程中都需要从磁盘上去读取数据，所以这样的高延迟和高IO开销计算过程并不适合于迭代计算，使用MapReduce进行迭代计算将会非常的耗资源。MapReduce是hadoop中的一个计算框架，具体核心是将编程抽象为map和reduce两个方法，程序员只需要编写map和reduce两个方法的具体代码就可以完成一个分布式计算操作，大大的简化了开发的难度，使开发难度减小。同时MapReduce程序是基于分布式集群运行，所以可以处理大量的数据。
--------------------- 

Spark中其实也借鉴了MapReduce的一些思想，但是在借鉴MapReduce的优点的同时，很好的解决了MapReduce所面临的问题。

相比于MapReduce，spark主要具有如下优点：

Spark的计算模式也是输入MapReduce，但不局限于map和reduce操作，还提供了多种数据集操作类型，编程模型比MapReduce更加的灵活
Spark提供了内存计算，可将中间结果放到内存中，不再像MapReduce那样，每一个map或reduce阶段完成后都进行落盘操作，对于迭代计算的效率更高。
Spark基于DAG（有向无环图）的任务调度机制，要优于MapReduce的迭代机制
Spark将数据载入内存中以后，之后的迭代计算都可以直接使用内存中的中间结果作运算，从而避免了从磁盘中频繁的读取数据，提高运行效率。
--------------------- 
![](G:\spark学习\mapreduce和spark的区别.png)

# Spark的工作原理

![](G:\spark学习\spark工作流程.png)

spark-submit 提交了应用程序的时候，提交spark应用的机器会通过反射的方式，创建和构造一个Driver进程，Driver进程执行Application程序，
Driver根据sparkConf中的配置初始化SparkContext,在SparkContext初始化的过程中会启动DAGScheduler和taskScheduler
taskSheduler通过后台进程，向Master注册Application，Master接到了Application的注册请求之后，会使用自己的资源调度算法，在spark集群的worker上，通知worker为application启动多个Executor。
Executor会向taskScheduler反向注册。
Driver完成SparkContext初始化
application程序执行到Action时，就会创建Job。并且由DAGScheduler将Job划分多个Stage,每个Stage 由TaskSet 组成
DAGScheduler将TaskSet提交给taskScheduler
taskScheduler把TaskSet中的task依次提交给Executor

Executor在接收到task之后，会使用taskRunner来封装task（TaskRuner主要将我们编写程序，也就是我们编写的算子和函数进行拷贝和反序列化）,然后，从Executor的线程池中取出一个线程来执行task。就这样Spark的每个Stage被作为TaskSet提交给Executor执行，每个Task对应一个RDD的partition,执行我们的定义的算子和函数。直到所有操作执行完为止。
--------------------- 
rdd的本质是什么？

RDD是一个弹性可复原的分布式数据集！

RDD是一个逻辑概念，一个RDD中有多个分区，一个分区在Executor节点上执行时，他就是一个迭代器。

一个RDD有多个分区，一个分区肯定在一台机器上，但是一台机器可以有多个分区，我们要操作的是分布在多台机器上的数据，而RDD相当于是一个代理，对RDD进行操作其实就是对分区进行操作，就是对每一台机器上的迭代器进行操作，因为迭代器引用着我们要操作的数据
--------------------- 
### 二、RDD的五大特性 

RDD是由多个分区组成的集合

每个分区上会有一个函数作用在上面，实现分区的转换

RDD与RDD之间存在依赖关系，实现高容错性

如果RDD里面装的是（K-V）类型的，有分区器

如果从HDFS这种文件系统中创建RDD，会有最佳位置，是为了数据本地化