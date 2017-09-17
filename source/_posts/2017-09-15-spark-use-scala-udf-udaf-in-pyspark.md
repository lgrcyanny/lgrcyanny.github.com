title: How to Use Scala UDF and UDAF in PySpark
date: 2017-09-15 11:16:33
tags: spark
---

Spark DataFrame API provides efficient and easy-to-use operations to do analysis on distributed collection of data. Many users love the Pyspark API, which is more usable than scala API. Sometimes when we use UDF in pyspark, the performance will be a problem. How about implementing these UDF in scala, and call them in pyspark? BTW, in spark 2.0, UDAF can only be defined in scala, and how to use it in pyspark? Let's have a try~
<!--more-->

## Use Scala UDF in PySpark

**1. define scala udf**

Suppose we want to calculate string length, lets define it in scala UDF.

```scala
import org.apache.spark.sql.expressions.UserDefinedFunction
import org.apache.spark.sql.functions._
 
object StringLength {
  def getStringLength(s: String) = s.length
  def getFun(): UserDefinedFunction = udf(getStringLength _)
}
```

**2. use udf in python**

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
from pyspark.sql import SparkSession
from pyspark.sql.column import Column
from pyspark.sql.column import _to_java_column
from pyspark.sql.column import _to_seq
from pyspark.sql.functions import col
 
spark = SparkSession.builder.appName("scala_udf_test").getOrCreate()
sc = spark.sparkContext
 
def string_length(col):
    _string_length = sc._jvm.com.learning.StringLength.getFun()
    return Column(_string_length.apply(_to_seq(sc, [col], _to_java_column)))
 
def process():
    rows = [
        ("k1", "aaa"),
        ("k2", "dd"),
        ("k3", "cc"),
        ("k4", "eee"),
    ]
    df = spark.createDataFrame(rows, ['key', 'value'])
    df.show(50)
    df.select(col("key"), string_length(col("value"))).show()
 
if __name__ == "__main__":
    process()
```

**3. submit the app**

compile the scala code and submit python files with --jars
 
```scala
./bin/spark-submit --jars testing/learning-1.0.0-SNAPSHOT.jar udf_test.py
```

the output would be:

key|value
---|---
k1|3
k2|2
k3|2
k4|3

**4. performance analysis**

let's explain the scala UDF in Python
![scala udf physical plan](http://wx1.sinaimg.cn/mw690/761b7938ly1fjmeol1jg8j20s405odim.jpg)
the Project Plan is Scala UDF

and if we implement Python UDF as follows:

```python
py_slen = udf(lambda s: len(s), IntegerType())
df_with_python_udf = (df.select(col("key"), py_slen("value").alias("slen")).orderBy(col("slen").desc()))
```

the Python plan is:
![python udf physical plan](http://wx4.sinaimg.cn/mw690/761b7938ly1fjmeofgzbmj210k06igot.jpg)
the UDF plan is different, which is BatchEvalPython.
It can prove that when use scala UDF in python, the evaluation is in JVM and data will not exchange with Python worker. And the performance should be improved.

I evaluated the performance in local environment with 4cores and 2GB memory, and generated 10million rows for each test, the result is as follows:
![scala vs python string len udf](http://wx2.sinaimg.cn/mw690/761b7938ly1fjmer43xfnj20kw0ckt8z.jpg)
**Scala UDF is 1.89 times Python UDF**

**And then I implemented another UDF in Scala and Python with regex string parsing**, the performance is
![scala vs python string regex parsing](http://wx3.sinaimg.cn/mw690/761b7938ly1fjmeopknh4j20a0061748.jpg)

**Scala udf is 2.23 times Python REGEX String Parsing UDF**

the Scala UDF is defined as follows:

```scala
import  org.apache.spark.sql.functions._

/**
  * Created by lgrcyanny on 17/9/13.
  */
object StringParse {
  val STRING_PATTERN = """(a.*b)""".r

  def parseString(str: String): String = {
    val matched = STRING_PATTERN.findFirstMatchIn(str)
    if (matched.isEmpty) {
      ""
    } else {
      matched.get.group(1)
    }
  }

  def getFun() = udf(parseString _ )
}
```

Python string parse UDF  vs Scala UDF:

```python
import time
import re
from pyspark.sql import SparkSession
from pyspark.sql.column import Column
from pyspark.sql.column import _to_java_column
from pyspark.sql.column import _to_seq
from pyspark.sql.functions import col
from pyspark.sql.functions import udf
from pyspark.sql.functions import length
from pyspark.sql.types import StringType
from pyspark.sql.types import IntegerType

def random_word(length):
    """get random word for generate rows"""
    letters = string.ascii_lowercase
    return ''.join([random.choice(letters) for i in range(length)])

def generate_rows(n):
    """generate rows in key value pair"""
    # generate rows
    letters = "abcdefghijklmnopqrstuvwxyz"
    rows = []
    for i in range(n):
        id = random.randint(0, 100)
        slen = random.randint(0, 20)
        word = random_word(slen)
        rows.append((id, letters))
    return rows

