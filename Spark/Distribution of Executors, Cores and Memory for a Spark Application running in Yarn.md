# Distribution of Executors, Cores and Memory for a Spark Application running in Yarn

source: https://spoddutur.github.io/spark-notes/distribution_of_executors_cores_and_memory_for_spark_application.html

```scala
spark-submit --class <CLASS_NAME> --num-executors ? --executor-cores ? --executor-memory ? ....
```
Ever wondered how to configure --num-executors, --executor-memory and --executor-cores spark config params for your cluster

## Theory
Following list captures some recommendations to keep in mind while configuring them:
- **Hadoop/Yarn/OS Deamons**: When we run spark application using a cluster manager like YARN, there'll be several daemons that'll run in the background like NameNode, Secondary NameNode, DataNode, JobTracker and TaskTracker. So, while specifying num-executors, we'll need to make sure that we leave aside enough cores (~1 per node) for these daemons to run smoothly.
- **YARN ApplicationMaster (AM)**: ApplicationMaster is responsible for negoiting resources from the ResourceManager and working with the NodeManager to execute and monitor the containers and their resource consumption. If we are runing spark on yarn, then we need to budget in the resources that AM would need (~1024MB and 1 Executor).
- **HDFS Throughput**: HDFS client has trouble with tons of concurrent threads. It was observed that HDFS achieves the full write throughput with ~5 tasks per executor. So it's good to keep the number of cores per executor below that number.
- **MemoryOverhead**
  - Full memory requested to YARN per executor = spark-executor-memory + spark.yarn.executor.memoryOverhead.
  - spark.yarn.executor.memoryOverhead = Max(384MB, 7% if spark.executor-memory)

So, if we request 20GB per executor, AM will actually get 20GB + memoryOverhead = 20 + 7% of 20GB = ~23GB memory for us.

Note:
- Running executors with too much memory often results in excessive garbage collection delays.
- Running tiny executors (with a single core and just enough memory needed to run a single task, for example) throws away the benefits that come from running multiple tasks in a single JVM.


