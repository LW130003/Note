# How to convert an RDD of Rows to DataFrame
https://stackoverflow.com/questions/37011267/how-to-convert-an-rddrow-back-to-dataframe

To create a DataFrame from an RDD of Rows, usually you have two main options:
1. You can use **toDF()** which can be imported by **import sqlContext.implicts._**. However, this approach only works for the following types of RDDs:
    - RDD[Int]
    - RDD[Long]
    - RDD[String]
    - RDD[T <: scala.Product]
(source: http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.SQLContext$implicits$ Scaladoc of the SQLContext.implicts.object)

The last signature actually means that it can work for an RDD of typles or an RDD of case classes (because tuples and case classes are subclasses of scala.Product).

So, to use this approach for an RDD[Row], you have to map it to an RDD of tuples or an RDD of case classes (because tuples and case classes are subclasses of scala.Product).

So, to use this approach for an RDD[Row], you have to map it to an RDD[T <: scala.Product]. This can be done by mapping each row to a custom case class or to a tuple, as in the following code snippets:

```scala
val df = rdd.map({
    case Row(val1: String, ..., valN: Long) => (val1, ..., valN)
}).toDF("col1_name", ..., "colN_name")
```

or

```scala
case class MyClass(val1: String, ..., valN: Long = 0L)
val df = rdd.map({
    case Row(val1: String, ..., valN: Long) => MyClass(val1, ..., valN)
}).toDF("col1_name", ..., "colN_name")
```
The main drawback of this approach (in my opinion) is that you have to explicitly set the schema of the resulting DataFrame in the map function, column by column. May be this can be done programatically if you don't know the schema in advance, but things can get a little messy there. So, alternatively, there is another option.

2. You can use **createDataFrame(rowRDD: RDD[Row], schema: StructType)**, which is available in the SQLContext object. Example:
```scala
val df = oldDF.sqlContext.createDataFrame(rdd, oldDF.schema)
```

Note that there is no need to explicitly set any schema column. We reuse the old DF's schema, which is of StructType class and can be easily extended. However, this approach sometimes is not possible, and in some cases can be less efficient than the first one.

# How to convert rdd object to dataframe in Spark
souce: https://stackoverflow.com/questions/29383578/how-to-convert-rdd-object-to-dataframe-in-spark

This code works perfectly from **Spark 2.x** with **Scala 2.11**

Import necessary classes
```scala
import org.apache.spark.sql.{Row, SparkSession}
import org.apache.spark.sql.types.{DoubleType, StringType, StructField, StructType}
```

Create SparkSession Object, Here it's spark
```scala
val spark: SparkSession = SparkSession.builder.master("local").getOrCreate
val sc = spark.sparkContext // Just used to create test RDDs
```

Let's create an **RDD** which will be converted into **DataFrame**
```scala
vak rdd = sc.parallelize(
    Seq(
        ("first", Array(2.0, 1.0, 2.1, 5.4)),
        ("test", Array(1.5, 0.5, 0.9, 3.7)),
        ("choose", Array(9.0, 2.9, 9.1, 2.5))
    )
)
```

## Method 1
Using **SparkSession.createDataFrame(RDD obj).
```scala
val dfWithutSchema = spark.createDataFrame(rdd)
df.withoutSchema.show()
+------+--------------------+
|    _1|                  _2|
+------+--------------------+
| first|[2.0, 1.0, 2.1, 5.4]|
|  test|[1.5, 0.5, 0.9, 3.7]|
|choose|[8.0, 2.9, 9.1, 2.5]|
+------+--------------------+
```

## Method 2
Using **SparkSession.createDataFrame(RDD obj)** and specifying column names.
```scala
val dfWithSchema = spark.createDataFrame(rdd).toDF("id", "vals")
dfWithSchema.show()
+------+--------------------+
|    id|                vals|
+------+--------------------+
| first|[2.0, 1.0, 2.1, 5.4]|
|  test|[1.5, 0.5, 0.9, 3.7]|
|choose|[8.0, 2.9, 9.1, 2.5]|
+------+--------------------+
```

## Method 3 (Actual answer to question)
This way requires the input **rdd** should be of type **RDD[Row]**.
```scala
val rowsRdd: RDD[Row] = sc.parallelize(
    Seq(
        Row("first", 2.0, 7.0),
        Row("second", 3.5, 2.5),
        Row("third", 7.0, 5.9)
    )
)
```

Create the schema
```scala
val schema = new StructType()
    .add(StructField("id", StringType, true))
    .add(StructField("val1", DoubleType, true))
    .add(StructFiedl("val2", DoubleType, true))
```

Now apply both **rowsRdd** and **schema** to **createDataFrame()
```scala
val df = spark.createDataFrame(rowsRdd, schema)
df.show()
+------+----+----+
|    id|val1|val2|
+------+----+----+
| first| 2.0| 7.0|
|second| 3.5| 2.5|
| third| 7.0| 5.9|
+------+----+----+
```


