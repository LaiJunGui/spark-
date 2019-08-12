

#                               RDD编程

## 1文件读写

### 1.1  本地文件系统的数据读写

```python
textFile = sc.textFile("file:///usr/local/spark/mycode/wordcount/word.txt")
```

​    textFile()方法是用来加载文件数据，加载本地文件，必须采用“file:///”开头的这种格式。执行上上面这条命令以后，并不会马上显示结果，因为，Spark采用惰性机制，只有遇到“行动”类型的操作，才会从头到尾执行所有操作。first()是一个“行动”（Action）类型的操作，会启动真正的计算过程，从文件中加载数据到变量textFile中，并取出第一行文本。屏幕上会显示很多反馈信息，这里不再给出，你可以从这些结果信息中，找到word.txt文件中的第一行的内容

```python
>>> textFile.first()
```

​    Spark采用了惰性机制，在执行转换操作的时候，即使我们输入了错误的语句，pyspark也不会马上报错，而是等到执行“行动”类型的语句时启动真正的计算，那个时候“转换”操作语句中的错误就会显示出来，

### 1.2  分布式文件系统HDFS的数据读写

  为了能够读取HDFS中的文件，首先要启动Hadoop中的HDFS组件。可以使用以下命令直接启动Hadoop中的HDFS组件（由于用不到MapReduce组件，所以，不需要启动MapReduce或者YARN）

```shell
$ cd /usr/local/hadoop
$ ./sbin/start-dfs.sh
```

  启动结束后，HDFS开始进入可用状态。如果你在HDFS文件系统中，还没有为当前Linux登录用户创建目录(本教程统一使用用户名hadoop登录Linux系统)**，HDFS文件系统为Linux登录用户开辟的默认目录是“/user/用户名”（注意：是user，不是usr），**

```shell
$./bin/hdfs dfs -mkdir -p /user/hadoop
```

****

使用命令查看一下HDFS文件系统中的目录和文件

```shell
$./bin/hdfs dfs -ls .
#第一条命令等价于第二、三条命令
$./bin/hdfs dfs -ls .
$./bin/hdfs dfs -ls /user/hadoop
```

最后一个点号“.”，表示要查看Linux当前登录用户hadoop在HDFS文件系统中与hadoop对应的目录下的文件，也就是查看HDFS文件系统中“/user/hadoop/”目录下的文件。

查看HDFS文件系统根目录下的内容，使用下面命令

```shell
$./bin/hdfs dfs -ls /
```

将本地文件系统中的“/usr/local/spark/mycode/wordcount/word.txt”上传到分布式文件系统HDFS中（放到hadoop用户目录下）：

```shell
$./bin/hdfs dfs -put /usr/local/spark/mycode/wordcount/word.txt .
```

用命令查看一下HDFS的hadoop用户目录下是否多了word.txt文件，可以使用下面命令列出hadoop目录下的内容

```shell
$./bin/hdfs dfs -ls .
```

使用cat命令查看一个HDFS中的word.txt文件的内容，命令如下

```shell
$./bin/hdfs dfs -cat ./word.txt
```

### 不同文件格式的读写

#### 文本文件

```shell
#本地文件系统中的文本文件加载到RDD中的语句
#当我们给textFile()函数传递一个“包含完整路径的文件名”时，就会把这个文件加载到RDD中。如果我们给textFile()函数传递的不是文件名，而是一个目录，则该目录下的所有文件内容都会被读取到RDD中。
rdd = sc.textFile("file:///usr/local/spark/mycode/wordcount/word.txt")
#把RDD中的数据保存到文本文件
 rdd.saveAsTextFile("file:///usr/local/spark/mycode/wordcount/outputFile")
```

#### JSON文件

```json
{"name":"Michael"}
{"name":"Andy", "age":30}
{"name":"Justin", "age":19}
```

Spark提供了一个JSON样例数据文件，存放在“/usr/local/spark/examples/src/main/resources/people.json”中。

首先来看一下把本地文件系统中的people.json文件加载到RDD中以后，数据是什么形式，请在spark-shell中执行如下操作：

```python
>>> jsonStr = sc.textFile("file:///usr/local/spark/examples/src/main/resources/people.json")
>>> jsonStr.foreach(print)
{"name":"Michael"}
{"name":"Andy", "age":30}
{"name":"Justin", "age":19}
```

Scala中有一个自带的JSON库——scala.util.parsing.json.JSON，可以实现对JSON数据的解析。JSON.parseFull(jsonString:String)函数，以一个JSON字符串作为输入并进行解析，如果解析成功则返回一个Some(map: Map[String, Any])，如果解析失败则返回None。

#                      键值对RDD的创建

