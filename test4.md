# Spark MLlib介绍

机器学习强调三个关键词：算法、经验、性能。

在数据的基础上，通过算法构建出模型并对模型进行评估。评估的性能如果达到要求，就用该模型来测试其他的数据；如果达不到要求，就要调整算法来重新建立模型，再次进行评估。如此循环往复，最终获得满意的经验来处理其他的数据。

Spark 立足于内存计算，天然的适应于迭代式计算。Spark提供了一个基于海量数据的机器学习库，它提供了常用机器学习算法的分布式实现，开发者只需要有 Spark 基础并且了解机器学习算法的原理，以及方法相关参数的含义，就可以轻松的通过调用相应的 API 来实现基于海量数据的机器学习过程。其次，Spark-Shell的即席查询也是一个关键

spark.mllib 包含基于RDD的原始算法API

spark.ml 则提供了基于DataFrames 高层次的API，可以用来构建机器学习工作流（PipeLine）。

目前MLlib支持的主要的机器学习算法：

![123](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1565962597932.png)

# 机器学习工作流

DataFrame：使用Spark SQL中的DataFrame作为数据集，它可以容纳各种数据类型。 较之 RDD，包含了 schema 信息，更类似传统数据库中的二维表格。它被 ML Pipeline 用来存储源数据

Transformer：翻译成转换器，是一种可以将一个DataFrame转换为另一个DataFrame的算法。比如一个模型就是一个 Transformer。

Estimator：翻译成估计器或评估器，它是学习算法或在训练数据上的训练方法的概念抽象。在 Pipeline 里通常是被用来操作 DataFrame 数据并生产一个 Transformer。从技术上讲，Estimator实现了一个方法fit（），它接受一个DataFrame并产生一个转换器。如一个随机森林算法就是一个 Estimator，它可以调用fit（），通过训练特征数据而得到一个随机森林模型。

Parameter：Parameter 被用来设置 Transformer 或者 Estimator 的参数。现在，所有转换器和估计器可共享用于指定参数的公共API。ParamMap是一组（参数，值）对

PipeLine：翻译为工作流或者管道。工作流将多个工作流阶段（转换器和估计器）连接在一起，形成机器学习的工作流，并获得结果输出。

要构建一个 Pipeline工作流，首先需要定义 Pipeline 中的各个工作流阶段PipelineStage，（包括转换器和评估器），比如指标提取和转换模型训练等

```python
pipeline =Pipeline(stages=[stage1,stage2,stage3])
```

然后就可以把训练数据集作为输入参数，调用 Pipeline 实例的 fit 方法来开始以流的方式来处理源训练数据。这个调用会返回一个 PipelineModel 类实例，进而被用来预测测试数据的标签。

![1565965039499](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1565965039499.png)

```python

from pyspark.ml import Pipeline
from pyspark.ml.classification import LogisticRegression
from pyspark.ml.feature import HashingTF, Tokenizer
spark = SparkSession.builder.master("local").appName("Word Count").getOrCreate()
# Prepare training documents from a list of (id, text, label) tuples.
training = spark.createDataFrame([
    (0, "a b c d e spark", 1.0),
    (1, "b d", 0.0),
    (2, "spark f g h", 1.0),
    (3, "hadoop mapreduce", 0.0)
], ["id", "text", "label"])
tokenizer = Tokenizer(inputCol="text", outputCol="words")
hashingTF = HashingTF(inputCol=tokenizer.getOutputCol(), outputCol="features")
lr = LogisticRegression(maxIter=10, regParam=0.001)
pipeline = Pipeline(stages=[tokenizer, hashingTF, lr])
model = pipeline.fit(training)
test = spark.createDataFrame([
    (4, "spark i j k"),
    (5, "l m n"),
    (6, "spark hadoop spark"),
    (7, "apache hadoop")
], ["id", "text"])
prediction = model.transform(test)
selected = prediction.select("id", "text", "probability", "prediction")
for row in selected.collect():
    rid, text, prob, prediction = row
    print("(%d, %s) --> prob=%s, prediction=%f" % (rid, text, str(prob), prediction))
 

```

```
(4, spark i j k) --> prob=[0.155543713844,0.844456286156], prediction=1.000000
(5, l m n) --> prob=[0.830707735211,0.169292264789], prediction=0.000000
(6, spark hadoop spark) --> prob=[0.0696218406195,0.93037815938], prediction=1.000000
(7, apache hadoop) --> prob=[0.981518350351,0.018481649649], prediction=0.000000
```

# 特征提取

特征抽取：从原始数据中抽取特征
特征转换：特征的维度、特征的转化、特征的修改
特征选取：从大规模特征集中选取一个子集

```python
from pyspark.ml.feature import HashingTF,IDF,Tokenizer
sentenceData = spark.createDataFrame([(0, "I heard about Spark and I love Spark"),(0, "I wish Java could use case classes"),(1, "Logistic regression models are neat")]).toDF("label", "sentence")
tokenizer = Tokenizer(inputCol="sentence", outputCol="words")
wordsData = tokenizer.transform(sentenceData)
hashingTF = HashingTF(inputCol="words", outputCol="rawFeatures", numFeatures=20)
featurizedData = hashingTF.transform(wordsData)
idf = IDF(inputCol="rawFeatures", outputCol="features")
idfModel = idf.fit(featurizedData)
rescaledData = idfModel.transform(featurizedData)
rescaledData.select("label", "features").show()
```

