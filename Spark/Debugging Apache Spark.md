# Debugging Apache Spark "Professional Stack Trace Reading" with your friends Holden & Joey

https://www.slideshare.net/hkarau/debugging-apache-spark-scala-python-super-happy-fun-times-2017
https://www.youtube.com/playlist?list=PLRLebp9QyZtaoIpE2iaF3Q8itJOcdgYoX

## What will be covered?
- Getting at Spark's logs & persisting them
- What your options for logging are
- Attempting to understand Spark error messages
- Understanding the DAG (and how pipelining can impact your life)
- Subtle attempts to get you to use spark-testing-base or similar
- Fancy Java Debugging tools & clusters - not entirely the path of sadness

## So where are the logs/errors?
**(e.g. before we can identify a monster we have to find it)**
- Error messages reported to the console\*
- Log messages reported to the console\*
- Log messages on the workers - access through the Spark Web UI or Spark History Server
- Where to error: driver versus worker
(\*When running in client mode)

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

## Regardless of language**
- Can be difficult to determine which element failed
- Stack trace _sometimes_helps (it did this time)
- take(1) + count() are your friends - but a lot of work 
- Persist can help a bit too

**Side note: Lambdas aren't always your friend**
- Lambda's can make finding the error more challenging
- I love lambda x,y: x/y as much as the next human but when y is zero
- A small bit of refactoring for you debugging never hurt anyone\* (A blatant lie, but... it hurts less often than it helps)

## Testing - you should od it!
- spark-testing-base provides simple classes to build your Spark tests with
  - It's available on pip & maven central
- Look at the youtube playlist

## Adding your own logging:
- Java users use Log4J & friends
- Python users: use logging library (or even print!)
- Accumulators - behave a bit weirdly, don't put large amounts of data in them

## Also not all errors are "hard" errors
- Parsing input? Going to reject some malformed records
- flatMap or filter + map can make this simpler
- Still want to track number of rejected records (see accumulators)
- Invest in dead letter queues - e.g. write malformed records to an Apache Kafka topic

## So using names & logging & accs could be:
```python
data = sc.parallelize(range(10))
rejectedCount = sc.accumulator(0)
def loggedDivZero(x):
  import logging
  try:
    return [x/0]
  except Exception as e:
    rejectedCount.add(1)
    logging.warning("Error found " + repr(e))
    return []
transform1 = data.flatMap(loggedDivZero)
transform2 = transform1.map(add1)
transform2.count()
print("Reject " + str(rejectedCount.value))
```

## Ok what about if we run out of memory?
In the middle of some Java stack traces: - MemoryError
- Out of memory can be pure JVM (worker)
  - OOM exception during join
  - GC timelimit exceeded
- OutOfMemory error, Executors being killed by kernel, etc.
- Running in YARN? "Application overhead exceeded"
- JVM out of memory on the driver side from Py4J

## Reasons for JVM worker OOMs (w/PySpark)
- Unbalanced Shuffles
- Bufferring of Rows with PySpark + UDFs
  - If you have a down stream select move it up stream
- Individual jumbo records (after pickling)
- Off-heap storage
- Native code memory leak

## Reasons for Python worker OOMs (w/PySpark)
- Insufficient memory reserved for Python worker
- Jumbo records
- Eager entire partition evaluation (e.g. sort + mapPartitions)
- Too large partitions (unbalanced or not enough partitions)
- And loading invalid paths:

## Connecting Java Debuggers
- Add the JDWP incantation to your JVM launch: -agentlib:jdwp=transport=dt_socket,server=y,address=[debugport]
  - spark.executor.extraJavaOptions to attch debugger on the executors
  - --driver-java-options to attach on the driver process
  - Add "suspend=y" if only debugging a single worker & exiting too quickly
- JDWP debugger is IDE specific - Eclipse & IntelliJ have docs

## Connecting Python Debuggers
- You're going to have to change your code a bit
- You can use broadcast + Singleton "hack" to start pydev or desired remote debugging lib on all of the interpreter
- See https://wiki.python.org/moin/PythonDebuggingTools for your remote debugging options and pick the one that works with your toolchain.

## Alternative approaches
- move take(1) up the dependency chain
- DAG in the WebUI -- less useful for Python
- toDebugString -- also less useful in Ptyhon
- Sample data and run locally
- Running in cluster mode? Consider debugging in client mode




