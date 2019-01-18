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

# Hands On
Let's consider 10 node cluster with following config and analyze different possibilities of executors-core-memory distirbution:
```
**Cluster Config:**
10 Nodes
16 Cores per Node
64GB RAM per Node
```

## First approach: Tiny Executors \[One Executor per core\]
Tiny executors essentially means one executor per core.
```
- `--num-executors` = `In this approach, we'll assign one executor per core`
                    = `total-cores-in-cluster`
                   = `num-cores-per-node * total-nodes-in-cluster` 
                   = 16 x 10 = 160
- `--executor-cores` = 1 (one executor per core)
- `--executor-memory` = `amount of memory per executor`
                     = `mem-per-node/num-executors-per-node`
                     = 64GB/16 = 4GB
```
**Analysis**: With only one executor per core, as we discussed above, we’ll not be able to take advantage of running multiple tasks in the same JVM. Also, shared/cached variables like broadcast variables and accumulators will be replicated in each core of the nodes which is 16 times. Also, we are not leaving enough memory overhead for Hadoop/Yarn daemon processes and we are not counting in ApplicationManager. NOT GOOD!

## Second Approach: Fat Executors (One Executor per Node):
Fat executors essentially means one executor per node. Following table depicts the values of our spark-config params with this approach:
```
- `--num-executors` = `In this approach, we'll assign one executor per node`
                    = `total-nodes-in-cluster`
                   = 10
- `--executor-cores` = `one executor per node means all the cores of the node are assigned to one executor`
                     = `total-cores-in-a-node`
                     = 16
- `--executor-memory` = `amount of memory per executor`
                     = `mem-per-node/num-executors-per-node`
                     = 64GB/1 = 64GB
```
**Analysis**: With all 16 cores per executor, apart from ApplicationManager and Daemon Processes are not counted for, HDFS Throughput will hurt and it'll result in excessive garbace results. Also, **NOT GOOD!**

## Third Approach: Balance Between Fat (vs) Tiny
**According to the recommendations which we discussed above**:
- 
