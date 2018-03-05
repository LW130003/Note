# **Sparks Fundamental I: **Module 1: Introduction to Spark**
## **1. Objectives**:
- Explain the purpose of Spark
- List and describe the components of the Spark unified stack.
- Understanding the basics of Resilent Distributed Dataset (RDD)
- Downloading and installing Spark standalone
- Scala and Python overview
- Launch and use Spark's Scala and Python shell

## **2. Big Data and Spark**
- Data is increasing in volume, velocity, variety.
- The need to have faster results from analytics becomes increasingly important.
- Apache Spark is a computing platform designed to be *fast* and *general-purpose*, and easy to use:
	- **Speed**
		- In-memory computations
		- Faster than MapReduce for complex applications on disk
	- **Generality
		- Covers a wide range of workloads on one system
		- Batch applications (e.g. MapReduce)
		- Iterative algorithms
		- Interactive queries and streaming
	- **Easy to use**
		- APIs for Scala, Python, Java
		- Libraries for SQL, machine learning, streaming, and graph processing
		- Runs on Hadoop clusters or as a standalone

## **3. Who uses Spark and why?**
- Parallel distributed processing, fault tolerance on commodity hardware, scalability, in-memory computing, high level APIs, etc.
- Saves time and money.
- **Data scientist**:
	- Analyze and model the data to obtain insight using ad-hoc analysis.
	- Transforming the data into a usable format
	- Statistics, machine learning, SQL
- **Engineers**:
	- Develop a data processing system or application
	- Inspect and tune their applications
	- Programming with the Spark's API
- **Others**:
	- Easy of use
	- Wide variety of functionality
	- Mature and reliable

## **4. Spark Unified Stack**
- Layer 1:
	- Spark SQL
	- Spark Streaming (Real Time Processing)
	- MLib (Machine Learning)
	- GraphX (Graph Processing)
- Layer 2:
	- Spark Core
- Layer 3:
	- Standalone Scheduler
	- YARN
	- Mesos

### **4.1 Spark Core**
<p> 
The Spark core (center of Spark unified stack) is a general-purpose system providing scheduling, distributing, and monitoring of the applications across a cluster.
</p>
<p>
Spark SQL, Spark Streaming, MLib and graphX are components that are designed to interoperate closely, letting the users combine them, just like they would any libraries in a software project. 
</p>
<p>
The benefit of such a stack is that all the higher layer components will inherit the improvements made at the lower layesr. Example: Optimization to the Spark Core will speed up the SQL, the streaming, the machine learning and the graph processing libraries as well. 
</p>
<p>
The Spark core is designed to scale up from one to thousands of nodes, it can run over a variety of cluster managers including Hadoop YARN and Apache Mesos. Or simply, it can even run as a standalone with its own built-in schduler.
</p>
### **4.2 Top Layer**
- **Spark SQL** is designed to work with the Spark via SQL and HiveQL (a Hive variant of SQL). Spark SQL allows developers to intermix SQL with Spark's programming language supported by Python, Scala, and Java.
- **Spark Streaming** provides processing of live streams of data. The Spark streaming API closely matches that of the Sparks Core's API, making it easy for developers to move between applications that processes data stored in memory vs arriving in real-time. It also provides the same degree of fault tolerance, throughput, and scalability that the Spark Core provides.
- **Mlib** is the machine learning library that provides multiple type of machine learning algorithms. All of these algorithms are designed to scale out across the cluster as well.
- **GraphX** is a graph processing library with APIs to manipulate graphs and performing graph-parallel computations.

## **5. Brief History of Sparks**
- 2002 - MapReduce @ Google
- 2004 - MapReduce paper
- 2006 - Hadoop @ Yahoo
- 2008 - Hadoop Summit
- 2010 - Spark paper
- 2014- Apache Spark top-level

<p>
MapReduce was designed as a fault tolerant framework that ran on commodity systems. Spark comes out about a decade later with the similar framework to rund data processing on commodity systems also using a fault tolerant framework.
</p>

<p>
MapReduce started a general batch processing paradigm, but there are two major limitations:
</p>
1. Difficulty programming in MapReduce
2. Batch processing did not fit many use cases
<p>
So this spawned a lot of specialized systems (Storm, Impala, Giraph, etc.) to handle other use cases. When you try to combine these third party systems in your applications there are a lot of overhead. 
</p>
On the other hands, Spark requires a considerable amount less. Even with Spark's libraries, it only adds a small amount of code due to how tightly everything is integrated with very little overhead. ***There is great value to be able to express a wide variety of use cases with few lines of code.***

## **6. Resilient Distributed Datasets (RDD)**

Spark's primary core abstraction is called Resillient Distributed Dataset or RDD. Essentially it just a **distributed collection of elements** that is **parallelized across the cluster**.

