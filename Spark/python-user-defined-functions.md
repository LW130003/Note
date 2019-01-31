source: https://docs.databricks.com/spark/latest/spark-sql/udf-python.html
# User-Defined Functions - Python
This topic contains Python user-defined function (UDF) examples. It shows how to register UDFs, how to invoke UDFs, and caveats regarding evaluation order of subexpressions in Spark SQL.

## Register a function as a UDF
```python
def squared(s):
  return s * s
spark.udf.register("squaredWithPython", squared)
```
You can optionally set the return type of your UDF. the default return type is StringType.
```python
from pyspark.sql.types import LongType
def squared_typed(s):
  return s * s
spark.udf.register("squaredWithPython", squared, LongType())
```

## Call the UDF in Spark SQL
```python
spark.range(1, 20).registerTempTable("test")
%sql select id, squaredWithPython(id) as id_squared from test
```

## Use UDF with DataFrames
```python
from pyspark.sql.functions import udf
from pyspark.sql.types import LongType
squared_udf = udf(squared, LongType())
df = spark.table("test")
display(df.select("id", squared_udf("id").alias("id_squared")))
```

Alternatively, you can declare the same UDF using annotation syntax:
```python
from pyspark.sql.functions import udf
@udf("long")
def squared_udf(s):
  return s * s
df = spark.table("test")
display(df.select("id", squared_udf("id").alias("id_squared")))
```

## Evaluation Order and Null Checking
Spark SQL (including SQL and the DataFrame and Dataset API) does not guarantee the order of evaluation of subexpressions. In particular, the inputs of an operator or function are not necessarily evaluated left-to-right or in any other fixed order. For example, logical AND and OR expressions do not have left-to-right “short-circuiting” semantics.

Therefore, it is dangerous to rely on the side effects or order of evaluation of Boolean expressions, and the order of WHERE and HAVING clauses, since such expressions and clauses can be reordered during query optimization and planning. Specifically, if a UDF relies on short-circuiting semantics in SQL for null checking, there’s no guarantee that the null check will happen before invoking the UDF. For example,
