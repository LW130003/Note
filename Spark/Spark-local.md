# Spark Local (pseudo-cluster)

You can run Spark in **local mode**. In this non-distributed single-JVM deployment mode, Spark spawns the execution components - driver, executor, LocalSchedulerBackend, and master - in the same single JVM. The default parallelism is the number of threads as specified in the master URL.

The local mode is very convenient for testing, debugging or demonstrating purposes as it requires no earlier setup to launch spark applications.

The mode of operations is also called Spark in-process (or less commonly) **a local version of Spark**

## Check if Spark runs in local mode
```scala
scala> sc.isLocal
```
**SparkContext.isLocal** returns **true** when Spark runs in local mode.

## Spark Shell defaults to local mode with **local[\*]** as the master URL
```scala
scala> sc.master
```

Tasks are not re-executed on failure in local mode (unless local-with retries master URL is used).

The task scheduler in local mode work with LocalSchedulerBackend task scheduler backend.

## Master URL
You can run Spark in local mode using **local**, **local[n]**, or the most general **local[*]** for the master URL.

The URL says how many threads can be used in total:
- **local** - uses 1 thread only
- **local[n]** - uses n threads
- **local[\*]** uses as many threads as the number of processors available to the Java Virtual Machine.
- **local[N, maxFailures]** (called **local-with-retries**) with N being \* or the number of threads to use (as explained above) and **maxFailures** being the value of spark.task.maxFailures.




