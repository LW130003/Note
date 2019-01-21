# Debugging Apache Spark
## "Professional Stack Trace Reading"
### with your friends Holden & Joey

https://www.slideshare.net/hkarau/debugging-apache-spark-scala-python-super-happy-fun-times-2017

## What will be covered?
- Getting at Spark's logs & persisting them
- What your options for logging are
- Attempting to understand Spark error messages
- Understanding the DAG (and how pipelining can impact your life)
- Subtle attempts to get you to use spark-testing-base or similar
- Fancy Java Debugging tools & clusters - not entirely the path of sadness

## So where are the logs/errors?
**(e.g. before we can identify a monster we have to find it)**
- Error messages reported to the console*
- Log messages reported to the console*
- Log messages on the workers - access through the Spark Web UI or Spark History Server
- Where to error: driver versus worker
(*When running in client mode)

## One weird trick to debug anything
- Don't read the logs (yet)
- Draw (possibly in your head) a model of how you think a working app would behave
- Then predict where in that model things are broken
- Now read logs to prove or disprove your theory
- Repeat

## Working in YARN?
**(e.g. before we can identify a monster we have to find it)**
- Use yarn logs to get logs after log collection
- Or set up the Spark History Server
- Or yarn.nodemanager.delete.debug-delay-sec

## Spark is pretty verbose by default
- Most of the time it tells you things you already know
- Or don't need to know
- You can dynamically control the log level with **sc.setLogLevel**
- This is especially useful to increase logging near the point of error in your code

## But what about when we get an error?
- Python Spark errors come in two-ish-parts often
- JVM Stack Trace
- Python Stack Trace

## So what is that JVM stack trace?
- Java/Scala
  - Normal stack trace
  - Sometimes can come from worker or driver, if from worker may be repeated several times for each partition & attempt with the error
  - Driver stack trace wraps worker stack trace
- R/Python
  - Same as above but...
  - Doesn't want your actual error message to get lonely
  - Wraps any exception on the workers (& some exceptions on the drivers)
  - Not always super useful

# Let's make a simple mistake & debug
## Error in transformation (divide by zero)
**Bad Outer transformation (Scala):**
```scala
val transform1 = data.map(x => x + 1)
val transform2 = transform1.map(x => x/0) // will thrown an exception when forced to evaluate
transform2.count() // Forces evaluation
```
**Bad Outer Transformation (Python):**
```python
data = sc.parallelize(range(10))
transform1 = data.map(lambda x: x + 1)
transform2 = transform1.map(lambda x: x/0)
transform2.count()
```

**Working in Jupyter > Try terminal > web UI is easier > Scroll down (not quite to the bottom) > Or look at the bottom of console logs >**

**"The error messages were so useless - I looked up how to disable error reporting in Jupyter" (paraphrased from PyData DC)**

**And in Scala.... > DAG differences illustrated (Python is simpler than Scala) 

## Pipelines (& Python)
- Some pipelining happens inside of Python
  - For performance (less copies from Python to Scala)
- DAG visualization is generated inside of Scala
  - Misses Python pipelines

Regardless of language
- Can be difficult to determine which element failed
- Stack trace _sometimes_helps (it did this time)
- take(1) + count() are your friends - but a lot of work 
- Persist can help a bit too

Side note: Lambdas aren't 
