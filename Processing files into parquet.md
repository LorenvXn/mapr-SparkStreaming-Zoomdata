
As per output from [part 1](https://github.com/Satanette/mapr-SparkStreaming-Zoomdata/blob/master/Spark-Streaming.md), 
we will now process files under /tmp/streaming_output into parquet 

```
import org.apache.spark._
import org.apache.spark.SparkContext._
import org.apache.spark.streaming._
import org.apache.spark.sql.types.{StructType,StructField,StringType}
import sqlContext.implicits._
import org.apache.spark.sql.Row

                                                                                                                    
val appName = "text to parquet"
val conf = new SparkConf()

conf.setAppName(appName)

val sqlContext = new org.apache.spark.sql.SQLContext(sc)

val rawRdd = sc.textFile("/tmp/streaming_output")

val schemaString = "date prot1 IPdestination portDestination " + "IPSource portSource prot2 Length"

val schema = StructType(schemaString.split(" ").map(fieldName => StructField(fieldName,StringType, true)))

val rowRDD = rawRdd.map(_.split(",")).map(p => Row( p(0),p(1),p(2),p(3),p(4),p(5),p(6),p(7)))

val packetZDataFrame = sqlContext.createDataFrame(rowRDD, schema)

packetZDataFrame.printSchema()

packetZDataFrame.select("portDestination", "IPdestination", "length").filter(packetZDataFrame("length")>300).show()

val parquet = packetZDataFrame.write.parquet("DemPacketZ.parquet")

```

Your schema should be looking like this:

```
scala> packetZDataFrame.printSchema()
root
 |-- date: string (nullable = true)
 |-- prot1: string (nullable = true)
 |-- IPdestination: string (nullable = true)
 |-- portDestination: string (nullable = true)
 |-- IPSource: string (nullable = true)
 |-- portSource: string (nullable = true)
 |-- prot2: string (nullable = true)
 |-- Length: string (nullable = true)
 
 ```
 
The select should be looking like this( just testing... )
 
 ```
 scala> packetZDataFrame.select("portDestination", "IPdestination", "length").filter(packetZDataFrame("length")>300).show()
 
 ------------+--------------+------+
|portDestination| IPdestination|length|
+---------------+--------------+------+
|          42618| 100.64.2.134 |  2896|
|          42618| 100.64.2.134 |  4344|
|          42618| 100.64.2.134 |  8688|
|          42618| 100.64.2.134 | 16840|
+---------------+--------------+------+
 
 ```
 The Parquet output:
 
 ```
 $ hadoop fs -ls /user/root/DemPacketZ.parquet
Found 3 items
-rwxr-xr-x   3 root root          0 2017-06-09 09:57 /user/root/DemPacketZ.parquet/_SUCCESS
-rwxr-xr-x   3 root root       2963 2017-06-09 09:57 /user/root/DemPacketZ.parquet/part-00000-0de89ecf-b39b-4c6c-938d-ca4e67a54f4c.snappy.parquet
-rwxr-xr-x   3 root root       2979 2017-06-09 09:57 /user/root/DemPacketZ.parquet/part-00001-0de89ecf-b39b-4c6c-938d-ca4e67a54f4c.snappy.parquet

 
 ```
 
If you want to process your files into JSON (and then into parquet ... or not :v)  as soon as data frame is created, 
it must be saved into a JSON format. 

Just don't forget to offer a path where the new files to be saved:

```
val jsonData = packetZDataFrame.toJSON
jsonData.rdd.saveAsTextFile("/tmp/somestuff.json")
```