## 第一种创建方式：从文件中加载

```python
>>>  lines = sc.textFile("file:///usr/local/spark/mycode/pairrdd/word.txt")
>>> pairRDD = lines.flatMap(lambda line : line.split(" ")).map(lambda word : (word,1))
>>> pairRDD.foreach(print)
(i,1)
(love,1)
(hadoop,1)
(i,1)
(love,1)
(Spark,1)
(Spark,1)
(is,1)
(fast,1)
(than,1)
(hadoop,1)
```

  可以采用多种方式创建键值对RDD，其中一种主要方式是使用map()函数来实现。从代码执行返回信息：pairRDD: org.apache.spark.rdd.RDD[(String, Int)] = MapPartitionsRDD[3] at map at :29，可以看出，返回的结果是键值对类型的RDD，即RDD[(String, Int)]。从pairRDD.foreach(println)执行的打印输出结果也可以看到，都是由(单词,1)这种形式的键值对。

## 第二种创建方式：通过并行集合（列表）创建RDD

```python
>>> list = ["Hadoop","Spark","Hive","Spark"]
>>> rdd = sc.parallelize(list)
>>> pairRDD = rdd.map(lambda word : (word,1))
>>> pairRDD.foreach(print)
(Hadoop,1)
(Spark,1)
(Hive,1)
(Spark,1)
```

# 常用的键值对转换操作

包括reduceByKey()、groupByKey()、sortByKey()、join()、cogroup()等。

reduceByKey(func)的功能是，使用func函数合并具有相同键的值。比如，reduceByKey((a,b) => a+b)，有四个键值对(“spark”,1)、(“spark”,2)、(“hadoop”,3)和(“hadoop”,5)，对具有相同key的键值对进行合并后的结果就是：(“spark”,3)、(“hadoop”,8)。可以看出，(a,b) => a+b这个Lamda表达式中，a和b都是指value，比如，对于两个具有相同key的键值对(“spark”,1)、(“spark”,2)，a就是1，b就是2

groupByKey()的功能是，对具有相同键的值进行分组。比如，对四个键值对(“spark”,1)、(“spark”,2)、(“hadoop”,3)和(“hadoop”,5)，采用groupByKey()后得到的结果是：(“spark”,(1,2))和(“hadoop”,(3,5))。

keys()只会把键值对RDD中的key返回形成一个新的RDD。比如，对四个键值对(“spark”,1)、(“spark”,2)、(“hadoop”,3)和(“hadoop”,5)构成的RDD，采用keys()后得到的结果是一个RDD[Int]，内容是{“spark”,”spark”,”hadoop”,”hadoop”}。

values()只会把键值对RDD中的value返回形成一个新的RDD。比如，对四个键值对(“spark”,1)、(“spark”,2)、(“hadoop”,3)和(“hadoop”,5)构成的RDD，采用values()后得到的结果是一个RDD[Int]，内容是{1,2,3,5}。

sortByKey()的功能是返回一个根据键排序的RDD。

我们经常会遇到一种情形，我们只想对键值对RDD的value部分进行处理，而不是同时对key和value进行处理。对于这种情形，Spark提供了mapValues(func)，它的功能是，对键值对RDD中的每个value都应用一个函数，但是，key不会发生变化。比如，对四个键值对(“spark”,1)、(“spark”,2)、(“hadoop”,3)和(“hadoop”,5)构成的pairRDD，如果执行pairRDD.mapValues(lambda x : x+1)，就会得到一个新的键值对RDD，它包含下面四个键值对(“spark”,2)、(“spark”,3)、(“hadoop”,4)和(“hadoop”,6)。

join(连接)操作是键值对常用的操作。“连接”(join)这个概念来自于关系数据库领域，因此，join的类型也和关系数据库中的join一样，包括内连接(join)、左外连接(leftOuterJoin)、右外连接(rightOuterJoin)等。最常用的情形是内连接，所以，join就表示内连接。
对于内连接，对于给定的两个输入数据集(K,V1)和(K,V2)，只有在两个数据集中都存在的key才会被输出，最终得到一个(K,(V1,V2))类型的数据集。

比如，pairRDD1是一个键值对集合{(“spark”,1)、(“spark”,2)、(“hadoop”,3)和(“hadoop”,5)}，pairRDD2是一个键值对集合{(“spark”,”fast”)}，那么，pairRDD1.join(pairRDD2)的结果就是一个新的RDD，这个新的RDD是键值对集合{(“spark”,1,”fast”),(“spark”,2,”fast”)}。