There are **two types** of RDD operations:
- Transformations
	- Creates a Direct Acyclic Graphs (DAG)
	- Only be evaluated at runtime (we call this Lazy evaluations)
	- No return value
- Actions
	- Performs the transformations and the action that follows
	- Returns a value
The **Fault tolerance** aspect of RDDs allows Spark to reconstruct the transformations used to build the lineage to get back the lost data. 

**Actions** are when the transformations get evaluated along with the action that is called for that RDD. Actions returns values. For example, You can do a count on a RDD, to get the number of elements within and that value is returned:

**Hadoop RDD** -> **Filtered RDD** -> **Mapped RDD** -> **Reduced RDD**

1. Load the dataset from Hadoop. 
2. Then apply successive transformations on dataset such as filter, map, or reduce. 

Nothing actually happens until an action is called. The DAG is just updated each time until an action is called. This provides fault tolerance. For example, let's say a node goes offline. All it needs to do when it comes back online is to reevaluate the graph to where it left off

Caching is provided with Spark to enable the processing to happen in memory. If it does not fit in memory, it will spill the disk

## **7. Downloading and installing Spark**

- Runs on both Windows and Unit-like systems (e.g. Linux, Mac OS)
- To run locally on one machine, all you need is to have Java installed on your system PATH or the JAVA_HOME pointing to a valid Java installation.
- Visit this page to download: http://spark.apache.org/downloads.html
	- Select the hadoop distribution you require under the "Pre-built packages"
	- Place a compiled version of Spark on each node on the cluster.
- Manually start the cluster by executing:
	- /sbin/start-master.sh
- Once started, the master will print out a spark://HOST:PORT URL for itself, which you can use to connect workers to it.
	- The default master's web UI is http://localhost8080
- Check out Spark's website for more information
	- http://spark.apache.org/docs/latest/spark-standalone.html

## **8 Spark jobs and shell**
- Spark jobs can be written in Scala, Python or Java. 
- Spark shells are available for Scala or python.
- APIs are available for all three.
- Must adhere to the appropriate versions for each Spark release.
- Spark's native language is Scala, so it is natural to write Spark applications using Scala.

**Note**: 
- It is recommended that you have at least some programming background to understand how to code in any of these.
- If you are setting up the Spark cluster yourself, you will have to make sure that you have a compatible version of the programming language you choose to use. This information can be found on Spark's website.
- In the lab environment, everything has been set up for you - all you do is launch up the shell and you are ready to go. 
- Java 8 Actually supports the functional programming style to include labdas, which concisely captures the functionality that are executed by the Spark engine. This bridge the gap between Java and Scala for developing applicaitons on Spark.
- Java 6 and 7 is supported but would require more work and an additional library to get the same amount of functionality as you would using Scala or Python.

## **9. Brief overview of Scala**
- Everything is an Object:
	- Primitive types such as numbers or boolean
	- Functions
- Numbers are objects
	- 1 + 2 * 3 / 4 -> (1).+(((2).*(3))./(x)))
	- Where the +,*,/ are valid identifiers in Scala
- Functions are Objects
	- Pass functions as arguments
	- Store them in variables
	- Return them from other functions
- Function declaration
	- def functionName ([list of parameters]): [return type]

Functions are objects in Scala and will play an important role in how applications are written for Spark. 

Numbers are objects. This means that in an expression that you see here: 1+2*/4 actually means that the individual numbers invoke the various identifiers +,=,*,/ wth the other numbers passed in as arguments using the dot notation.

Functions are objects. You can pass functions as arguments into another function. You can store them as variables. You can return them from other functions.

The Function Declaration is the function name followed by the list of parameters and then the return type.

**Note**: Learn more about Scala from its website for tutorials and guide

## **10. Scala - anonymous functions**
Anonymous functions are very common in Spark applications. Essentially, if the function you need is only going to be required once, there is really no value naming it. Just use it anonymously on the go and forget about it.

## **11. Spark's Scala and Python shell**
The **Spark Shell**:
- provides a simple way to learn Spark's API. 
- is also a powerful tool to analyze data interactively. 
- is available in either Scala, which runs on the Java VM (A good way to use existing Java libraries), or Python.

**Scala**:
- To launch the Scala shell:
	- ./bin/spark-shell
- To create a RDD from a text file (read in a text file):
	- scala> val textFile = sc.textFile('README.md")

**Python**:
- To launch the Python shell:
	- .bin/pyspark
- To read in a text file:
	- >>> textFile = sc.textFile("README.md")

## **12. Summary**
- Explain the purpose of Spark
- List and describe the components of the Spark Unified Stack
- Understand the basics of Resillient Distributed Dataset (RDD)
- Downloading and Installing Spark standalone
- Scala and Python overview
- Launch and use Spark's Scala and Python shell.


