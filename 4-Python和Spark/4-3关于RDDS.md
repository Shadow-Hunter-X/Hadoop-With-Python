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

RDD.glom()方法返回所有元素分区的列表,RDD.collect()方法将所有元素都传送驱动程序节点。结果[[1, 2, 3, 4, 5]]是列表中的原始集合