```python
>>> pairRDD.reduceByKey(lambda a,b : a+b).foreach(print)
(Spark,2)
(Hive,1)
(Hadoop,1)
>>> pairRDD.groupByKey()
PythonRDD[11] at RDD at PythonRDD.scala:48
>>> pairRDD.groupByKey().foreach(print)
('spark', <pyspark.resultiterable.ResultIterable object at 0x7f1869f81f60>)
('hadoop', <pyspark.resultiterable.ResultIterable object at 0x7f1869f81f60>)
('hive', <pyspark.resultiterable.ResultIterable object at 0x7f1869f81f60>)
 >>> pairRDD.keys()
PythonRDD[20] at RDD at PythonRDD.scala:48
>>> pairRDD.keys().foreach(print)
Hadoop
Spark
Hive
Spark
scala> pairRDD.values()
PythonRDD[22] at RDD at PythonRDD.scala:48
scala> pairRDD.values().foreach(print)
1
1
1
1
>>> pairRDD.sortByKey()
PythonRDD[30] at RDD at PythonRDD.scala:48
>>> pairRDD.sortByKey().foreach(print)
(Hadoop,1)
(Hive,1)
(Spark,1)
(Spark,1)
>>> pairRDD.mapValues(lambda x : x+1)
PythonRDD[38] at RDD at PythonRDD.scala:48
>>> pairRDD.mapValues( lambda x : x+1).foreach(print)
(Hadoop,2)
(Spark,2)
(Hive,2)
(Spark,2)
>>> pairRDD1 = sc.parallelize([('spark',1),('spark',2),('hadoop',3),('hadoop',5)])
>>> pairRDD2 = sc.parallelize([('spark','fast')])
>>> pairRDD1.join(pairRDD2)
PythonRDD[49] at RDD at PythonRDD.scala:48 
>>> pairRDD1.join(pairRDD2).foreach(print)
```

# 共享变量

**Spark中的两个重要抽象是RDD和共享变量。**

​    在默认情况下，当Spark在集群的多个不同节点的多个任务上并行运行一个函数时，它会把函数中涉及到的每个变量，在每个任务上都生成一个副本。但是，有时候，需要在多个任务之间共享变量，或者在任务（Task）和任务控制节点（Driver Program）之间共享变量。为了满足这种需求，Spark提供了两种类型的变量：广播变量（broadcast variables）和累加器（accumulators）。广播变量用来把变量在所有节点的内存之间进行共享。累加器则支持在所有不同节点之间进行累加计算（比如计数或者求和）。

# 广播变量

广播变量（broadcast variables）允许程序开发人员在每个机器上缓存一个只读的变量，而不是为机器上的每个任务都生成一个副本。通过这种方式，就可以非常高效地给每个节点（机器）提供一个大的输入数据集的副本。Spark的“动作”操作会跨越多个阶段（stage），对于每个阶段内的所有任务所需要的公共数据，Spark都会自动进行广播。通过广播方式进行传播的变量，会经过序列化，然后在被任务使用时再进行反序列化。这就意味着，显式地创建广播变量只有在下面的情形中是有用的：当跨越多个阶段的那些任务需要相同的数据，或者当以反序列化方式对数据进行缓存是非常重要的。

可以通过调用SparkContext.broadcast(v)来从一个普通变量v中创建一个广播变量。这个广播变量就是对普通变量v的一个包装器，通过调用value方法就可以获得这个广播变量的值，具体代码如下：

```python
>>> broadcastVar = sc.broadcast([1, 2, 3])
>>> broadcastVar.value
[1,2,3]
```

这个广播变量被创建以后，那么在集群中的任何函数中，都应该使用广播变量broadcastVar的值，而不是使用v的值，这样就不会把v重复分发到这些节点上。此外，一旦广播变量创建后，普通变量v的值就不能再发生修改，从而确保所有节点都获得这个广播变量的相同的值。

# 累加器

累加器是仅仅被相关操作累加的变量，通常可以被用来实现计数器（counter）和求和（sum）。Spark原生地支持数值型（numeric）的累加器，程序开发人员可以编写对新类型的支持。如果创建累加器时指定了名字，则可以在Spark UI界面看到，这有利于理解每个执行阶段的进程。
一个数值型的累加器，可以通过调用SparkContext.accumulator()来创建。运行在集群中的任务，就可以使用add方法来把数值累加到累加器上，但是，这些任务只能做累加操作，不能读取累加器的值，只有任务控制节点（Driver Program）可以使用value方法来读取累加器的值。
下面是一个代码实例，演示了使用累加器来对一个数组中的元素进行求和：

```Python
>>> accum = sc.accumulator(0)
>>> sc.parallelize([1, 2, 3, 4]).foreach(lambda x : accum.add(x))
>>> accum.value
10
```

请使用下面命令创建请用RDD实现一个PAT排名器，题目要求

