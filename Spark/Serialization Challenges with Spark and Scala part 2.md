# Serialization Challenges with Spark and Scala - Part 2 - Now for something really challenging

https://medium.com/onzo-tech/serialization-challenges-with-spark-and-scala-part-2-now-for-something-really-challenging-bd0f391bd142

Each example steps through some ways you may try to debug the problems, eventually resulting in a working solution.

## 9. Base Example **\*\*FAILS\*\***
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

## 10. Make Class Serializable **\*\*FAILS\*\***
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

## 11a. Use anon function **\*\*FAILS\*\***
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

## 11b. Use anon function, with enclosing **\*\*PASSES\*\***
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

## 12a. Use function with def **\*\*FAILS\*\***
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

## 12b. Use function with val **\*\*PASSES\*\***
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

## 12c. use function with val explained part 1 **\*\*FAILS\*\***
```scala
object Example {
    val testRdd = sc.parallelize(List(1,2,3,4,5))
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
Example.run
```
This example serves to illustrate the point more clearly. Here the **addOne** function references the **one** value, which will cause the whole **Example** object to be serialized, which will fail.

**\*\*BONUS POINTS\*\***
One helpful experiment to try here is to resolve this by making the **Example** object Serializable.
```scala
object Example extends Serializable {
    val testRdd = sc.parallelize(List(1,2,3,4,5))
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
Example.run
```
Note: this method won't work if testRdd not inside Example, the reason is because it will trying to serialize the whole class / object where testRdd located.

## 12d. use function with val explained part 2 **\*\*PASSES\*\***
```scala
object Example {
    val testRdd = sc.parallelize(List(1,2,3,4,5))
    val one = 1
    
    class WithSparkMap(reduceInts: Int => Int){
        def myFunc = {
            val reduceIntsEnc = reduceInts
            testRdd
                .map(e => reduceIntsEnc(e))
                .collect
                .toList
        }
    }
    
    def run = {
        val oneEnc = one
        val addOne = (num: Int) => num + oneEnc
        val withSparkMap = new WithSparkMap(num => addOne(num))
        withSparkMap.myFunc
    }
}
Example.run
```
As above, the best way to fix the issue is to reference value only in the immediate scope. here we added oneEnc, which prevents the serialization of the whole Example object.

## 13. Back to the problem, no class params **\*\*PASSES\*\***
```scala
object Example {
    val testRdd = sc.parallelize(List(1,2,3,4,5))
    class WithFunction {
        val plusOne = (num2: Int) => num2 + 1
    }
    
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
        val withSparkMap = new WithSparkMap((new WithFunction).plusOne)
        withSparkMap.myFunc
    }
}
Example.run
```
Coming back from the issue we originally had, now we understand a little more let's introduce our WithFunction class back in. To simplify things we've taken out the constructor parameter here. We're also using a val or the method rather than a def. No serialization issue now!

## 14. Back to the problem, with class params **\*\*FAILS\*\***
```scala
object Example {
    val testRdd = sc.parallelize(List(1,2,3,4,5))
    class WithFunction(val num: Int) {
        val plusOne = (num2: Int) => num2 + num
    }
    
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
        val withSparkMap = new WithSparkMap(new WithFunction(1).plusOne)
        withSparkMap.myFunc
    }
}
Example.run
```
We've now added back in the class params. The plusOne function references num, outside the immediate scope, again causing more objects to be serialized which is failing.

## 15a. Back to the problem, with calss params, and enclosing **\*\*PASSES\*\***
```scala
object Example {
    val testRdd = sc.parallelize(List(1,2,3,4,5))
    
    class WithFunction(val num: Int) {
        val plusOne = {
            val encNum = num
            num2: Int => num2 + encNum
        }
    }
    
    class WithSparkMap(reduceInts: Int => Int){
        def myFunc = {
            val reduceIntsEnc = reduceInts
            testRdd
                .map(e => reduceIntsEnc(e))
                .collect
                .toList
        }
    }
    
    def run = {
        val withSparkMap = new WithSparkMap(new WithFunction(1).plusOne)
        withSparkMap.myFunc
    }    
}
Example.run
```
This is a simple fix, we can enclose the num value with encNum which resolve s the last of our serialization issues. Finally, this is a complete working example that is equivalent to our first implementation that failed!

## 15b. Adding some complexity - testing understanding **\*\*FAILS\*\***
```scala
object Example {
    val testRdd = sc.parallelize(List(1,2,3,4,5))
    class WithFunction(val num: Int){
        val plusOne = { num2: Int => 
            {
                val encNum = num
                num2 + encNum
            }
        }
    }
    
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
        val withSparkMap = new WithSparkMap(new WithFunction(1).plusOne)
        withSparkMap.myFunc
    }
}
Example.run
```
One more failing example! Can you see why the above fails?

The issue is that **encNum** won't be evaluated until plusOne is actually called, effectively within the map function. At this point then the num value will need to be accessed, causing additional serialization of the containing object and the failure here.
