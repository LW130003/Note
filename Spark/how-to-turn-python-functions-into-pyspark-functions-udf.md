# How to Turn Python Functions into PySpark Functions (UDF)
https://changhsinlee.com/pyspark-udf/

**Problem**: I have a Python funciton that iterates over my data, but going through each row in the dataframe takes several days. If I have a computing cluster with many nodes, how can I distribute this Python function in PySpark to speed up this process - maybe cut the total time down to less than a few hours - with the least amount of work? In other words, **how do I turn a Python function into a Spark user defined function, or UDF?**

## Registering a UDF
PySpark UDFs work in a similar way as the pandas **.map()** and **.apply()** methods for pandas series and dataframes. If I have a function that can use values from a row in the dataframe as input, then I can map it ot the entire dataframe. The only difference is that with PySpark UDFs I have to specify the output data type.

As an example, I will create a PySpark dataframe from a pandas dataframe.
```python
# example data
df_pd = pd.DataFrame(
    data={'integers': [1, 2, 3],
     'floats': [-1.0, 0.5, 2.7],
     'integer_arrays': [[1, 2], [3, 4, 5], [6, 7, 8, 9]]}
)
df = spark.createDataFrame(df_pd)
df.printSchema()

root
 |-- floats: double (nullable = true)
 |-- integer_arrays: array (nullable = true)
 |    |-- element: long (containsNull = true)
 |-- integers: long (nullable = true)
 
 df.show()
 
 +------+--------------+--------+
|floats|integer_arrays|integers|
+------+--------------+--------+
|  -1.0|        [1, 2]|       1|
|   0.5|     [3, 4, 5]|       2|
|   2.7|  [6, 7, 8, 9]|       3|
+------+--------------+--------+
```

## Primitive Type Outputs
Let's say I have a Python function **square()** that squares a number, and I want to register this function as a Spark UDF.
```python
def square(x):
    return x**2
```
As long as the python function's output has a corresponding data type in Spark, then I can turn it into a UDF. When registering UDFs, I have to specify the data type using the types from **pyspark.sql.types**.

Here’s a small gotcha — because Spark UDF doesn’t convert integers to floats, unlike Python function which works for both integers and floats, a Spark UDF will return a column of NULLs if the input data type doesn’t match the output data type, as in the following example.

### Registering UDF with integer type output
```python
# Integer type output
from pyspark.sql.types import IntegerType
square_udf_int = udf(lambda z: square(z), IntegerType())

(
    df.select('integers',
              'floats',
              square_udf_int('integers').alias('int_squared'),
              square_udf_int('floats').alias('float_squared'))
    .show()
)

+--------+------+-----------+-------------+
|integers|floats|int_squared|float_squared|
+--------+------+-----------+-------------+
|       1|  -1.0|          1|         null|
|       2|   0.5|          4|         null|
|       3|   2.7|          9|         null|
+--------+------+-----------+-------------+
```

### Registering UDF with float type output
```python
# float type output
from pyspark.sql.types import FloatType
square_udf_float = udf(lambda z: square(z), FloatType())

(
    df.select('integers',
              'floats',
              square_udf_float('integers').alias('int_squared'),
              square_udf_float('floats').alias('float_squared'))
    .show()
)

+--------+------+-----------+-------------+
|integers|floats|int_squared|float_squared|
+--------+------+-----------+-------------+
|       1|  -1.0|       null|          1.0|
|       2|   0.5|       null|         0.25|
|       3|   2.7|       null|         7.29|
+--------+------+-----------+-------------+
```

### Specifying float type output in the Python function

Specifying the data type in the Python function output is probably the safer way. Because I usually load data into Spark from Hive tables whose schemas were made by others, specifying the return data type means the UDF should still work as intended even if the Hive schema has changed.

```python
## Force the output to be float
def square_float(x):
    return float(x**2)
square_udf_float2 = udf(lambda z: square_float(z), FloatType())
(
    df.select('integers',
              'floats',
              square_udf_float2('integers').alias('int_squared'),
              square_udf_float2('floats').alias('float_squared'))
    .show()
)
+--------+------+-----------+-------------+
|integers|floats|int_squared|float_squared|
+--------+------+-----------+-------------+
|       1|  -1.0|        1.0|          1.0|
|       2|   0.5|        4.0|         0.25|
|       3|   2.7|        9.0|         7.29|
+--------+------+-----------+-------------+
```

### Composite Type Outputs
If the output of the Python function is a list, then the values in the list have to be of the same type, which is specified within ArrayType() when registering the UDF.
```python
from pyspark.sql.types import ArrayType

def square_list(x):
    return [float(val)**2 for val in x]

square_list_udf = udf(lambda y: square_list(y), ArrayType(FloatType()))

df.select('integer_arrays', square_list_udf('integer_arrays')).show()

+--------------+------------------------+
|integer_arrays|<lambda>(integer_arrays)|
+--------------+------------------------+
|        [1, 2]|              [1.0, 4.0]|
|     [3, 4, 5]|       [9.0, 16.0, 25.0]|
|  [6, 7, 8, 9]|    [36.0, 49.0, 64.0...|
+--------------+------------------------+
```
For a function that returns a tuple of mixed typed values, I can make a corresponding StructType(), which is a composite type in Spark, and specify what is in the struct with StructField(). For example, if I have a function that returns the position and the letter from ascii_letters,

```python
import string

def convert_ascii(number):
    return [number, string.ascii_letters[number]]

convert_ascii(1)

[1, 'b']

array_schema = StructType([
    StructField('number', IntegerType(), nullable=False),
    StructField('letters', StringType(), nullable=False)
])

spark_convert_ascii = udf(lambda z: convert_ascii(z), array_schema)

df_ascii = df.select('integers', spark_convert_ascii('integers').alias('ascii_map'))
df_ascii.show()

+--------+---------+
|integers|ascii_map|
+--------+---------+
|       1|    [1,b]|
|       2|    [2,c]|
|       3|    [3,d]|
+--------+---------+
```

Note that the schema looks like a tree, with nullable option specified in StructFiled().
```python
df_ascii.printSchema()

root
 |-- integers: long (nullable = true)
 |-- ascii_map: struct (nullable = true)
 |    |-- number: integer (nullable = false)
 |    |-- letters: string (nullable = false)
```

# Some UDF problem I have encountered
