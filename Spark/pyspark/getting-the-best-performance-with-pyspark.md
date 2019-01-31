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
