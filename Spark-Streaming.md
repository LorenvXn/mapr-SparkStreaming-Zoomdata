


>Step I - Prepare Kafka environment


1) start Zookeeper

```
#  /opt/mapr/zookeeper/zookeeper-3.4.5/bin/zkServer.sh start /opt/mapr/kafka/kafka-0.9.0/config/zookeeper.properties &
[1] 47384
# JMX enabled by default
Using config: /opt/mapr/kafka/kafka-0.9.0/config/zookeeper.properties
Starting zookeeper ... STARTED
```
2) From another terminal, start the broker:

```
# /opt/mapr/kafka/kafka/kafka-0.9.0/bin/kafka-server-start.sh config/server.properties
```

<i> N.B: Created and used topic  <b>fast-messages</b>, with group-id <b>console-consumer-6246 </b> </i>
```
/opt/mapr/kafka/kafka/kafka-0.9.0/bin/kafka-topics.sh --create --zookeeper localhost:2181 \
--replication-factor 1 --partitions 1 --topic   fast-messages
Created topic "fast-messages".

```
```
# /opt/mapr/kafka/kafka-0.9.0/bin/zookeeper-shell.sh localhost ls "/consumers/console-consumer-6246/offsets"
Connecting to localhost

WATCHER::

WatchedEvent state:SyncConnected type:None path:null
[fast-messages]
#
```




>Step II - Spark Streaming - integration with Kafka Consumer

<i>Below Scala lines work in either Zeppelin or spark-shell </i>

<i> Directory /tmp/streaming_output/ has been created </i>

Introduce in spark-shell the below lines:

```
import _root_.kafka.serializer.DefaultDecoder
import _root_.kafka.serializer.StringDecoder
import org.apache.spark.storage.StorageLevel
import org.apache.spark.streaming._
import org.apache.spark.streaming.kafka09.LocationStrategies.PreferConsistent
import org.apache.spark.streaming.kafka09.ConsumerStrategies.Subscribe
import org.apache.kafka.clients.consumer.ConsumerRecord
import org.apache.kafka.common.serialization.StringDeserializer
import org.apache.spark.streaming.kafka09._
import org.apache.spark.streaming.kafka09.{ConsumerStrategies, KafkaUtils, LocationStrategies}
import org.apache.kafka.clients.consumer.ConsumerConfig


sc.setLogLevel("ERROR")


val ssc = new StreamingContext(sc, Seconds(5))
val brokers = "localhost:9092"
val groupId="console-consumer-6246"
val offsetReset="earliest"
val pollTimeout ="1000"

val topics1 = Array("fast-messages")

val kafkaParams = Map[String, String](
    ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG -> brokers,
   ConsumerConfig.GROUP_ID_CONFIG -> groupId,
    ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG ->
      "org.apache.kafka.common.serialization.StringDeserializer",
    ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG ->
      "org.apache.kafka.common.serialization.StringDeserializer",
    ConsumerConfig.AUTO_OFFSET_RESET_CONFIG -> offsetReset,
    ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG -> "false",
    "spark.kafka.poll.time" -> pollTimeout)

val consumerStrategy = ConsumerStrategies.Subscribe[String, String](topics1, kafkaParams)

val messages = KafkaUtils.createDirectStream[String, String](ssc, LocationStrategies.PreferConsistent, consumerStrategy)

 
 val hdfsdir = "/tmp/streaming_output/"    

 val lines = messages.map(_.value())

lines.foreachRDD(rdd => {
     if (rdd.count() > 0) {
 rdd.saveAsTextFile(hdfsdir)
 }
 })

ssc.start()
```

From a different terminal, start the producer and send the csv file:

```
#/opt/mapr/kafka/kafka-0.9.0/bin/kafka-console-producer.sh --broker-list localhost:9092 \
--topic  fast-messages < /home/packetzoutput.csv
# echo $?
  0
#
```

Check under /tmp/streaming_output if the files are present:

```
[mapr@Host ~]$ hadoop fs -ls /tmp/streaming_output
Found 2 items
-rwxr-xr-x   3 root root          0 2017-06-02 08:47 /tmp/streaming_output/_SUCCESS
-rwxr-xr-x   3 root root       8898 2017-06-02 08:47 /tmp/streaming_output/part-00000
[mapr@instance-29219 ~]$
```

Now proceed with step III: https://github.com/Satanette/mapr-SparkStreaming-Zoomdata/blob/master/HiveMetastore.md
