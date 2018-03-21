Run Spark on Riak CS
====================

[Spark](http://spark.apache.org/) is fast distributed processing framework, by keeping intermediate data or input data on memory, as known as RDD. Spark not only runs on top of HDFS but also on top of S3 - then it should work on top of [Riak CS](http://docs.basho.com/riakcs/latest/) . This document shows result of connectivity test.

## Test Spark with Local Filesystem

So easy as ABC. Get whatever Spark release from [download page](http://spark.apache.org/downloads.html) . ::

 > tar xzf spark-1.0.0-bin-hadoop2.tgz 
 > cd spark-1.0.0-bin-hadoop2/

 > bin/spark-shell
 Spark assembly has been built with Hive, including Datanucleus jars on classpath
 14/07/01 23:32:59 INFO SecurityManager: Using Spark's default log4j profile: org/apache/spark/log4j-defaults.properties
 14/07/01 23:32:59 INFO SecurityManager: Changing view acls to: kuenishi
 14/07/01 23:32:59 INFO SecurityManager: SecurityManager: authentication disabled; ui acls disabled; users with view permissions: Set(kuenishi)
 14/07/01 23:32:59 INFO HttpServer: Starting HTTP Server
 Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 1.0.0
      /_/

 Using Scala version 2.10.4 (Java HotSpot(TM) 64-Bit Server VM, Java 1.7.0_25)
 Type in expressions to have them evaluated.
 Type :help for more information.
 (snip)
 14/07/01 23:33:19 INFO Executor: Using REPL class URI: http://172.17.42.1:49255
 14/07/01 23:33:19 INFO SparkILoop: Created spark context..
 Spark context available as sc.

 scala>


That's easy. And run some test query as in [quickstart](http://spark.apache.org/docs/latest/quick-start.html). ::


 val textFile=sc.textFile("README.md")
 14/07/01 23:34:00 INFO MemoryStore: ensureFreeSpace(138803) called with curMem=0, maxMem=308713881
 14/07/01 23:34:00 INFO MemoryStore: Block broadcast_0 stored as values to memory (estimated size 135.5 KB, free 294.3 MB)
 textFile: org.apache.spark.rdd.RDD[String] = MappedRDD[1] at textFile at <console>:12

 scala> textFile.count()
 14/07/01 23:37:55 INFO FileInputFormat: Total input paths to process : 1
 14/07/01 23:37:55 INFO SparkContext: Starting job: count at <console>:15
 14/07/01 23:37:55 INFO DAGScheduler: Got job 0 (count at <console>:15) with 2 output partitions (allowLocal=false)
 (snip)
 14/07/01 23:37:56 INFO TaskSchedulerImpl: Removed TaskSet 0.0, whose tasks have all completed, from pool 
 14/07/01 23:37:56 INFO SparkContext: Job finished: count at <console>:15, took 0.316897949 s
 res0: Long = 127

 scala> 


That's also easy, right? In the next chapter let's try run the same query against the file in Riak CS.


## Setup Riak CS

Riak CS version is 1.5.0-beta1. The package is not officially
released. But you can build by yourself, by starting from
[github repository](https://github.com/basho/riak_cs).

## Let Spark find Riak CS

Suppose http://localhost:8080 is Riak CS and its name is
s3.amazonaws.com as usual. Spark also uses `jets3t` as S3 connector,
thus putting `jets3t.properties` into `$SPARK_HOME/conf` and setting
environment variables will be sufficient. ::


 > cp $SPARK_HOME
 > cp your/jets3t.properties conf
 > cat conf/jets3t.properties
 s3service.https-only=false
 #s3service.s3-endpoint=localhost
 #s3service.s3-endpoint-http-port=8080
 #s3service.s3-endpoint-https-port=8080
 #s3service.disable-dns-buckets=true

 httpclient.proxy-autodetect=false
 httpclient.proxy-host=localhost
 httpclient.proxy-port=8080
 httpclient.retry-max=11
 > export AWS_ACCESS_KEY_ID=KI_-C956C0F0GNFMKMW4
 > export AWS_SECRET_ACCESS_KEY=c7Xb0rzhErBFH8JrHcHTMSfNnqAuxdRToI94zA==
 > s3cmd put README.md s3://test/
 ../hadoop/spark-1.0.0-bin-hadoop2/README.md -> s3://test/README.md  [1 of 1]
  4221 of 4221   100% in    0s    27.04 kB/s  done
 > bin/spark-shell


## Just do it

And then try exactly the same example as above, and make sure the
result is same. ::


 scala> val data = sc.textFile("s3n://test/README.md")
 14/07/02 01:38:14 INFO MemoryStore: ensureFreeSpace(139483) called with curMem=0, maxMem=308713881
 14/07/02 01:38:14 INFO MemoryStore: Block broadcast_0 stored as values to memory (estimated size 136.2 KB, free 294.3 MB)
 data: org.apache.spark.rdd.RDD[String] = MappedRDD[1] at textFile at <console>:12

 scala> data.count()
 14/07/02 01:38:17 INFO RestUtils: Using Proxy: localhost:8080
 14/07/02 01:38:17 INFO FileInputFormat: Total input paths to process : 1
 14/07/02 01:38:17 INFO SparkContext: Starting job: count at <console>:15
 14/07/02 01:38:17 INFO DAGScheduler: Got job 0 (count at <console>:15) with 2 output partitions (allowLocal=false)
 14/07/02 01:38:17 INFO DAGScheduler: Final stage: Stage 0(count at <console>:15)
 14/07/02 01:38:17 INFO DAGScheduler: Parents of final stage: List()
 14/07/02 01:38:17 INFO DAGScheduler: Missing parents: List()
 14/07/02 01:38:17 INFO DAGScheduler: Submitting Stage 0 (MappedRDD[1] at textFile at <console>:12), which has no missing parents
 14/07/02 01:38:17 INFO DAGScheduler: Submitting 2 missing tasks from Stage 0 (MappedRDD[1] at textFile at <console>:12)
 14/07/02 01:38:17 INFO TaskSchedulerImpl: Adding task set 0.0 with 2 tasks
 14/07/02 01:38:17 INFO TaskSetManager: Starting task 0.0:0 as TID 0 on executor localhost: localhost (PROCESS_LOCAL)
 14/07/02 01:38:17 INFO TaskSetManager: Serialized task 0.0:0 as 1696 bytes in 2 ms
 14/07/02 01:38:17 INFO TaskSetManager: Starting task 0.0:1 as TID 1 on executor localhost: localhost (PROCESS_LOCAL)
 14/07/02 01:38:17 INFO TaskSetManager: Serialized task 0.0:1 as 1696 bytes in 12 ms
 14/07/02 01:38:17 INFO Executor: Running task ID 1
 14/07/02 01:38:17 INFO Executor: Running task ID 0
 14/07/02 01:38:17 INFO BlockManager: Found block broadcast_0 locally
 14/07/02 01:38:17 INFO BlockManager: Found block broadcast_0 locally
 14/07/02 01:38:17 INFO HadoopRDD: Input split: s3n://test/README.md:0+2110
 14/07/02 01:38:17 INFO HadoopRDD: Input split: s3n://test/README.md:2110+2111
 14/07/02 01:38:17 INFO deprecation: mapred.tip.id is deprecated. Instead, use mapreduce.task.id
 14/07/02 01:38:17 INFO deprecation: mapred.task.id is deprecated. Instead, use mapreduce.task.attempt.id
 14/07/02 01:38:17 INFO deprecation: mapred.task.is.map is deprecated. Instead, use mapreduce.task.ismap
 14/07/02 01:38:17 INFO deprecation: mapred.task.partition is deprecated. Instead, use mapreduce.task.partition
 14/07/02 01:38:17 INFO deprecation: mapred.job.id is deprecated. Instead, use mapreduce.job.id
 14/07/02 01:38:17 INFO deprecation: mapred.tip.id is deprecated. Instead, use mapreduce.task.id
 14/07/02 01:38:17 INFO NativeS3FileSystem: Opening 's3n://test/README.md' for reading
 14/07/02 01:38:17 INFO NativeS3FileSystem: Opening 's3n://test/README.md' for reading
 14/07/02 01:38:17 INFO NativeS3FileSystem: Opening key 'README.md' for reading at position '2110'
 14/07/02 01:38:17 INFO NativeS3FileSystem: Opening key 'README.md' for reading at position '0'
 14/07/02 01:38:18 INFO Executor: Serialized size of result for 1 is 597
 14/07/02 01:38:18 INFO Executor: Serialized size of result for 0 is 597
 14/07/02 01:38:18 INFO Executor: Sending result for 1 directly to driver
 14/07/02 01:38:18 INFO Executor: Sending result for 0 directly to driver
 14/07/02 01:38:18 INFO Executor: Finished task ID 0
 14/07/02 01:38:18 INFO Executor: Finished task ID 1
 14/07/02 01:38:18 INFO TaskSetManager: Finished TID 0 in 237 ms on localhost (progress: 1/2)
 14/07/02 01:38:18 INFO DAGScheduler: Completed ResultTask(0, 0)
 14/07/02 01:38:18 INFO DAGScheduler: Completed ResultTask(0, 1)
 14/07/02 01:38:18 INFO DAGScheduler: Stage 0 (count at <console>:15) finished in 0.329 s
 14/07/02 01:38:18 INFO TaskSetManager: Finished TID 1 in 247 ms on localhost (progress: 2/2)
 14/07/02 01:38:18 INFO TaskSchedulerImpl: Removed TaskSet 0.0, whose tasks have all completed, from pool 
 14/07/02 01:38:18 INFO SparkContext: Job finished: count at <console>:15, took 0.444235349 s
 res0: Long = 127

 scala>


Here it works.

- Spark on S3 (Japanese) http://itsneatlife.blogspot.jp/2014/01/sparks3.html
