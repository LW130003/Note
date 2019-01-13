# SparkContext - Entry Point to Spark Core

SparkContext (aka **Spark Context**) is the heart of a Spark Application

|Note| You could also assume that a SparkContext Instance is a Spark Application|
|---|---|

Spark Context sets up internal services and establishes a connection to a Spark execution environment.

Once a *SparkContext* is created you can use it to create RDDs, accumulators and broadcast variables, access Spark services and run jobs (until *SparkContext* is stopped)

## Creating SparkContext Instance
You can create a SparkContext instance with or without creating a SparkConf object first.
|Note|You may want to read Inside Creating SparkContext to learn what happens behind the scenes when SparkContext is created|
|---|---|

### Getting Existing or Creating New SparkContext - *getOrCreate* Methods

```scala
getOrCreate(): SparkContext
getOrCreate(conf: SparkConf): SparkContext
```
getOrCreate methods allow you to get the existing SparkContext or create a new one.

```scala
import org.apache.spark.SparkContext
val sc = SparkContext.getOrCreate()

// Using an explicit SparkConf object
import org.apache.spark.SparkConf
val conf = new SparkConf()
  .setMaster("local[*]")
  .setAppName("SparkMe App")
val sc = SparkContext.getOrCreate(conf)
```

The no-param getOrCreate method requires that the two mandatory Spark settings - master and application name - are specified using spark-submit.

### Constructors
```scala
SparkContext()
SparkContext(conf: Configuration)
SparkContext(master: String, appName: String, conf: SparkConf)
SparkContext(
  master: String,
  appName: String,
  sparkHome: String = null,
  jars: Seq[String] = Nil,
  environment: Map[String, String] = Map())
```
You can create a SparkContext instance using the four constructors
```scala
import org.apache.spark.SparkConf
val conf = new SparkConf()
  .setMaster("local[*]")
  .setAppName("SparkMe App")

import org.apache.spark.SparkContext
val sc = new SparkContext(conf)
```
When a SparkContext starts up you should see the following INFO in the logs (amongst the other messages that come from the Spark services):
```scala
INFO SparkContext: Running Spark version 2.0.0-SNAPSHOT
```
|Note|Only one SparkContext may be running in a single JVM (check out SPARK-2243 Support multiple SparkContexts in the same JVM). Sharing access to a SparkContext in the JVM is the solution to share data within Spark (without relying on other means of data sharing using external data stores).
|---|---|

## Getting Current SparkConf - *getConf* Method
```scala
getConf: SparkConf
```
getConf returns the current SparkConf
|Note|Changing the **SparkConf** object does not change the current configuration (as the method returns a copy).|
|---|---|

## SparkContext and RDDs
You can use Spark Context to create RDDs. When an RDD is created, it belongs to and is completely owned by the Spark Context it originated from. RDDs can't by design be shared between SparkContexts.

### Creating RDD - *parallelize* Method
*SparkContext* allows you to create many different RDDs from input sources like:
- Scala's collections, i.e. *sc.parallelize(0 to 100)*
- local or remote filesystems, i.e. *sc.textFile("README.md")*
- Any Hadoop *InputSource* using *sc.newAPIHadoopFile*

## Stopping SparkContext - *stop* Method
```scala
stop(): Unit
```
*stop* stops the SparkContext.

Internally, *stop* enables *stopped* internal flag. If already stopped, you should see the following INFO message in the logs:
```
INFO SparkContext: SparkContext already stopped
```
