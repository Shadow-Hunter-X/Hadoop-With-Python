<h2>关于RDDS</h2>

弹性分布数据集(RDDs)是Spark中的基本编程抽象。RDDs是不变的数据集合,跨计算机进行分区,使操作可以并行。RDDs 可以通过多种方式构造:通过并行化现有的Python集合,通过引用外部存储系统(如 HDFS) 中的文件, 或者将转换应用到现有的RDDs

<h3>从集合创建 RDDs</h3>

RDDs可以通过调用Spark上下文来从Python Context.parallelize()方法。复制集合的元素以形成可并行操作的分布式数据集。下面的示例从 Python 列表中创建一个并行集合:

```
>>> data = [1, 2, 3, 4, 5]
>>> rdd = sc.parallelize(data)
>>> rdd.glom().collect()
...
[[1, 2, 3, 4, 5]]
```

RDD.glom()方法返回所有元素分区的列表,RDD.collect()方法将所有元素都传送驱动程序节点。结果[[1, 2, 3, 4, 5]]是列表中的原始集合。
若要指定RDD应创建的分区数,可以将第二个参数传递到parallelize()方法。下面的示例从上一个示例中的相同 Python 集合创建一个 RDD, 但此时将创建四个分区

```
>>> rdd = sc.parallelize(data, 4)
>>> rdd.glom().collect()
...
[[1], [2], [3], [4, 5]]
```

使用glom()和collect()方法, 在此创建的 RDD示例包含四个内部列表:[1]、[2]、[3] 和 [4、5],表的数目表示在RDD分区的数目。

<h3>从外部源创建 RDDs</h3>
还可以使用SparkContext.text.File()方法从文件创建RDDs。Spark可以读取驻留在本地文件系统上的文件、Hadoop、亚马逊 S3 支持的任何存储源。Spark支持文本文件、SequenceFiles、任何其他Hadoop InputFormat、目录、压缩文件和通配符,例如,my/directory/*.txt。下面的示例从位于本地文件系统上的文件创建分布式数据集

```
>>> distFile = sc.textFile('data.txt')
>>> distFile.glom().collect()
...
[[u'jack be nimble', u'jack be quick', u'jack jumped over the candlestick']]
```

与以前一样, glom()和collect()方法允许RDD显示在其分区中。这个测试结果,distFile只有一个个分区。
与parallelize()方法类似, textFile()方法第二个参数指定要创建的分区数。下面的示例从输入文件创建一个具有三个分区的RDD:

```
>>> distFile = sc.textFile('data.txt', 3)
>>> distFile.glom().collect()
...
[[u'jack be nimble', u'jack be quick'], [u'jack jumped over
the candlestick'], []]
```

<h3>RDD操作</h3>

RDD支持两种的操作方式: transformations 和 actions。transformations从现有的数据集创建新的dataset, actions在数据集上运行计算并将结果返回给驱动程序。
transformations是惰性的: 即,它们的结果立即不计算。相反, Spark会记住应用到基数据集的所有transformations。当Actions要求将结果返回给驱动程序时, 将计算transformations。这使得Spark能够有效地运行,只在操作之前传输转换的结果。
默认情况下, 每次对其执行操作时都可能重新计算过期transformations。这使得Spark能够有效地利用内存, 但如果不断处理相同的transformations, 它可能会利用更多的处理资源。为了确保只计算一次转换, 可以使用 RDD.cache()方法在内存中保留生成的RDD。

<h4>RDD Workflow</h4>
使用 RDDs 的一般工作流如下所示:
* 从数据源创建 RDD。
* 将转换应用于 RDD。
* 对 RDD 应用行动。
下面的示例使用此工作流计算数字文件中的字符:

```
>>> lines = sc.textFile('data.txt')
>>> line_lengths = lines.map(lambda x: len(x))
>>> document_length = line_lengths.reduce(lambda x,y: x+y)
>>> print document_length
59
```