def string_parse(col):
    """scala udf parse string"""
    _string_parse = sc._jvm.com.learning.StringParse.getFun()
    return Column(_string_parse.apply(_to_seq(sc, [col], _to_java_column)))
    
def test_regex_udf(n=1000):
    """test udf with regex parse"""
    rows = generate_rows(n)
    df = spark.createDataFrame(rows, ['key', 'value'])
    df.show(20)
    pattern = re.compile(r"(a.*b)")
    def parse_string(str):
        """parse string with python regex"""
        matched = re.search(pattern, str)
        if matched:
            return matched.group(1)
        else:
            return ""
    py_parse_str = udf(parse_string, StringType())
    start_time = time.time()
    df_with_python_udf = (df.select(col("key"), py_parse_str(col("value")).alias("parsed_value"))
                          .filter(length(col("parsed_value")) > 0))
    df_with_python_udf.explain(True)
    df_with_python_udf.show()
    print("matched rows: {}".format(df_with_python_udf.count()))
    print("duration for python regex parse: {}s".format(time.time() - start_time))

    start_time = time.time()
    df_with_scala_udf = (df.select(col("key"), string_parse(col("value")).alias("parsed_value"))
                          .filter(length(col("parsed_value")) > 0))
    df_with_python_udf.explain(True)
    df_with_scala_udf.show()
    print("matched rows: {}".format(df_with_scala_udf.count()))
    print("duration for scala regex parse: {}s".format(time.time() - start_time))

```

**5. Conclusion**

Databricks used to give a performance for Python vs Scala DataFrame and RDD API:
![databricks performance](http://wx2.sinaimg.cn/mw690/761b7938ly1fjmeo7vk38j210c0fy48j.jpg)

the blog is [here](https://databricks.com/blog/2015/02/17/introducing-dataframes-in-spark-for-large-scale-data-science.html).
The performance is a running group-aggregation on 10 million integer pairs on a single machince. The Scala DF is almost 5 times Python lambda function in RDD Python.

Even though, the Scala UDF is not 5 times Python UDF, about 2 times in my test, using scala UDF can improve performance indeed.


## Use Scala UDAF in PySpark

UDAF now only supports defined in Scala and Java(spark 2.0)

**1. define scala UDAF**

when define UDAF, it must extend class `UserDefinedAggregateFunction`

```scala
import org.apache.spark.sql.Row
import org.apache.spark.sql.expressions.{MutableAggregationBuffer, UserDefinedAggregateFunction}
import org.apache.spark.sql.types.{ArrayType, DataType, StringType, StructType}
 
import scala.collection.mutable.ArrayBuffer
 
object GroupConcat extends UserDefinedAggregateFunction {
  override def inputSchema: StructType = new StructType().add("s", StringType)
  override def bufferSchema: StructType = new StructType().add("buff", ArrayType(StringType))
  override def dataType: DataType = StringType
  override def deterministic: Boolean = true
  override def initialize(buffer: MutableAggregationBuffer): Unit = {
    buffer.update(0, ArrayBuffer.empty[String])
  }
 
  override def update(buffer: MutableAggregationBuffer, input: Row): Unit = {
    if (!input.isNullAt(0)) {
      buffer.update(0, buffer.getSeq[String](0) :+ input.getString(0))
    }
  }
 
  override def merge(buffer1: MutableAggregationBuffer, buffer2: Row): Unit = {
    buffer1.update(0, buffer1.getSeq[String](0) ++ buffer2.getSeq[String](0))
  }
 
  override def evaluate(buffer: Row): Any = {
    buffer.getSeq[String](0).mkString(",")
  }
}
```

**2. use UDAF in python**

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
from pyspark.sql import SparkSession
from pyspark.sql.column import Column
from pyspark.sql.column import _to_java_column
from pyspark.sql.column import _to_seq
 
spark = SparkSession.builder.appName("scala_udf_test").getOrCreate()
sc = spark.sparkContext
 
def group_concat(col):
    _groupConcat = sc._jvm.com.learning.GroupConcat.apply
    return Column(_groupConcat(_to_seq(sc, [col], _to_java_column)))
 
def process():
    rows = [
        ("k1", "a"),
        ("k1", "b"),
        ("k1", "c"),
        ("k2", "d"),
        ("k3", "e"),
        ("k3", "f"),
    ]
    df = spark.createDataFrame(rows, ['key', 'value'])
    df.show(50)
    df.groupBy("key").agg(group_concat("value").alias("concat")).show()
 
if __name__ == "__main__":
    process()
```

**3. submit the app**

```
./bin/spark-submit --jars testing/learning-1.0.0-SNAPSHOT.jar udf_test.py
```

the output would be:

key|cancat
---|---
k1|a,b,c
k2|d
k3|e,f

**4. references**

+ [spark-sql-replacement-for-mysql-group-concat-aggregate-function](https://stackoverflow.com/questions/31640729/spark-sql-replacement-for-mysql-group-concat-aggregate-function)
+ [spark-how-to-map-python-with-scala-or-java-user-defined-functions](https://stackoverflow.com/questions/33233737/spark-how-to-map-python-with-scala-or-java-user-defined-functions)



