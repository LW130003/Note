# PySpark DataFrame Tutorial: Introduction to DataFrames
https://dzone.com/articles/pyspark-dataframe-tutorial-introduction-to-datafra

## What are DataFrames?
DataFrames generally refer to a data structure, which is tabular in nature. It represents rows, each of which consists of a number of observations. Rows can have a variety of data format (heterogeneous), whereas a column can have data of the same data type (homogeneous). DataFrames usually contain some metadata in addition to data; for example, column and row names.

>![](https://d1jnx9ba8s6j9r.cloudfront.net/blog/wp-content/uploads/2018/06/table-528x215.jpg)

## Why Do We Need DataFrames?
### 1. Processing Structured and Semi-Structured Data
DataFrames are designed to process **a large collection of structured as well as semi-structured data**. Observations in Spark DataFrame are organized under named columns, which helps Apache Spark understand the schema of a DataFrame. This helps Spark optimize the execution plan on these queries. It can also handle petabytes of data.

### 2. Slicing and Dicing
DataFrames APIs usually support elaborate methods for slicing-and-dicing the data. It includes operations such as "selecting" rows, columns, and cells by name or by number, filtering out rows, etc. Statistical data is usually very messy and containing lots of missing and incorrect values and range violations. So a critically important feature of DataFrames is the explicit managament of missing data.

### 3. Data Sources
DataFrames has support for a wide range of data formats and sources, we'll look into this later on in this PySpark DataFrames tutorial.

### 4. Support for Multiple Languages
It has API support for different languages like Python, R, Scala, Java, which makes it easier to be used by people having different programming backgrounds.

## Feature of DataFrames
- DataFrames are **distributed** in nature, which makes it a fault tolerant and highly available data structure.
- **Lazy evaluation** is an evaluation startegy which holds the evaluation of an expression until its value is needed. It avoids repeated evaluation. Lazy evaluation in Spark means that the execution will not start until an action is triggered. In Spark, the picture of lazy evaluation comes when Spark transformations occur.
- DataFrames are **immutable** in nature. By immutable, I mean that it is an object whose state **cannot be modified** after it is created. But we can transform its values by applying a certain transformation like in RDDs.

## PySpark DataFrame Sources
PySpark DataFrame can be created in multiple ways:
- It can be created through loading a **CSV**, **JSON**, **XML**, or a Parquet file.
- It can also be created using an existing **RDD** and through any other database, like **Hive** or **Cassandra**.
- It can also take in data from HDFS or the local file system

Example
```python
from pyspark.sql import *

# Create Employee and Department Instances
Employee = Row("firstName", "lastName", "email", "salary")

employee1 = Employee('Basher', 'armbrust', 'bash@edureka.co', 100000)
employee2 = Employee('Daniel', 'meng', 'daniel@stanford.edu', 120000)
employee3 = Employee('Muriel', None, 'muriel@waterloo.edu', 140000)
employee4 = Employee('Rachel', 'wendell', 'rach_3@eureka.co', 160000)
employee5 = Employee('Zach', 'galifianakis', 'zac_3@eureka.co', 160000)

print(Employee[0])
print(employee3)

department1 = Row(id='123456', name='HR')
department2 = Row(id='789012', name='OPS')
department3 = Row(id='345678', name='FN')
department4 = Row(id='901234', name='DEV')

# Create DepartmentWithEmployees instance from the Employee and Departments
departmentWithEmployees1 = Row(department=department1, employees=[employee1, employee2, employee3])
departmentWithEmployees2 = Row(department=department2, employees=[employee3, employee4])
departmentWithEmployees3 = Row(department=department3, employees=[employee1, employee4, employee3])
departmentWithEmployees4 = Row(department=department4, employees=[employee2, employee3])

# Create DataFrame from the list or rows
deparmentWithEmployees_Seq = [departmentWithEmployees1, departmentWithEmployees2]
dframe = spark.createDataFrame(departmentsWithEmployees_Seq)
display(dframe)
dframe.show()
```

## PySpark DataFrames Example
```python
# 1. Reading Data from CSV file
fifa_df = spark.read.csv("path-of-file/fifa_players.csv", inferSchema=True, header=True)
fifa_df.show() # Show the top 20 rows

# 2. Schema of DataFrame
# Schema is the structure of the dataframe. printSchema method will give us the different columns in our DataFrame, along with the data type and the nullable conditions for that particular column
fifa_df.printSchema()

# 3. Column Names and Count (Rows and Columns)
# When we want to have a look at the names and a count of the number of rows and columns of a particular dataframe, we use the following methods.

fifa_df.columns # Column Names
fifa_df.count() # Row Count
len(fifa_df.columns) # Column Count
```
