# **Sparks Fundamental I: **Module 2: Resillient Distributed Dataset and DataFrames**
## **1. Objectives**:
- Describe Spark's primary data abstraction
- Understand how to create parallelized collections and external datasets
- Work with Resilient Distributed Dataset (RDD) operations
- Utilize shared variables and key-value pairs

## **2. Resilient Distributed Dataset (RDD)**
RDD is:
- Spark's primary abstraction. RDD is a fault tolerant collection of elements that can be parallelized. In other words, they can be made to be operated on in parallel.
- Is immutable. These are the fundamental primary units of data in Sparks.

Three methods for creating RDD:
- Parallelizing an existing collection
- Referencing a dataset
- Transformation from an existing RDD

Dataset from any storage supported by Hadoop:
- HDFS
- Cassandra
- HBase
- Amazon S3
- etc.

Types of files supported:
- Text files
- Sequence Files
- Hadoop InputFormat
- 
When RDDs are created, a direct acyclic graph (DAG) is created. This type of operation is called transformations. Transformations  make updates to that graph, but nothing actually happens until some action is called.

Actions are another type of operations.  The notion here is that the graphs can be replayed on nodes that need to get back to the state it was before it went offline - thus providing fault tolerance.

The elements of RDD can be operated on in parallel across the cluster. Remember, transformations return a pointer to the RDD created and actions return values that comes from the action.

There are 3 methods for creating a RDD. You can parallelize an existing collection. This means that the data already resides within Spark and can now be operated on in parallel. As an example, if you have an array of data, you can create a RDD out of it by calling the parallelized method. This method returns a pointer to the RDD. So this new distributed dataset can now be operated upon in parallel through out the cluster.

The second method to create a RDD, is to reference a dataset. This dataset can come from any storage source supported by Hadoop such as HDFS, Cassandra, HBase. Amazon S3, etc.

The third method to create a RDD is from transforming an existing RDD to create a new RDD. In other words, let's say you have the array of data that you parallelized earlier. Now you want to filter out strings that are shorter than 20 characters. A new RDD is created using the filter method.

## **3. Creating an RDD**
- Launch the Spark shell
	- */bin/spark-shell*
- Create some data
	- val data = 1 to 10000
- Parallelize that data (creating the RDD)
	- val distData = sc.parallelize(data)
- Perform additional transformations or invoke an action on it.
	- distData.filter(...)
- Alternatively, create an RDD from an external dataset
	- val readmeFile = sc.textFile("Readme.md")

First thing is to launch the Spark shell. 
Once the shell is up, create some data with values from 1 to 10,000. Then, create an RDD from that data using the parallelize method from the SparkContext, shown as sc on the slide. This means that the data can now be operated on in parallel.

SparkContext (sc object) that is invoking the parallelized function, in our programming lesson, so for now, just know that when you initialize a shell, the SparkContext, sc, is initialized for you to use.

The parallelize method returns a pointer to the RDD. Remember, transformation operations such as parallelize, only returns a pointer to the RDD. It actually won't create that RDD until some action is invoked on it. With this new RDD, you can perform additional transformation or actions such as the filter transformation.

Another way to create a RDD is from an external dataset. In the example here, we are creating a RDD from a text fill using the textfile method of the SparkContext object.  	

