# Notes
http://highscalability.com/

1. DataNodes send block report to the NameNode
2. Data Node also have metadata of data blocks. It maintains CRC tovalidate blocks.
3. Updates are not allowed in hdfs
4. Small file size limitaion in hdfs
5. SecondaryNameNode does check pointing
6. Changes to the metadata are stored in edits which are transfred to the SNN.
7. fs image + edits => coz append is faster than merge
8. SNN created new fs image by merging old fs image + edits =  new fs image.
9. The merging of fs image can happen based on time, size of transaction
10. The SNN does the merging and send the new fs image to NameNode.
11. The NFS is shared among the NameNode and SNN.
12. If there is no SNN there will be no checkpointing and the merging will happen on the NAmeNode when it reboots but the boot time will increase.
    That is The MTR mean time to recovery will increase.
13. Data Pipeline = Data Ingestion(Sqoop,Flume, Kafka) -> Data Storage(HDFS, HBase) -> Data Processing(Spark, MR, STORM)
    -> Data Analysis(Impala, Hive, Pig) -> Data Exploration(Solr, Elasticsearch) -> Data Visualizaton(Kibana, BI tools)
14. MRv1 problems:
    Lot of responsibility on NameNode
	Job tracker is the SPOF
    Unfair distribution of resources
    Restricted to Java
    4000 Node limitation
15. YARN
    Resource Manager
    Node Manager
    Application Master => Map tasks, Reduce tasks report to App Master	
16. MRv2 = Application Master which could be anything like scala, c++, etc..	
17. Filtering : select * from products where price > 100    => only Map jobs
18. select count(*) from products where price > 100    => end to end Map Reduce jobs
19. HIVE, Pig,Sqoop is Map Reduce based, so internally invokes MR jobs unlike Impala which use MPP, Llama.
20. By default   1 input split = 1 block = 1 mapper = 1 JVM
21. We can configure more mappers  by saying 1 input split = 10 mappers or vice versa
22. While submitting the job we can specify the split size.
23. RecordReader passes data from Input Split to Mapper
24. Mapper is responsible for processing 1 Input split and map() of Mapper will be called for each record.
25. The shuffle and sort is done by Node Manager   https://farm3.static.flickr.com/2275/3529146683_c8247ff6db_o.png
26. partitioning = key.hashcode % no Of Reducers
27. Shuffle and Sort is an auxillary service specific to MR or spark.
28. HistoryMaster daemon can be used for logs monitoring. Log files are aggregated on history server
    yarn logs -applicationId application_1482757250148_0003
29. Data Skewness, Speculative Execution in MapReduce
30. Speculative Exceution is configurable.
31. Job recovery parameter can be turned on to resume task in case of failures from the failure point.
32. Distributed Cache
33. NullWritable is absence of value.
34. For sorting a file with a list of numbers just write  context.write(value, NullWritable)
35. job.setNumReduceTasks(2)
36. job.setPartitioner(MyPartitioner.class)
37. Concept of Queueing/Pooling is there for cluster capacity and fair scheduling.
38. YARN does a fair scheduling of resources according to priority.
39. For custom data type keys and values write your own by implementing Writable,WritableComparable interfaces.
40. Writable interface has  methods read(), write().
41. LineRecordReader code
42. Reduce side join for   dataset1[cityId, cityName]  dataset2[empId, empName, cityId] 
    from mapper with key as cityId  for both datasets they will reach to same reducer.
43. For Map side join we can use distributed cache to get the data at map side.
    http://stackoverflow.com/questions/11803450/join-of-two-datasets-in-mapreduce-hadoop
44. HIVE(a batch framework unlike Impala which is interactive) Runs on MR/Spark
45. Steps: 
    a. Parse
    b. Plan Execution
    c. Optimization
    d. Submit
    e. Monitor	
46. HIVE is schema on read(no schema validation on write time but it is validated on read time) unlike RDBMS which is schema on write(need schema before write).
47. To connect to HIVE use >hive or >beeline
48. You have to specify the HDFS location of the folder where your table specific data is located and row delimiter 
    and a field delimiter which by default is  	Ctrl A
