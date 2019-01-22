# Serialization Challenges with Spark and Scala - Part 2 - Now for something really challenging

https://medium.com/onzo-tech/serialization-challenges-with-spark-and-scala-part-2-now-for-something-really-challenging-bd0f391bd142

Each example steps through some ways you may try to debug the problems, eventually resulting in a working solution.

## 9. Base Example **FAILS**
```scala
object Example {
    val testRdd = sc.parallelize(List(1,2,3,4,5))
    class WithFunction(val num: Int){
        def plusOne(num2: Int) = num2 + num
    }
    
    class WithSparkMap(reduceInts: Int => Int){
        def myFunc = 
            testRdd.map(reduceInts).collect.toList
    }
    
    def run = {
        val withFunction = new WithFunction(1)
        val withSparkMap = new WithSparkMap(withFunction.plusOne)
        withSparkMap.myFunc
    }
}
Example.run
```

This example is relatively complex and needs a few changes to work successfully. The next few examples walk through a solution step by step, and some things you may try.

## 10. Make Class Serializable **FAILS**
```scala
object Example {
    val testRdd = sc.parallelize(List(1,2,3,4,5))
    class WithFunction(val num: Int) extends Serializable {
        def plusOne(num2: Int) = num2 + num
    }
    
    class WithSparkMap(reduceInts: Int => Int) extends Serializable {
        def myFunc =
            testRdd.map(reduceInts)
            .collect
            .toList
    }
    
    def run = {
        val withFunction = new WithFunction(1)
        val withSparkMap = new WithSparkMap(withFunction.plusOne)
        withSparkMap.myFunc
    }
}
Example.run
```

One approach to serialization issues can be to make everything Serializable. However, in this case you will find it doesn't solve the issue. You'll find it easier (but not that easy..!) to spot why if you look at the complete examples. It's because when trying to serialize the classes will find references to testRdd. This will trigger Serialization of the Example and the Example class is not Serializable.

## 11a. Use anon function **FAILS**
```scala
object Example{
    val testRdd = sc.parallelize(List(1,2,3,4,5))
    class WithSparkMap(reduceInts: Int => Int){
        def myFunc = {
            testRdd
                .map(e => reduceInts(e))
                .collect
                .toList
        }
    }
    
    def run = {
        val withSparkMap = new WithSparkMap(num => num + 1)
        withSparkMap.myFunc
    }
}
Example.run
```
In order to debug this you might try simplifying things by replacing the WithFunction class with a simple anonymous function. However, in this case it still have failure. Why?

## 11b. Use anon function, with enclosing **PASSES**
```scala
object Example {
    val testRdd = sc.parallelize(List(1,2,3,4,5))
    class WithSparkMap(reduceInts: Int => Int){
        def myFunc = {
            val reduceIntsEnc = reduceInts
            testRdd
                .map{ e =>
                    reduceIntsEnc(e)                
                }
                .collect
                .toList
        }
    }
    def run = {
        val withSparkMap = new WithSparkMap(num => num + 1)
        withSparkMap.myFunc
    }
}
Example.run
```
By enclosing the reduceInts method the map function can now access everything it needs in that one closure, no need to serialize the other classs.

## 12a. Use function with def **FAILS**
```scala
object Example {
    val testRdd = sc.parallelize(List(1,2,3,4,5))
    class WithSparkMap(reduceInts: Int => Int) {
        def myFunc = {
            val reduceIntsEnc = reduceInts
            testRdd
                .map{ e => 
                    reduceIntsEnc(e)
                }
                .collect
                .toList                
        }
    }
    
    def run = {
        def addOne(num: Int) = num + 1
        val withSparkMap = new WithSparkMap(num => addOne(num))
        withSparkMap.myFunc
    }
}
Example.run
```
Taking small steps, we now replace the anonymous function with a function declared with a def. Again you will find this fails, but seeing why isn't easy. It is because of how **def** works. Essentially, a method defined with def contains an implicit reference to **this**, which in this case is an object which can't be serialized. 

To find out more about differences between def and val here: https://alvinalexander.com/scala/fp-book-diffs-val-def-scala-functions

## 12b. Use function with val **PASSES**
```scala
object Example {
    val testRdd = sc.parallelize(List(1,2,3,4,5))
    class WithSparkMap(reduceInts: Int => Int) {
        def myFunc = {
            val reduceIntsEnc = reduceInts
            testRdd
                .map(e => reduceIntsEnc(e))
                .collect
                .toList                    
        }
    }
    
    def run = {
        val addOne = (num: Int) => num + 1
        val withSparkMap = new WithSparkMap(num => addOne(num))
        withSparkMap.myFunc
    }
}
Example.run
```
Declaring the method with **val** works. A **val** method equates to a Function1 object, which is serializable, and doesn't contain an implicit reference to this, stopping the attempted serialization of the Example object.

## 12c. use function with val explained part 1 **FAILS**
```scala
object Example {
    val one = 1
    
    class WithSparkMap(reduceInts: Int => Int) {
        def myFunc = {
            val reduceIntsEnc = reduceInts
            testRdd
                .map(e => reduceIntsEnc(e))
                .collect
                .toList
        }        
    }
    
    def run = {
        val addOne = (num: Int) => num + one
        val withSparkMap = new WithSparkMap(num => addOne(num))
        withSparkMap.myFunc
    }
}
```
