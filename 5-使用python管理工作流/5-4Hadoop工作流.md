<h2>Hadoop工作流</h2>
本节包含控制MapReduce和在hadoop集群上运行pig任务的工作流

<h3>配置文件</h3>
本节中的示例需要一个luigi配置文件,client.cfg,指定HadoopStreaming Jar的位置和pig主目录的路径。配置文件应位于当前工作目录和配置文件。Example 5-2示例如下：

* Example 5-2. python/Luigi/client.cfg
```
[hadoop]
streaming-jar: /usr/lib/hadoop-xyz/hadoop-streaming-xyz-123.jar
[pig]
home: /usr/lib/pig
```

<h3>mapreduce in luigi</h3>
luigi脚本可以使用hadoop Streaming控制在hadoop群集上执行MapReduce作业。
* Example 5-3. python/Luigi/luigi_mapreduce.py
```
import luigi
import luigi.contrib.hadoop
import luigi.contrib.hdfs
class InputFile(luigi.ExternalTask):
    """
    A task wrapping the HDFS target
    """
    input_file = luigi.Parameter()
    def output(self):
    """
    Return the target on HDFS
    """
        return luigi.contrib.hdfs.HdfsTarget(self.input_file)

class WordCount(luigi.contrib.hadoop.JobTask):
"""
A task that uses Hadoop streaming to perform WordCount
"""
    input_file = luigi.Parameter()
    output_file = luigi.Parameter()
    # Set the number of reduce tasks
    n_reduce_tasks = 1
    def requires(self):
    """
    Read from the output of the InputFile task
    """
        return InputFile(self.input_file)

    def output(self):
    """
    Write the output to HDFS
    """
        return luigi.contrib.hdfs.HdfsTarget(self.output_file)

    def mapper(self, line):
    """
    Read each line and produce a word and 1
    """
        for word in line.strip().split():
            yield word, 1

    def reducer(self, key, values):
    """
    Read each word and produce the word and the sum of its values
    """
        yield key, sum(values)

if __name__ == '__main__':
    luigi.run(main_task_cls=WordCount)
```

luigi支持Hadoop Streaming 流。实现MapReduce作业的任务必须luigi.contrib.hadoop.JobTask子类。可以重写mapper()和reducer()方法以实现映射并减少 MapReduce作业的方法。

以下命令执行workflow，从/user/hduser/input.txt读取数据并将结果存储在/user/hduser/wordcount:
```
$ python luigi_mapreduce.py --local-scheduler \
--input-file /user/hduser/input/input.txt \
--output-file /user/hduser/wordcount
```

<h3>Pig in Luigi<>

luigi可用于控制在Hadoop群集上执行pig脚本：

* Example 5-4. python/Luigi/luigi_pig.py
```
import luigi
import luigi.contrib.pig
import luigi.contrib.hdfs

class InputFile(luigi.ExternalTask):
"""
A task wrapping the HDFS target
"""
    input_file = luigi.Parameter()
    def output(self):
        return luigi.contrib.hdfs.HdfsTarget(self.input_file)

class WordCount(luigi.contrib.pig.PigJobTask):
"""
A task that uses Pig to perform WordCount
"""
    input_file = luigi.Parameter()
    output_file = luigi.Parameter()
    script_path = luigi.Parameter(default='pig/wordcount.pig')

    def requires(self):
    """
    Read from the output of the InputFile task
    """
        return InputFile(self.input_file)

    def output(self):
    """
    Write the output to HDFS
    """
        return luigi.contrib.hdfs.HdfsTarget(self.output_file)

    def pig_parameters(self):
    """
    A dictionary of parameters to pass to pig
    """
        return {'INPUT': self.input_file, 'OUTPUT': self.output_file}

    def pig_options(self):
    """
    A list of options to pass to pig
    """
        return ['-x', 'mapreduce']

    def pig_script_path(self):
    """
    The path to the pig script to run
    """
        return self.script_path

if __name__ == '__main__':
    luigi.run(main_task_cls=WordCount)

```

luigi支持对pig打包。执行pig工作的任务必须luigi.contrib.hadoop.PigJobTask子类。pig_script_path()方法用于定义要运行的pig脚本的路径。pig_options()方法用于定义传递给pig脚本的选项。pig_parameters() 方法用于将参数传递给pig脚本

以下命令执行workflow，从 /user/hduser/input.txt 读取数据，存储数据在 /user/hduser/output,参数--script-path被用于定义pig脚本:
```
$ python luigi_pig.py --local-scheduler \
--input-file /user/hduser/input/input.txt \
--output-file /user/hduser/output \
--script-path pig/wordcount.pig
``` 
