spark是一种集群计算框架, 它使用内存中的基元来使程序运行速度比Hadoop MapReduce应用程序快100倍。Spark应用程序由驱动程序组成,控制跨群集执行并行操作。Spark提供的主要编程抽象称为弹性分布式数据集 (RDDs)。RDDs 是在群集节点上分区的元素集合, 可以并行操作

Spark被创造了在许多平台上运行并且可使用许多语言开发。目前,Spark可以运行在 hadoop 1.0, hadoop 2.0, Apache Mesos, 或一个独立的Spark集群。Spark本身也支持 Scala、Java、Python 和 R。除了这些功能外, Spark还可以与命令行 shell 交互使用。

本章从一个示例Spark脚本开始。然后介绍了PySpark,并详细介绍了RDDs的示例。本章以Python中编写的Spark程序为例进行总结。

<h2>使用PySpark编写WordCount程序</h2>

示例4-1 中的代码在PySpark中实现了WordCount算法。它假定数据文件input.txt加载在HDFS下/user/hduser/input中, 输出将放在 HDFS下/user/hduser/output

* Example 4-1. python/Spark/word_count.py

```
from pyspark import SparkContext
def main():
    sc = SparkContext(appName='SparkWordCount')
    input_file = sc.textFile('/user/hduser/input/input.txt')
    counts = input_file.flatMap(lambda line: line.split()) \
        .map(lambda word: (word, 1)) \
        .reduceByKey(lambda a, b: a + b)
    counts.saveAsTextFile('/user/hduser/output')
    sc.stop()

if __name__ == '__main__':
    main()
```

要执行Spark应用程序, 将文件的名称传递给Spark提交脚本

```
$ spark-submit --master local word_count.py
```

作业在运行时,会将大量文本打印到控制台。word_count Spark脚本的结果显示如下, 可在HDFS下/user/hduser/output/part-00000 中找到。

```
(u'be', 2)
(u'jumped', 1)
(u'over', 1)
(u'candlestick', 1)
(u'nimble', 1)
(u'jack', 3)
(u'quick', 1)
(u'the', 1)
```

<h3>WordCount说明</h3>
本节介绍在word_count在Spark脚本中转换。
* 第一个语句创建一个SparkContext对象。此对象告诉Spark如何以及在何处访问群集。

```
sc = SparkContext(appName='SparkWordCount')
```

* 第二个语句使用SparkContext将文件从HDFS并将其存储在变量input_file中
```
input_file = sc.textFile('/user/hduser/input/input.txt')
```

* 第三个语句对输入执行多个转换数据。Spark自动平衡这些转换运行跨多台计算机

```
counts = input_file.flatMap(lambda line: line.split()) \
.map(lambda word: (word, 1)) \
.reduceByKey(lambda a, b: a + b)
```

* 第四条语句将结果存储到 HDFS:

```
counts.saveAsTextFile('/user/hduser/output')
```

* 第五条语句关闭了SparkContext:

```
sc.stop()
```

