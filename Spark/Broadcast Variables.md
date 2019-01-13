# Broadcast Variables

**Broadcast** is part of Spark that is responsible for broadcasting information across the nodes in a cluster.

When you broadcast a value, it is copied to executors only once (while it is copied multiple times for tasks otherwise). It means that broadcast can help to get your Spark application faster if you have a large value to use in tasks or there are more tasks than executors.

Broadcast Variables allow the programmer to keep a read-only varibale cached on each machine rather than shipping a copy of it with tasks.

Explicitly creating broadcast variables is only useful when tasks across multiple stages need the same data or when caching the data in desrialized form is important.

To use a broadcast value in a Spark transformation you have to create it first using **SparkContext.broadcast** and then use **value** method to access the shared value. 

The Broadcast feature in Spark uses SparkContext to create broadcast values and BroadcastManager and ContextCleaner to manage their lifecylce.

|Tip|Not only can Spark developers use broadcast variables for efficient data distribution, but Spark itself uses them quite often. A very notable use case is when Spark distributes tasks to executors for their execution. That *does* change my perspective on the role of broadcast variables in Spark.|
|---|---|

**Broadcast API**

|Method Name|Description|
|---|---|
|id|The unique identifier|
|value|The value|
|unpersist|Asynchronously deletes cached copies of this broadcast on the executors|
|destroy|Destroys all data and metadata related to this broadcast variables|
|toString|The string representation|

