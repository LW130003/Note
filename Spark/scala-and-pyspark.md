source: https://stackoverflow.com/questions/43812365/spark-1-6-how-do-convert-an-rdd-generated-from-a-scala-jar-to-a-pyspark-rdd

Just because it is a valid PySpark RDD it doesn't mean that the content can be understood by Python. What you pass is an RDD of java objects. For internal covnversions Spark use Pyrolite to re-serialize objects between Python and JVM.

This is an internal API, but you can:

```python
from pyspark.ml.common import _java2py
rdd = _java2py(sc, sc._jvm.com.clickfox.combinations.lab.PySpark.getTestRDD(sc._jsc.sc()))
```

Note that this approach is fairly limited and supports only basic type conversions.

You can also use replace RDD with DataFrame:
```scala
object PySpark{
    def getTestDataFrame(sqlContext: SQLContext): DataFrame = {
        sqlContext.range(1,10)
    }
}
```

```python
from pyspark.sql.dataframe import DataFrame
DataFrame(sc._jvm.com.clickfox.combinations.lab.PySpark.getTestDataFrame(sqlContext._jsqlContext), sqlContext)
```
