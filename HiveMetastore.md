
>Step III 

Now that Step I and II are done, it's time to be using Hive. 

<i>Supposing directory /user/hive/zommeet has been created  </i>

Below lines, to be run either from spark-shell or Zeppelin:

```
import org.apache.spark.{SparkConf, SparkContext}
import org.apache.spark.sql._
import org.apache.spark.sql.hive.HiveContext

  val hiveContext = new HiveContext(sc)

   import hiveContext.implicits._
   import hiveContext.sql

      import scala.sys.process._

Seq("hadoop","fs","-cp","/tmp/streaming_output/*", "/user/hive/zoomeet/").!!

hiveContext.sql("create table if not exists monitoringpackets( date String, prot1  String, destination String, PortDestination String, source String, PortSource Integer, prot2 String, length String) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' LOCATION '/user/hive/zoomeet/'")

val resRDD = hiveContext.sql("SELECT COUNT(*) FROM  monitoringpackets")

resRDD.map(t => "Count : " + t(0) ).collect().foreach(println)


```


If no errors:

1) The files under /tmp/streaming_output should have been copied under /user/hive/zoomeet:

```
[mapr@Host ~]$ hadoop fs -ls /user/hive/zoomeet
Found 2 items
-rwxr-xr-x   3 root root          0 2017-06-02 08:52 /user/hive/zoomeet/_SUCCESS
-rwxr-xr-x   3 root root       8898 2017-06-02 08:52 /user/hive/zoomeet/part-00000

```

2) Check Hue application, at Metastore Manager ( http://[insert an IP/hostname]:8888/metastore/tables/ ), if the newly created table is present:

![ScreenShot](https://github.com/Satanette/test/blob/master/ipip.png)

3) Check PostgreSQL metastore dba, just to make sure the table is there:

```
metadb=>  select * from "TBLS";
 TBL_ID | CREATE_TIME | DB_ID | LAST_ACCESS_TIME | OWNER | RETENTION | SD_ID |     TBL_NAME      |    TBL_TYPE    | VIEW_EXPANDED_TEXT | VIEW_ORIGINAL_TEXT
--------+-------------+-------+------------------+-------+-----------+-------+-------------------+----------------+--------------------+--------------------
[====snip====]
    71 |  1496405576 |     1 |                0 | root  |         0 |    71 | monitoringpackets | EXTERNAL_TABLE |                    |
```

4) Now, go to Zoomdata, and see if you get any table ID (71 in this case),that has been created recently (you can filter it by hour and day):


![ScreenShot](https://github.com/Satanette/test/blob/master/snip_snip.png)
