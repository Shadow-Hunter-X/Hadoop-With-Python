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

第一条语句从外部数据文件data.txt创建RDD，但是这个文件并不会再此时被载入，对于参数lines只是只是只想这个文件的指针。第二条语句执行transformations在RDD上，使用map函数计算文本中每行中字符数。参数line_lengt并不糊马上被计算，由于transformations操作惰性的。最后reduce()函数被调用，这是一个action操作,在此时Spark将计算分解成多个任务在多台服务器上执行，在每台服务器上都对其本地的数据执行map和reducer操作，将计算的结果返回给启动程序。
如果应用程序想再次使用参数line_lengths,那么最好是将map transformations的结果保存，以防止map过程被重复执行。以下语句，将会将line_lengths参数存放到内存中在第一次计算完毕之后:

```
>>> line_lengths.persist()
```

<h4>python lambda函数</h4>

许多Spark的transformations和actions需要函数要从驱动程序中传递的对象, 以便在群集上运行。定义和传递函数的最简单方法是通过使用Python lambda 函数

Lambda函数是在运行时创建的匿名函数 (即, 它们没有名称)。它们可以在需要函数对象的任何位置使用, 并且在语法上仅限于单个表达式。以下的示例展示计算两个数之和的lambda表达式：
```
lamdba a , b : a + b
```
通过关键字lambda来定义lamdba函数，函数参数通过逗号分割，在参数后使用冒号，将参数和表达式区分开，函数表达式是为提供的参数生成结果的单个表达式。

在上一个Spark示例中, map()函数使用以下lambda 函数,这个lambda接受一个参数，并返回这个参数的长度：
```
lambda x: len(x)
```

<h4>Transformations</h4>
Transformations从现有的数据集创建新dataset。惰性的转换评估允许Spark记住应用到基RDD的转换集。这使Spark能够优化所需的计算

接下来介绍一些Spark最常见的transformations。有关transformations的完整列表,请参阅Spark的Python RDD API 文档。

* map函数

map函数通过应用一个函数返回一个新的RDD,到源的每个元素。下面的示例将源 RDD 的每个元素乘以两个
```
>>> data = [1, 2, 3, 4, 5, 6]
>>> rdd = sc.parallelize(data)
>>> map_result = rdd.map(lambda x: x * 2)
>>> map_result.collect()
[2, 4, 6, 8, 10, 12]
```

* filter函数

filter(功能)函数返回一个新的 RDD, 其中包含只有提供的函数返回为 true 的源元素。下面的示例仅返回源 RDD 中的偶数:

```
>>> data = [1, 2, 3, 2, 4, 1]
>>> rdd = sc.parallelize(data)
>>> distinct_result = rdd.distinct()
>>> distinct_result.collect()
[4, 1, 2, 3]
```

* ﬂatMap

flatMap(功能) 函数类似于map()函数, 但它返回的结果的扁平化个数数据。为了进行比较,下面的示例返回源RDD及其正方形中的原始元素。使用map()函数的示例以列表中的列表形式返回对。

```
>>> data = [1, 2, 3, 4]
>>> rdd = sc.parallelize(data)
>>> map = rdd.map(lambda x: [x, pow(x,2)])
>>> map.collect()
[[1, 1], [2, 4], [3, 9], [4, 16]]

使用flatMap()函数

>>> rdd = sc.parallelize()
>>> flat_map = rdd.flatMap(lambda x: [x, pow(x,2)])
>>> flat_map.collect()
[1, 1, 2, 4, 3, 9, 4, 16]

```
<h4>Actions</h4>

Actions会引起Spark来计算transformations。在群集上计算transformations后, 结果将返回给驱动程序。下面一节介绍一些Spark最常见的动作。有关操作的完整列表, 请参阅Spark的Python RDD API文档。

* reduce

reduce()方法在RDD聚合元素,使用一个函数, 它需要两个参数并返回一个。在减少方法中使用的函数是交换和关联的, 确保可以并行地计算。下面的示例返回RDD中所有元素的乘积。
```
>>> data = [1, 2, 3]
>>> rdd = sc.parallelize(data)
>>> rdd.take(2)
[1, 2]
```

*collect 

collect()方法将RDD的所有元素作为数组返回,下面的示例返回RDD中的所有元素。
```
>>> data = [1, 2, 3, 4, 5]
>>> rdd = sc.parallelize(data)
>>> rdd.collect()
[1, 2, 3, 4, 5]
```
必须注意, 在大型数据集上调用collect()可能会导致驱动程序耗尽内存。为了检查大型的RDDs,tabke()和collect()方法可以用来检查大型RDD上的前n个元素。下面的示例将RDD的前100个元素返回给驱动程序:

```
>>> rdd.take(100).collect()
```

* takeOrdered

takeOrdered (n,ke=func)方法返回RDD的第一个n个元素,以自然顺序的方式,或者由函数功能所指定。下面的示例以降序返回RDD的前四个元素

```
>>> data = [6,1,5,2,4,3]
>>> rdd = sc.parallelize(data)
>>> rdd.takeOrdered(4, lambda s: -s)
[6, 5, 4, 3]
```