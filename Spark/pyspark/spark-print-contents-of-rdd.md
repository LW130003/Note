# Spark - Prints contents of RDD
RDD (Resilient Distributed Dataset) is a fault-tolerant collection of elements that can be operated on in parallel. In this tutorial, we shall learn some of the ways in Spark to print contents of RDD.

## Use RDD collect Action
RDD.collect() returns all the elements of the dataset as an array at the driver program, and using for loop on this array, print elements of RDD.

```python
import sys
 
from pyspark import SparkContext, SparkConf
 
if __name__ == "__main__":
 
  # create Spark context with Spark configuration
  conf = SparkConf().setAppName("Print Contents of RDD - Python")
  sc = SparkContext(conf=conf)
 
  # read input text file to RDD
  rdd = sc.textFile("data/rdd/input/file1.txt")
 
  # collect the RDD to a list
  list_elements = rdd.collect()
 
  # print the list
  for element in list_elements:
    print(element)
```

## Use RDD foreach action
RDD foreach(func) runs a function func on each element of the dataset.

```python
import sys
 
from pyspark import SparkContext, SparkConf
 
if __name__ == "__main__":
 
  # create Spark context with Spark configuration
  conf = SparkConf().setAppName("Print Contents of RDD - Python")
  sc = SparkContext(conf=conf)
 
  # read input text file to RDD
  rdd = sc.textFile("data/rdd/input/file1.txt")
 
  def f(x): print(x)
 
  # apply f(x) for each element of rdd
  rdd.foreach(f)
```
