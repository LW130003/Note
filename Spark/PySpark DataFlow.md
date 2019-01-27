source: https://stackoverflow.com/questions/31684842/calling-java-scala-function-from-a-task

Communication using default Py4J gateway is simply not possible. To understand why we have to take a look at the following diagram from the PySpark Internals document [1]:

>![](https://i.stack.imgur.com/sfcDU.jpg)

Since Py4J gateway runs on the driver it is not accessible to Python interpreters which communicate with JVM workers through sockets.

Theoretically, it could be possible to create a separate Py4J gateway for each worker but in practice it is unlikely to be useful. Ignoring issues like reliability Py4J is simply not designed to perform data intensive tasks.

Are there any workarounds?
1. Using **Spark SQL Data Source API** to wrap JVM code. https://databricks.com/blog/2015/01/09/spark-sql-data-sources-api-unified-data-access-for-the-spark-platform.html
- **Pros**: Supported, high level, doesn't require access to internal PySpark API
- **Cons**: Relatively verbose and not very well documented, limited mostly to the input data.
