# Optimization Techniques

There are several aspects of tuning Spark Applications toward better optimization tehcniques. In this section, we will discuss how we can further optimize our Spark applications by applying data serialization by tuning the main memory with better memory management. We can also optimize performance by tuning the data structure in your Scala code while developing Spark applications. The storage, on the other hand, can be maintained well by utilizing serialized RDD storage.

One of the msot important aspects is garbage collection, and it's tuning if you have written your Spark application using Java or Scala. We will look at how we can also tune this for optimized performance. For distributed environment- and cluster-based system, a level of parallelism and data locality has to be ensured. Moreover, performance could be further be improved by using broadcast variables.

# Data Serialization

Serialization is an important tuning for performance improvement and optimization in any distributed computing environment. Spark is not an exception, but Spark jobs are often data and computing extensive. Therefore, if your data objects are not in a good format, then you first need to convert them into serialized data objects. This demands a large number of bytes of your memory. Eventually, ht ewhole process will slow down the entire processing and computation drastically.

