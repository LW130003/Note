# Scala Fundamentals - Development Cycle - Build and run jar file using set
source: https://spark.apache.org/docs/latest/submitting-applications.html
## Development Cylce
- Develop the application
- Build a jar file
- Run as jar file

src/main/scala
src - source code
main - main module
scala - programming language

## Spark submit commands
```bash
./bin/spark-submit \
  --class <main-class> \
  --master <master-url> \
  --deploy-mode <deploy-mode> \
  --conf <key>=<value> \
  ... # other options
  <application-jar> \
  [application-arguments]
```

## Examples
## Write scala code
```scala
package com.scalapyspark

object SelfHelp {
    def quoteRandall = println("Open unmarked doors")
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.scalapyspark</groupId>
    <artifactId>scalapyspark</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <!-- https://mvnrepository.com/artifact/org.apache.spark/spark-core -->
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-core_2.11</artifactId>
            <version>2.3.0</version>
        </dependency>
    </dependencies>
</project>
```
## Build a jar file
```bash
mvn package
```
## Running a jar file
After creating a jar file, copy the file to Linux Machine (if you develop in the windows) and use the spark-submit commands:
1. Go to the project directory, where you put the jar file (Or you can use the full name location)
2. Run the spark-submit command
```bash
spark-submit \
  --class com.scalapyspark.SelfHelp \
  --master /home/hdfsf10n/livardy/wafer-map-similarity/scalapyspark-1.0-SNAPSHOT.jar \
  --deploy-mode cluster \
```

