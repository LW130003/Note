# How to use SparkSession in Apache Spark 2.0
## A Unified entry point for manipulating data with Spark

https://databricks.com/blog/2016/08/15/how-to-use-sparksession-in-apache-spark-2-0.html

Generally, a session is an interaction between two or more entities. In computer paralance, its usage is prominent in the realm of networked computers on the internet. First with TCP session, then with login session, followed by HTTp and user session, so no surprise that we now have *SparkSession*, introduced in Apache Spark 2.0

Beyond a time-bounded interaction, *SparkSession* provides a single point of entry to interact with underlying Spark functionality and allows programming Spark with DataFrame and Dataset APIs. Most importantly, it curbs the number of concepts and constructs a developer has to juggle while interacting with Spark.

## Exploring SparkSession's Unified Functionality
First, we will examine a Spark application, SparkSessionZipsExample, that reads zip codes from a JSON file and do some analytics using DataFrames APIs, followed by issuing Spark SQL queries, without accessing SparkContext, SQLContext or HiveContext.

### Creating a SparkSession
In previous versions of Spark, you had to create a SparkConf and SparkContext to interact with Spark, as shown here:
```scala
//set up the spark configuration and create contexts
val sparkConf = new SparkConf().setAppName("SparkSessionZipsExample").setMaster("local")
// your handle to SparkContext to access other context like SQLContext
val sc = new SparkContext(sparkConf).set("spark.some.config.option", "some-value")
val sqlContext = new org.apache.spark.sql.SQLContext(sc)
```

Whereas in Spark 2.0 the same effects can be achieved through SparkSession, without explicitly creating SparkConf, SparkContext or SQLContext, as they're encapsulated within the SparkSession. Using a builder design pattern, it instantiates a SparkSession object if one does not already exist, along with its associated underlying contexts.

```scala
// Create a SparkSession. No need to create SparkContext
// You automatically get it as part of the SparkSession
val warehouseLocation = "file:${system:user.dir}/spark-warehouse"
val spark = SparkSession
   .builder()
   .appName("SparkSessionZipsExample")
   .config("spark.sql.warehouse.dir", warehouseLocation)
   .enableHiveSupport()
   .getOrCreate()
```
At this point you can use the *spark* variables as your instance object to access its public methods and instances for the duration of your Spark job.

### Configuring Spark's Runtime Properties
Once the SparkSession is instantiated, you can configure Spark's runtime config properties. For example, in this code snippet, we can alter the existing runtime config options. Since *configMap* is a collection, you can use all of Scala's iterable methods to access the data.

```scala
// set new runtime options
spark.conf.set("spark.sql.shuffle.partitions", 6)
spark.conf.set("spark.executor.memory", "2g")
// get all settings
val configMap: Map[String, String] = spark.conf.getAll()
```

### Accessing Catalog Metadata
Often, you may want to access and peruse the underlying catalog metadata. SparkSession exposes "catalog" as a public instance that contains methods that work with the metastore (i.e. data catalog). Since these methods return a Dataset, you can use Dataset API to acess or view data. In this snippet, we access table names and list of databases.

```scala
// fetch metadata from the catalog
spark.catalog.listDatabases.show(false)
spark.catalog.listTables.show(false)
```
![](https://databricks.com/wp-content/uploads/2016/08/Screen-Shot-2016-08-12-at-5.38.30-PM-1024x464.png)

### Creating Datasets and DataFrames


