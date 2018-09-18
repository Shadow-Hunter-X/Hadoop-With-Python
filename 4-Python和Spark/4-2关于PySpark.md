<h2>关于Spark</h2>

PySpark是Spark的Python API,PySpark允许Spark应用程序从交互式shell或Python进行创建。

在执行Spark内的任何代码之前, 应用程序必须创建一个SparkContext对象。SparkContext对象告诉火花如何以及在何处访问群集。master属性是一个群集URL, 用于确定Spark的执行位置。对于master最常见的值是

* local     单线程运行Spark
* local[n]  n个线程运行Spark
* spark://HOST:PORT  连接到独立的Spark集群
* mesos://HOST:PORT  连接到mesos集群

<h3>交互模型</h3>

在Spark Shell中, SparkContext当shell启动时被创建。SparkContext在存储在变量sc中。当shell启动时可以使用参数 **--master** 来启动Spark Master模式。

```
$ pyspark --master local[4]
...
Welcome to
____ __
/ __/__ ___ _____/ /__
_\ \/ _ \/ _ `/ __/ '_/
/__ / .__/\_,_/_/ /_/\_\ version 1.5.0
/_/
Using Python version 2.7.10 (default, Jul 13 2015 12:05:58)
SparkContext available as sc, HiveContext available as sqlContext.
>>>
```

<h3>Spark自包含的应用程序</h3>
自包含的应用程序必须首先创建一个 SparkContext对象在使用任何Spark方法之前。可以设置master模式在SparkContext()方法被调用的时候。

```
sc = SparkContext(master='local[4]')
```
要执行自包含的应用程序, 必须将它们提交到"spark-submit"脚本。Spark提交脚本包含许多选项;要查看完整的列表, 使用 spark-submit --help

```
$ spark-submit --master local spark_app.py
```