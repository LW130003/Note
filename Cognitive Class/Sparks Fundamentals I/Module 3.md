# **Sparks Fundamental I: **Module 3: Spark Application Programming**
## **1. Objectives**:
- Understand the purpose and usage of the SparkContext
- Initialize Spark with the various pogramming languages
- Describe and run some Spark examples
- Pass functions to Spark
- Create and run a Spark standalone application
- Submit applications to the cluster

## **2. SparkContext
- The main entry point for Spark functionality
- Represents the connection to a Spark cluster
- Create RDDs, accumulators, and broadcast variables on that cluster
- In the Spark shell, the SparkContext, sc, is automatically initialized for you to use
- In a Spark program, import some classes and implicit conversions into your program:
	- import org.apache.spark.SparkContext
	- import org.apache.spark.SparkContext._
	- import org.apache.spark.SparkConf

