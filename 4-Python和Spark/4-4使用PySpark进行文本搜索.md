<h2>使用PySpark进行文本搜索</h2>

文本搜索程序搜索与给定字符串匹配的影片标题。电影数据来自groupLens数据集;应用程序希望此存储在 HDFS下的/user/hduser/input/movies。

* Example 4-3. python/Spark/text_search.py
```
from pyspark import SparkContext
import re
import sys
def main():
    # Insure a search term was supplied at the command line
    if len(sys.argv) != 2:
        sys.stderr.write('Usage: {} <search_term>'.format(sys.argv[0]))
        sys.exit()

    # Create the SparkContext
    sc = SparkContext(appName='SparkWordCount')

    # Broadcast the requested term
    requested_movie = sc.broadcast(sys.argv[1])

    # Load the input file
    source_file = sc.textFile('/user/hduser/input/movies')

    # Get the movie title from the second fields
    titles = source_file.map(lambda line: line.split('|')[1])

    # Create a map of the normalized title to the raw title
    normalized_title = titles.map(lambda title: (re.sub(r'\s*\(\d{4}\)','', title).lower(), title))

    # Find all movies matching the requested_movie
    matches = normalized_title.filter(lambda x: requested_movie.value in x[0])

    # Collect all the matching titles
    matching_titles = matches.map(lambda x: x[1]).distinct().collect()

    # Display the result
    print '{} Matching titles found:'.format(len(matching_titles))
    for title in matching_titles:
        print title

    sc.stop()

if __name__ == '__main__':
    main()
```

可以通过向sparksubmit脚本传递程序的名称、text_search 和要搜索的术语来执行Spark应用程序。可以看到应用程序的示例运行:

```
$ spark-submit text_search.py gold
...
6 Matching titles found:
GoldenEye (1995)
On Golden Pond (1981)
Ulee's Gold (1997)
City Slickers II: The Legend of Curly's Gold (1994)
Golden Earrings (1947)
Gold Diggers: The Secret of Bear Mountain (1995)
...
```

由于计算转换可能是一项代价高昂的操作,Spark可以将normalized_titles的结果缓存到内存中, 以加快将来的搜索速度。从上面的示例中, 将normalized_titles加载到内存中, 使用cache()方法。