## **4. RDD Operations - Basics**
- Loading a file
	- val lines = sc.textFile('hdfs;//data.txt")
- Applying transformation
	- val linelengths = lines.map(s=>s.length)
- Invoking action
	- val totalLengths = lineLengths.reduce((a,b)=>a+b)
- MapReduce example:
	- val wordCounts = textFile.flatMap(line => line.split (" "))
		- map(word=>(word,1))
		- reduceByKey((a,b)=>a+b)
- wordCounts.collect()

This time we are loading a file from hdfs. Loading the file creates a RDD, wich is only a pointer to the file. The dataset is not loaded into memory yet. Nothing will happen until some action is called. The transformation basically updates the direct acyclic graph (DAT). So the transformation here is saying map each line s, to the length of that line. Then, the action operation is reducing it to get the total length of all the lines. When the action is called, Spark goes through the DAG and applies all the transformation up until that point, followed by the action and then a value is returned back to the caller.

A common example is a MapReduced word count. You first split up the file by words and then map each word into a key value pair with the word as the key, and the value of 1. Then you reduce by the key, which adds up all the vlaue of the same key, effectively, counting the number of occurrences of that key. Finally, you call the collect() function, which is an action, to have it print out all the words and its occurrences.

## **5. Direct Acyclic Graph (DAG)**

A DAG is essentially a graph of the business logic and does not get executed until an action is called -- often called lazy evaluation.

To view the DAG of a RDD after a series of transformation, use the *toDebugString* method

Sample DAG
	res5: String =
		MappedRDD[4] at map at <console>:16 (3 partitions)
			MappedRDD[3] at map at <console>:16 (3 partitions)
				FilteredRDD[2] at filter at <console>:14 (3 partitions)
					MappedRDD[1] at textFile at <console>:12 (3 partitions)
						HadoopRDD[0] at textFile at <console>:12 (3 partitions)

The DAT above, you can see that it starts as textFile and goes through a series of transformation such as map and filter, followed by more map operations.

Remember, that it is this behavior that allows for fault tolerance. If a node goes offline and comes back on, all it has to do is just grab a copy of this from a neighboring node and rebuild the graph back to where it was before it went offline.

## **6 What happens when an action is executed**

//Creating the RDD
val logFile = sc.textFile('hdfs://...")

//Transformations
val errors = logFile.filter(_.startWith("ERROR"))
val messages = errors.map(_.split('\t')).map(r=>r(1))
//Caching
messages.cache()
//Actions
messages.filter(_.contains("mysql")).count()
messages.filter(_.contains("php")).count()

- The first line you load the log from the hadoop file system. 
- The next two lines you filter out the messages within the log errors. Before you invoke some action on it, you tell it to cache the filtered dataset - it doesn't actually cache it yet as nothing has been done up until this point.
- Then you do more filters to get specific error messages relating to mysql and php followed by the count action to find out how many errors were related to each of those filters.

Now let's walkthorugh each steps.
1. When you load in the text file is the data is partitioned into different blocks across the cluster.
2. Driver sends the code to be executed on each block. In the example, it would be the various transformations and actions that will be sent out to the workers. Actually, it is the executor on each workers that is going to be performing the work on each block.
3. Then  the executors read the HDFS block sto prepare the data for the operations in parallel.
4. After series of transformations, you want to cache the results up until that point into memory. A cache is created.
5. After the first actions completes, the results are sent back to the driver. In this case, we're looking for messages that relate to mysql. This is then returned back to the driver.
6. to process the second action, Spark will use the data on the cache -- it doesn't need to go to the HDFS data again. It just reads it from the cache and processes the data from there.
7. Finally the results go back to the driver  and we have completed a full cycle.

## **7. RDD operations - Transformations**
- A subset of the transformations available. Full set can be found on Spark's website.
- Transformations are lazy evaluations. (Nothing is executed until an action is called).
- - Each transformation function basically updates the graph and when an action is called, the graph is executed.
- Returns a pointer to the transformed RDD

1. map(func) : Return a new dataset formed by passing each element of the source through a function func.
2. filter(func): Return a new dataset formed by electing those elements of the source on which *func* returns true
3. flatMap(func): Similar to map, but each input item can be mapped to 0 or more output items. What this means is that the returned pointer of the func method, should return a Sequence of objects, rather than a single item. It would mean that the flatMap would flatten a list of lists for the operations that follows. Basically this would be used for MapReduce operations where you might have a text file and each time a line is read in, you split that line up by spaces to get indiividual keywords. Each of those lines ultimately is flatten so that you can perform the map operation on it to map each keyword to the value of one.
4. join(*otherDataset, [numTasks]*): When called on a dataset of (K, V) pairs, returns a dataset of (K,V) pairs with all pairs of elements for each key. For example, you have a K,V pair and a K,W pair. When you join them together, you will get a K, (V,W) set.
5. reduceByKey(func): When called on a dataset of (K,V), returns a dataset of (K,V) pairs where the values for each key are aggregated using the given reduce function *func*. The reduceByKey function aggregates on each key by using the given reduce function. This is something you would use in a WordCount to sum up the values for each word to count its occurrences.
6. sortByKey([ascending], [numTasks]): When called on a dataset of (K,V) pairs where K implements. Ordered, returns a dataset of (K,V) pairs sorted by keys in ascending or descending order.


## **8. RDD operations - Actions**
Actions returns values. Action is just a subset.

1. collect(): returns all the elements of the dataset as an array of the driver program. This is usually after a filter or another operation that returns a sufficiently small subset of data to make sure that your filter function works correctly.
2. count(): return the number of elements in a dataset and can also be used to check and test transformations.
3. first(): return the first element of the dataset.
4. take(n): return an array with the first n elements of the dataset. Note that this is currently not executed in parallel. The driver computes all the elements.
5. foreach(func): run a function func on each element of the dataset.

## **9. RDD persistence**

The cache function is actually the default of the persist function with the MEMORY_ONLY storage. One of the key capability of spark is its speed through persisting or caching. 
- Each node stores any partitions of the cache and computes it in memory. When a subsequent action is called on the same dataset, or derived dataset, it uses it from memory instead of having to retrieve it again. Future actions in such cases are often 10 times faster.
- Reuses them in other actions on the dataset (or dataset derived from it)
	- Future actions are much faster (often by more than 10x)

- The first time a RDD is persisted, it is kept in memory on the node. Caching is fault tolerant because if it any of the partition is lost, it will automatically be recomputed using the transformations that originally created it. 

- Fault tolerance is the property that enables a system to continue operating properly in the event of the failure of (or one or more faults within) some of its compoenents.

- Two methods to invoke RDD persistence:
	- persist()
	- cache()
The persist() method allows you to specify a different storage level of caching. For example, you can choose to persist the data set on disk, persist it in memory but as serialized objects to save space, etc. 

Again the cache() method is just the default way fo using persistence by storing deserialized objects in memory.

Storage level:
1. MEMORY_ONLY: Store as deserialized java objects in the JVM. If the RDD does not fit in memory, part of it will be cached. The other will be recomputed as needed. This is the default. The cache() method uses this.
2. MEMORY_AND_DISK: Same as MEMORY_ONLY except also store on disk if it doesn't fit in memory. Read from memory and disk when needed/
3. MEMORY_ONLY_SER: Store as serialized java objects (one by one array per partition). Space efficient, but will require the RDD to deserialized before it can be read, so it takes up more CPU workload.
4. MEMORY_AND_DISK_SER: similar to MEMORY_AND_DISK but stored as serialized objects.
5. DISK_ONLY: Store only on disk
6. MEMORY_ONLY_2, MEMORY_AND_DISK_2, etc.: Same as above, but replicate eac partion on two cluster nodes.
7.  OFF_HEAP (experimental): Store RDD in serialized format in Tachyon. This level reduces garbage collection overhead and allows the executors to be smaller and to share a pool of memory.

## **10. Which storage level to choose?**
- If your RDDs fit conmfortably with the default storage level (MEMORY_ONLY), leave them that way. This is the most CPU-efficient option, allowing operations on the RDDs to run as fast as possible.
- If not, try using MEMORY_ONLY_SER and selecting a fast serialization library to make the objects much more space efficient, but still reasonably fast to access.
- Don't spill to disk unless the functions that computed your dataset are expensive, or they filter a large amount of the data. Otherwise, recomputing a partition may be as fast as reading if it from disk.
- Use the replicated storage levels if you want fast fault recovery (e.g. If using Spark to serve requests from a web application). All the storage levels provide full fault tolerance by recomputing lost data, but the replicated ones let you continue running tasks on the RDD without waiting to recompute a lost partition.
- In the environments with high amounts of memory or multiple applications, the experiemntal OFF_HEAP mode has several advantages:
	- It allows multiple executors to share the same pool of memory in Tachyon.
	- It significantly reduces garbage collection costs.
	- Cached data is not lost if individual executors crash.
- More in Sparks website.

## **11. Shared variables and key-value pairs**

- When a function is passed from the driver to a worker, normally a swparate copy fo the variables are used.
- Two types of varibales:
	- Broadcast variables
		- Read-only copy on each machine
		- Distribute broadcast variables using efficient broadcast algorithms
	- Accumulators
		- Variables added through an associative operation
		- Implement counters and sums
		- Only the driver can read the accumulators value
		- Numeric types accumulators. Extend for new types.

Scala: key-value pairs
val pair = ('a','b')
pair._1 // will return 'a'
pair._2 // will return 'b'

Python: key-value pairs
pair = ('a', 'b')
pair[0] // will return 'a'
pair[1] // will return 'b'

Java: key-value pairs
Tuple2 pair = new Tuple2('a','b');
pair._1 // will return 'a'
pair._2 // will return 'b'

Broadcast variables allow each machine to work with a read-only variable cached on each machine. Spark attempts to distribute broadcast variables using efficient algorithms. As an example, broadcast variables can be used to give every node a copy of a large dataset efficiently.

The other shared variables are accumulators. These are used for counters in sums that works well in parallel. These variables can only be added through an associated operation. Only the driver can read the accumulators value, not the tasks. The tasks can only add to it.

Sparks supports numeric types but programmers can add support for new types. As an example, you can use accumulator variables to implement counters or sums, as in MapReduce.

## **12. Programming with key-value pairs**
- There are special operations avilable on RDDs of key-value pairs
	- Grouping or aggregating elements by a key.
- Tuple2 objects created by writing (a,b)
	- Must import org.apache.spark.SparkContext_
- PairRDDFunctions contains key-value pair operations
	- reduceByKEY((a,b)=>a+b)
- Custom objects as key in key-value pair requires a custom equals() method with a matching hashCode() method.
- Example:
	- val textFile = sc.textFile('...')
	- val readmeCount = textFile.flatMap(line => line.split(" ")).map(word => word, 1)).reduceByKey(_+_)

In example, you have a textFile that is just a normal RDD. Then you perform some transformations on it and it creates a PairRDD which allow it to invoke the reduceByKey method that is part of the PairRDDFunctions API.

## **13. Summary**
You should be able to:
- Describe Spark's primary data abstraction
- Understand how to create parallelized collections and external datasets
- Work with Resilient Distributed Dataset (RDD) operations
- Utilize shared variables and key-value pairs.
