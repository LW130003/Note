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

## Cakk t
