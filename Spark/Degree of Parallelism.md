# Apache Spark Performance Tuning - Degree of Parallelism

This is the third article of a four-part series about Apache Spark on YARN.

- Apache Spark allows developers to run multiple tasks in parallel across machines in a cluster, or across multiple cores on a desktop.
- A partition, or split, is a logical chunk of a distributed data set.
- Apache Spark builds a Directed Acyclic Graph (DAG) with jobs, stages, and tasks for the submitted application.
- The number of tasks will be determined based on the number of partitions.

## Spark Partition Principles
The general principle to be followed when tuning partition for Spark Application are as follows:
- Too few partitions - Cannot utilize all cores available in the cluster
- Too many partitions - Excessive overhead in managing many small tasks
- Reasonable partitions - Helps us to utilize the cores available in the cluster avoids excessive overhead in managing small tasks

## Understanding Spark Data Partitions


