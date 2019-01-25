source: https://stackoverflow.com/questions/36023860/how-to-use-a-scala-class-inside-pyspark

# Could you call a Scala class inside PySpark?

**Answers**
Yes it is possible although can be far from trivial. Typically, you want a Java (friendly) wrapper so you don't have to deal with Scala features which cannot be easily expressed using plain Java and as a result don't play well Py4J gateway.

Assumming your class is in the package **com.example** and have Python DataFrame called **df**
```python
df = ... # Python DataFrame
```
You'll have to:
1. Build a jar using your favorite tools. Ex: http://www.scala-sbt.org/
2. Include it in the driver class path for example using **--driver-class-path** argument for PySpark shell / **spark-submit**. Depending on the exact code you may have to pass it using **--jars** as well
3. Extract JVM instance from a Python SparkContext instance:
```python
jvm = sc._jvm
```
4. Extract Scala **SQLContext** from a **SQLContext** instance:
```python
ssqlContext = sqlContext._ssql_ctx
```
5. Extract Java DataFrame from the **df**:
```python
jdf = df._jdf
```
6. Create new instance of **SimpleClass**:
```python
simpleObject = jvm.com.example.SimpleClass(ssqlContext, jdf, "v")
```
7. Call exe method and wrap the result using Python DataFrame
```python
from pyspark.sql import DataFrame
DataFrame(simpleObjet.exe(), ssqlContext)
```
The result should be a valid PySpark **DataFrame**. You can of course combine all the steps into a single call.

**Important**: This approach is possible only if Python code is executed solely on the driver. It cannot be used inside Python action or transformation. See **How to use Java/Scala function from an action or a transformation?** for details. https://stackoverflow.com/q/31684842/1560062
