# Getting the Best Performance with PySpark
source: https://www.slideshare.net/SparkSummit/getting-the-best-performance-with-pyspark

## What is going to be covered:
- What I think I might know about you
- A quick background of how PySpark works
- RDD re-use (caching, persistence levels, and checkpointing)
- Working with key/value data
    - Why group key is evil and what we can do about it
- When Spark SQL can be amazing and wonderful
- A brief introduction to Datasets (new in Spark 1.6.)
- Calling Scala code from Python with Spark

## Spark in Scala, how does PySpark work/
- Py4J + pickling + magin (This can be slow sometimes)
- RDDs are generally RDDs of pickled objects
- Spark SQL (and DataFrames) avoid some of this

So how does that impact PySpark?
- Data from Spark worker serialized and piped to Python worker
    - Multiple iterator-to-iterator transformation are still pipelined
- Double serialization cost makes everything more expensive
- Python worker startup takes a bit of extra time
- Python memory isn't controlled by the JVM - easy to go over container limits if deploying on YARN or similar
- Error messages make ~0 sense
- etc.

```python
words = rdd.flatMap(lambda x: x.split(" "))
wordPairs = words.map(lambda w: (w, 1))
grouped = wordParis.groupByKey()
grouped.mapValues(lambda counts: sum(counts))
warnings = rdd.filter(lambda x: x.lower.find("warning") != -1).count()
```

RDD re-use sadly not magic
- If we know we are going to re-use the RDD what should we do?
    - If it fits nicely in memory caching in memory
    - Persisting at another level
        - MEMORY, MEMORY_AND_DISK
    - Checkpointing
- Noisey clusters
    - \_2 & checkpointing can help
- Persist first for checkpointing

## What is key skew and why do we care?
- Keys aren't evenly distributed
    - Sales by zip code, or records by city, etc
- groupByKey will explode (but it's pretty easy to break)
- We can have really unbalanced partitions
    - If we have enough key skew sortByKey could even fail
    - Stragglers (uneven sharding can make some tasks take much longer).
    
**groupByKey - just how eveil is it?**
- Pretty evil
- Groups all of the records with the same key into a single record
    - Even if we immediately reduce it (e.g. sum it or similar)
    - This can be too big to fit in memory, then our job fails
- Unless we are in SQL then happy pandas

**So what did we do instead?**
- reduceByKey - works when the types are the same (e.g. in our summing version)
- aggregateByKey - doesn't require the types to be the same (e.g. computing stats model or similar)

Allow Spark to pipeline the reduction & skip making the list. We also got a map-side reduction (note the difference in shuffled read)

**Can just the shuffle cause problems?**
- Sorting by key can put all of the records in the same partition
- We can run into partition size limits (aorund 2GB)
- Or just get bad performance
- So we can handle data like the above we can add some "junk" to our key.

**Our saviour from serialization: DataFrames
- For the most part keeps data in the JVM
    - Notable exception is UDFs written in Python
- Takes our python calls and turns it into a query plan
- If we need more than the native operations in Spark's DataFrames
- be wary of Distributed Systems bringing claims of usability...

**So what are Spark DataFrames?**
- More than SQL tables
- Not Pandas or R DataFrames
- Semi-structured (have schema information)
- Tabular
- Work on expression instead lambdas
    - e.g. df.filter(df.col("happy") == true) instead of rdd.filter(lambda x: x.happy==true))

**Where can Spark SQL benefit perf?
- Structured or semi-structured data
- Ok with having less\* complex operations available to us
- We may only need to operate on a subset of the data
    - The fastest data to process isn't even read
- Remember that non-magic cat? Its got some magic\*\* now
    - In part from peeking inside of boxes.
- non-JVM (aka Python & R) users: saved from double serialization cost!

\*\*Magic may cause stack overflow. Not valid in all states. Consult local magic bureau before attempting magic

**Why is Spark SQL good for those things?**
- Space efficient columnar cached representation
- Able to push down operations to the data store.
- Optimizer is able to look inside of our operations
    - Regular spark can't see inside our operations to spot the difference between (min(_, _)) and (append(_, _))

**Mixing Python & JVM code FTW:**
- DataFrames are an example of pushing our processing to the JVM
- Python UDFs & maps lose this benefit
- But we can write Scala UDFs and call them from Python
    - Py4J error messages can be difficult to understand
- Trickier with RDDs since stores pickled objects

**Exposing functions to be callable from Python:**
```scala
// functions we want to be callable from Python
object functions {
    def kurtosis(e: Column): Column = new Column(Kurtosis(EvilSqlTools.getExpr(e)))
    
    def registerUDFs(sqlCtx: SQLContext): Unit = {
        sqlCtx.udf.register("rowKurtosis", helpers.rowKurtosis _)
    }
}
```

**Calling the functions with py4j\*:**
- The SparkContext has a reference to the jvm (\_jvm)
- Many Python objects which are wrappers of JVM objects have \_j