49. Data Types: INT, DOUBLE, FLOAT, TINYINT, DECIMAL
50. Default location is /user/hive/warehouse
60. Database is parent dir
61. 2 Types of tablesin HIVE:
    Managed: Lifecycle of data is managed by HIVE(on drop table  metadata and data on hdfs both are deleted.)
	Special: Lifecycle of data is not managed by HIVE(on drop table only metadata is deleted.)
62. We can do INSERT into table tbl (SELECT .....)
63. We can do CREATE table tbl AS(SELECT .....)
64. Connecting to beeline:  beeline -u jdbc:hive2://localhost:10000 -n root    or  beeline -u jdbc:hive2://localhost:10000 -n cloudera -p cloudera
65. explain create table temp1 as select * from temp_batting;
66. DESCRIBE FORMATTED table_name;
67. HIVE partitiong
    Dynamic Partitioning: is on distinct values of column
	Static  Partitioning: 
68. CREATE EXTERNAL TABLE customers_by_state (
      cust_id int,
	  fname string,
	  lname string,
	  address string,
	  city string,
	  zipcode string)	
	  PARTITIONED BY (state string)
	  ROW FORMAT DELIMITED
	  FIELDS TERMINATE DBY '\t'
	  LOCATION 'dualcore/customers_by_state'
	  STORED AS TEXTFILE;
69.	INSERT OVERWRITE TABLE customers_by_state  PARTITION(state)  SELECT cust_id, fname, lname, address, city, zipcode, state FROM customers;
70. ALTER TABLE call_logs ADD PARTITION (call_date='2014-10-11')	
71. LOAD DATA LOCAL INPATH '/tmp/pv_2008-06-08_us.txt' INTO TABLE page_view PARTITION(date='2008-06-08', country='US', type); 

72. Sequence File has problem of interoperability as it is Java based.
73. Sequece File does not support Schema evolution.
74. Avro is interoperable, support schema evolution, self describing. The .avro files contains the schema info as well.
75. >avro-tools tojson  abc.avro
    >avro-tools getschema  abc.avro
76. In HIVE you can specify STORED AS AVRO TBLPROPERTIES ("avro.schema.hdfs.url") while creating table.
77. CREATE TABLE order_details_avro()
    STORED AS AVRO
    TBLPROPERTIES ();	
78. We can import data as avro format directly from database using sqoop;

::SPARK::

1. Spark Operation has 2 main types:
   Transformation => map
   Action => count(), saveAsTextFile
2. Storm gives you latency in millis for streaming data whereas Spark Streaming gives you latency in seconds.
3. Storm is at leaset once semantics while Spark Streaming is exactly once.   
4. sc.textFile("hdfs://localhost:8020/user/cloudera/sample.txt")
5. sc.textFile(file:/home/training/sample.txt)
6. All computations on RDDs are transformations or actions.
7. RDD{
      map(fn(x)){
	     for each x
		 emit fn(x)
	  }
   }
8. val rdd1 = sc.wholeTextFiles(color_directory)
   val rdd2 = rdd1.map( p=>
     JSON.parseFull(p._2).getInstanceOf[Map[String, String]]
   )
   
   for(data <- rdd2.take(2))
         println(data.getOrElse("colorName", null))
9. groupByKey() vs reduceByKey()  : More data is transfered in groupByKey()		 
10. spark-shell --executor-memory 2048m
11. val datacount = sc.textFile("access_log.txt").flatMap(line => line.split(' ')).map(word=>(word,1)).reduceByKey((v1,v2)=>v1+v2);
12. :paste mode of spark-shell
13. def getActivations(xmlString: String): Iterator[Node]= {val nodes = XML.loadString(xmlString) \\ "activation"; nodes.toIterator }
    def getModel(activation:Node):String = {(activation\"model").text}
    def getModel(activation:Node):String = {(activation\"account-number").text}
    def getActivations(xmlString: String): Iterator[Node]= {val nodes = XML.loadString(xmlString) \\ "activation"; nodes.toIterator }
14. Multi RDD transformations
15. What if your data contains delimiters	
16. Executors => cache data, run jvm on worker m/cs, run logic
17. Driver => JVm onspark-shell m/cs, creating sparkcontext:entry into spak api, stores DAG
18. RDD (data abstraction) => files, sc.paralleize, on transformations
19. Spark execution Modes: 
    Local[noOfThreads]   e.g spark-submit --master local
	YARN -> yarn-client, yarn-cluster      for e.g spark-submit --master yarn-cluster
	StandAlone
	Mesos
20. Default Spark	UI: localhost:4040
21. noOfPartitions = 2;  sc.textFile("filename", noOfPartitions)
22. To get Plan: println(data.toDebugString)
23. rdd  cache()  vs persist() In persist u can give option for disk, memory, memory and disk  LRU is default strategy.
24. Broadcast Variables, Accumulators
25. sqlContext is compatible with hiveContext
26. The file format & compression algo should supoort splittability as each block corresponds to a mapper.
27. avro-tools getschema movies.avro
28. avro-tools tojson movies.avro
29. The formats like avro and parquet are binary encoded.
30. While importing data from sqoop we can give  --as-avrofile    to save data in avro format.
31. While creating table in Hive using "CRETAE As TABLE"  we can give   'as avro' to save data in avro format.
32. parquet is the default format of Spark SQL.
33. rdd = dataFrame.rdd
34. dataFrame = sqlContext.createDataFrame(rdd, schema) df.show() // schema = StructType([StructField, StructField])
35. beeline -u jdbc:hive2://localhost:10000/default -n root -p cloudera
36. sqlContext.sql("CREATE TABLE IF NOT EXISTS kblist(id INT, name STRING, age INT) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' LINES TERMINATED BY '\n'")
37. Spark Streaming provides latency in second ( > 500 ms),for sub second latency use Apache Storm.
38. Spark Streamning is exactly once while Apache Storm is aleast once semantics.
39. Stateful Streaming:
    Times Based Slicing
	Window based
	updateStateByKey
40. Machine Learning:
    Classification
    Clustering
    Collaborative filtering.
	
	===============================HABSE ==========================
1. HBASE has small block size of 64KB therefore it is good for random read and write.
2. Graph DB => Neo4j	
3. Polyglot Persistence  https://www.youtube.com/watch?v=tLUvWwPRIbU   
4. In terms of CAP theorom HBASE is CP  while Cassandra is AP (Cassandra works ongossip protocol while Hbas eis master server arch.)
5. In HBASE we should have limited no of column families (max three.)
6. We can alter HABSE table to add column family.
7. Roles of Zookeeper:
   a. Distributes synchronization
   b. Distributed coordination
   c. Shared data
   d. Common State
   
8.  >zookeeper-client  ls habse/
9. It is Highly Available (3, 5 or 7 instances)
10. Zookeeper Quorom (consensus based protocol)
11. flush 'table_name' will write the in memory data to Hfile on disk
12. hbase.hregion.flush.size = 128MB   
13. MIN_VERSIONS  of the column family will allow the data to survive the ttl expiry.
14. HBASE can be entered into maintenance mode through console during which major compaction(all hflies are merged into single) will occur.
15. NTP daemon should be installed on all m/cs for distributed computing especially zookeeper ones to keep the time in sync.
16. If you have 3 versions in memory and you flush and then if you have 2 versions in memory and you flush then even if you have min_version=3 
    you will have 5 versions in all. This will be corrected during compaction.
17. sudo service hbase-master status/restart   sudo service hbase-regionserver status/restart
18. Apache Phoenix provide sql like support on Hbase and secondary indexing features built in.
19. Tall vs Wide table      
    Tall:  |user|   email1  email2  email3  email4 .......
	        
	Wide:  |user   emailId| email1
	       |user   emailId| email2  
    For Scan we would like Tall table.
20. To read hbase.apache.org   and  "Architecting HBase" applications on safari.	
21. MVCC for wide table
