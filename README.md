# sparkTutorial
RDD(core abstraction for working with data)--> Resilient Distributed Datasets.
Dataset is basically a collection of data it can be a list of strings,a list of integers or even a no of rows in a relational database.
RDDs can contain any type of objects, including user-defined classes.An RDD is simply a capsulation around a very large dataset.In spark all work is expressed as either creating 
a new RDDs , transforming existing RDDs, or calling operations on RDDs to compute a result.
Under the hood , spark will automatically distribute the data contained in RDDs across your cluster and parallelize the operations you perform on them
RDD offer two type of operations --> Transformation and Action
Transformations --> Apply some functions to the data in RDD to create a new RDD.
one of the most common transformations is filter which will return a new RDD with a subset of the data in the original RDD. 
JavaRDD<String> lines = sc.textFile("in/uppercase.text");
JavaRDD<String> linesWithFriday = lines.filter(line -> line.contains("Friday"));
Actions --> Compute a result based on an RDD , One of the most popular Actions is first,which returns the first element in an RDD.
JavaRDD<String> lines = sc.textFile("in/uppercase.text");
String firstLine = lines.first();
Spark RDD general workflow
. Generate initial RDDs from external data.
. Apply transformations.
. Launch Actions.

Creating RDDs 
How to create a RDD ::-> Take an existing collection in your program and pass it to SparkContext's parallelize method.
List<Integer> inputIntegers = Arrays.asList(1,2,3,4,5);
JavaRDD<Integer> integerRdd = sc.parallelize(inputIntegers);
All the elements in the collection will then be copied to form a distributed dataset that can be operated on in parallel.Very handy to create an RDD with little effort.Not practical working with large dataset as it won't fit into memory.
So the most common way to create RDD is to load RDDs from external storage by calling textFile method on SparkContext
JavaSparkContext sc = new JavaSparkContext(conf);
JavaRDD<String> lines = sc.textFile("in/uppercase.text");
The external storage is usually a distributed file system such as Amazon S3 or HDFS
There are other data sources which can be integrated with spark and used to create RDDs including JDBC,Cassandra and Elastisearch etc.

Spark JDBC driver:

https://docs.databricks.com/spark/latest/data-sources/sql-databases.html

Spark Cassandra connector:

http://www.datastax.com/dev/blog/kindling-an-introduction-to-spark-with-cassandra-part-1

Spark Elasticsearch connector:

https://www.elastic.co/guide/en/elasticsearch/hadoop/current/spark.html

Check out the full list of DevOps and Big Data courses that James and Tao teach.

https://www.level-up.one/courses/

Transformations:: Transformations are operations on RDDs , which will return a new RDD.The two most common transformations are filter and map.

Filter Transformation :: Takes in a function and returns an RDD formed by selecting those elements which pass the filter function.
can be used to remove some invalid rows to cleanup the input RDD or just get a subset of the input RDD based on the filter function.
JavaRDD<String> cleanedLines = lines.filter(line -> !line.isEmpty());

Map Transformation :: Takes in a function and passes each element in the input RDD through the function, with the result of the function being the new value of each element in the resulting RDD , it can be used to make HTTP requests to each URL in our input RDD, or it can be used to calculate the square root of each number.
JavaRDD<String> URLs = sc.textFile("in/urls.text");
URLs.map(url -> makeHttpRequest(url));

The return type of the map function is not necessary the same as its input type.
JavaRDD<String> lines = sc.textFile("in/uppercase.text");
JavaRDD<Integer> lengths = lines.map(line -> line.lengths());

TERMINOLOGY::
local[2] : 2 cores
local[*] : all available cores
local : 1 core

flatMap vs map :: map -> 1 to 1 relationship ; 1 to many relationship.

Passing functions to spark
Function1 ::-> it takes in one input and one output
Function2 : it takes in two input and one output.
FlatMap ::-> takes in one input and return zero or more ouput records from each input records.
FlatMap and Map are both different in the sense , for example in case of Map for each N input lines you will get N output lines, whereas in case of FlatMap for every N input you will get collection of N elements as output.

Set Transformation:: Set operations are performed on one RDD : - sample ; - distinct
the sample operation will create a random sample from an RDD, useful for testing purpose.
JavaRDD<T> sample(boolean withReplacement, double fraction).

The distinct transformation returns the distinct row from the input RDD.
JavaRDD<T> distinct()
The distinct transformation is expensive because it requires shuffling all the data across partitions to ensure that we receive only one copy of each element.

Set operations which are performed on two RDDs and produce one resulting RDD:
union , intersection , subtract , cartesian product.

Union Operation ::-> Union operation gives us back an RDD consisting of the data from both input RDDs.
JavaRDD<T> union(JavaRDD<T> other)
If there are any duplicates in the input RDDs,the resulting RDD of spark's union operation will contain duplicates as well.

Intersection Operation ::-> Returns the common elements which appear in both input RDDs. Intersection operation removes all duplicates including the duplicate from single RDD before returning the results. Intersection operation is quite expensive since it requires shuffling all the data across partitions to identify common elements.
JavaRDD<T> intersection(JavaRDD<T> other)

Subtract Operation ::-> Subtract operation takes in another RDD as an argument and returns us an RDD that only contains element present in the first RDD and not the second RDD.
Subtract operation requires a shuffling of all the data which could be quite expensive for large datasets.
JavaRDD<T> subtract(JavaRDD<T> other)

Cartesian Operation ::-> public <U> JavaPairRDD<T, U> cartesian(JavaRDDLike<U,?>other)
Cartesian Transformation returns all possible pairs of a and b where a is in the source RDD and b is in the other RDD.
Cartesian transformation can be very handy if we want to compare the similarity between all possible pairs.

Actions ::->> Actions are the operations which will return a final value to the driver program or persist data to an external storage system. Actions will force the evaluation of the transformations required for the RDD they were called on.

Common Actions in Spark :: collect ; count ; countByValue ; take ; saveAdTextFile ; reduce.

collect :: Collect operation retrieves the entire RDD and returns it to the driver program in the form of a regular collection or value. If you have a string RDD when you call collect action on it , you would get a list of strings. This is quite useful if your spark program has filtered RDDs down to a relatively small size and you want to deal with it locally.The entire dataset must fit in memory on a single machine as it all nneds to be copied to the driver when the collect action is called. So collect action should not be used on large datasets.

count and countByValue ::
If you just want to count how many rows in an RDD, count operation is a quick way to do that. It would return the count of the elements.
countByValue will look at unique values in the each row of your RDD and return a map of each unique value to its count.It filters out the duplicates.

take :: List<String> words = WordRDD.take(n);
take action takes n elements from an RDD , take operation can be very useful if you would like to take a peek at the RDD for unit tests and quick debugging.
take will return n elements from the RDD and it will try to reduce the number of partitions it accesses, so it is possible that the take operation could end up giving us back a biased collection and it doesn't necessary return the elements in the order we might expect.

saveAsTextFile :: can be used to write data out to distributed storage system such as HDFS or Amazon S3, or even local file system.

reduce :: reduce takes a function that oprates on two elements of the type in input RDD and returns a new element of the same type.It reduces the element of this RDD using the specified binary function.
Integer product = integerRdd.reduce((x,y) -> x*y) 
This function produces the same result when repetitively applied on the same set of RDD data and reduces to a single value.
Witn reduce operations we can perform different types of aggregations.

Important aspects abbout RDD.
RDDs are distributed : each RDD is broken into multiple pieces called partitions, and these partitions are divided across the cluster. This partitioning process is done automatically by spark , so you don't need to worry about all the details that how your data is partitioned across the cluster.

RDDs are immutable : They cannot be changed after they are created , immutablity rules out a significant set of potential problems due to updates from multiple threads at once.

RDDs are resilient : RDDs are deterministic function of their input. This plus immutablity also means the RDDs parts can be recreated at any time.
In case of any node in the cluster goes down , spark can recover the parts of the RDDs from the input and pick up from where it left off.
Spark does heavy lifting for you to make sure that RDDs are fault tolerant.

Lazy evaluation :: Transformations pn RDDs are lazily evaluated meaning that spark will not begin to execute until it sees an action.
Rather than thinking of an RDD as containing specific data, it might be better to think of each RDD as consisting of instructions on how to compute the data that we build up through transformations.
Spark uses lazy evaluation to reduce the number of passes it has to tale over our data by grouping operations together.
//nothing happens when spark sees textFile statement.
JavaRDD<String> lines = sc.textFile("in/uppercase.text");
//nothing happens when spark sees textFile statement.
JavaRDD<String> linesWithFriday = lines.filter(line -> line.startsWith("Friday"));
//Spark only starts loading the in/uppercase.text file when first() action is called on linesWithFriday.
String firstLineWithFriday = linesWithFriday.first();
//Spark scans the file only until the first line starting with Friday is detected, it doesn't even need to go through the entire file

Transformations return RDDs , whereas actions return some other data type.

Caching and Persistence.

Persistence:: sometimes we would like to call actions on the same RDD multiple times, if we do this naively , RDDs and all of its dependencies are recomputed , each time an action is called on the RDD. This can be very expensive especially for some iterative algoritms which would call actions on the same dataset many times.
If you want to reuse an RDD in multiple actions, you can also Spark to persist by calling the persist() method on the RDD.When you persist an RDD, the first time it is computed in an action it will be kept in memory across the nodes.

Different Storage level
RDD.persist(StorageLevel level)
RDD.cache()=RDD.persist(MEMORY_ONLY)

MEMORY_ONLY : Store RDD as deserialized java objects in the JVM, if the RDD doesnot fit in memory, some partitions will not be cached and will be recomputed on the fly each time they are needed , this is default level.
MEMORY_AND_DISK : Store RDD as deserialized java objects in the JVM, if the RDD doesnot fit in memory, store partitions that don't fit on disk and read them from there when they are needed.
MEMORY_ONLY_SER : Store RDD as deserialized java objects(one byte array per partition),this is generally more space efficient than deserialized objects especially when using a fast serializer but more CPU-intensive to read.
MEMORY_AND_DISK_SER(Java and Scala) : Similar to MEMORY_ONLY_SER but spill partitions that don't fit in memory to disk instead of recomputing on the fly each time they are needed 
DISK_ONLY : Store the RDD partitions only on disk.

Which Storage Level we should choose?
Sparks storage level are meant to provide different trade-offs between memory usage and cpu efficiency.
If the RDDs can fit comfortably with the default storage level MEMORY_ONLY is the ideal option. This is the most CPU-efficient option allowing operations on the RDDs to run as fast as possible.
If not try using MEMORY_ONLY_SER to make the objects much more space-efficient , but still reasonably fast to access.

What would happen if you attempt to cache too much data to fit in memory?
Spark will evict old partitions automatically using a least recently used cache policy.
For the MEMORY_ONLY storage level ,spark will re-compute these partitions the next time they are needed.
For the MEMORY_AND_DISK storage level, spark will write these partitions to disk.
In either case your spark job won't break even if you ask Spark to cache too much data.
Caching unnecessary data can cause spark to evict useful data and lead to longer re-computation time.

Spark Architecture:
Master and slave architecture:: Driver Program[SparkContext] {Master} --> Worker Node[Executor{Cache}-> Task]{Slave}

Spark Components :: 
Engine{Spark Core->[Execution Model] [The Shuffle] [Caching]}-> HDFS
UI/API{[Spark Sql] [Spark Streaming][Mlib][GraphX]}

Spark Sql::-> Provides an SQL like interface for working with structured data, built on top of spark core.
Spark Streaming::-> Provides an api for manipulating data streams that closely match the Spark Core's RDD API.

Pair RDD : A Pair RDDs is a particular type of RDD that can store key-value pairs. A lot of datasets we see in real life examples are usually key value pairs.
Tuple2<Integer, String> tuple = new Tuple2<>(12,"value");
Integer key=tuple._1();
String value=tuple._2();

Transformations on PairRDD
Pair RDDs are allowed to use all the transformations available to regular RDDs, and thus support the same functions as regular RDDs.
Since pair RDDs contain tuples, we need to pass functions that operate on tuples rather than on individual elements.
Filter Transformation :: This filter transformation that can be applied to a regular RDD can also be applied to a pair RDD.
The filter transformation takes in a function and returns a Pair RDD formed by selecting those elements which pass the filter function.

Map and MapValues transformations :: The map transformations also works for pair RDDs , it can be used to convert an RDD to another one.
But most of the time, when working with pair RDDs, we don't want to modify the keys, we just want to access the value part of our pair RDD.
Since this is a typical pattern, spark provides the mapValues function. The mapValues function will be applied to each key value pair and will convert the values
based on mapValues function, but it will not change the keys.

Aggregation:: When our dataset is described in the format of key-value pairs, it is quite common that we would like to aggregate statistics across 
all elements with the same key.
reduceByKey runs several parallels reduce operations one for each key in the dataset where each operation combines values that have the same key.
Considering  input datasets could have a huge number of keys , reduceByKey operation is not implemented as an action that returns a value to the driver program.
Instead, it returns a new RDD consisting of each key and the reduced value for that key.

groupByKey:: A common usecase for pair RDD is grouping our data by key.For example , viewing all of an account's transactions together.
If our data is already keyed in the way we want, groupByKey will group our data using the key in our pair RDD.
let's say the input pair RDD has keys of type K and values of type V , if we call group by key on the RDD, we get back a Pair RDD of type K and Iterable V.
public JavaPairRDD<K,Iterable<V>> groupByKey()

Use groupByKey only if you have to as in case of huge data  as the shuffle operation brings unnecessary data to be operated upon, which is not in case of reduceByKey as the data are reduce before as well as after shuffle operation.

sortByKey:: We can sort a pair RDD as long as there is ordering defined on the key. Once we have sorted our pair RDD any subsequent call on the sorted pair RDD to collect or save will return us ordered data.
sortByKey in reverse order :: public JavaPairRDD<K,V> sortByKey(boolean ascending)
wordsPairRdd.sortByKey(true);
custom comparator
public JavaPairRDD<K,V> sortByKey(Comparator<K> comp)
countPairRDD.sortByKey((a,b) -> Math.abs(a) - Math.abs(b))

Data Partitioning :: 
In order to reduce the amount of shuffle for groupByKey
JavaPairRDD<String, Integer> partitionedWordPairRDD = wordsPairRdd.partitionBy(new HashPartitioner(4));
partitionedWordPairRDD.persist(StorageLevel.DISK_ONLY));
partitionedWordPairRDD.groupByKey().mapToPair(word -> new Tuple2<>(word._1(),getSum(word._2()))).collect();

Operations which would benefit from partitioning :: Join,leftOuterJoin,rightOuterJoin,groupByKey,reduceByKey,combineByKey,lookup.
How reduceByKey benefits from partitioning ?
Running reduceByKey on a pre-partitioned RDD will cause all the values for each key to be computed locally on a single machine, requiring only the final, locally reduced value to be sent from each worker node back to the master.
Operations which would be affected by partitioning ?
Operations like map could cause the new RDD to forget the parent's partitioning information , as such operations could in theory , change the key of each element in the RDD.
general guidance is to prefer mapValues over map operation.

Join Operation :: Join operation allows us to join two RDDs together which is probably one of the most common operations on a Pair RDD.
Joins types: leftOuterJoin,rightOuterJoin,crossJoin,innerJoin etc.

If both RDDs have duplicate keys, join operation can dramatically expand the size of the data, it's recommended to perform a distinct or combineByKey operation to reduce the key space if possible.
Join operation may require large network transfers or even create datasets beyond our capability to handle.
Joins, in general are expensive since they require that corresponding keys from each RDD are located at the same partition so that they can be combine locally.
If the RDDs do not have known partitioners, they will need to be shuffled so that both RDDs share a partitioner and data with the same keys live in the same partitioner and data with the same keys live in the same partitions.

Shuffled Hash Join: To join data, Spark needs the data that is to be joined to live on the same partition. The default implementation of join in Spark is a shuffled hash join.
The shuffled hash join ensures that data on each partition will contain the same keys by partitioning the second dataset with the same default partitioner as the first so that the 
keys with the same hash value from both datasets are in the same partition.

The shuffle can be avoided if both RDDs have a known partitioner. If they have the same partitioner the data may be colocated, it can prevent network transfer.So it is recommended to call partitionBy on the two join RDD with the same partitioner before joining them.
val partitioner = new HashPartitioner(20)
ages.partitionBy(partitioner) 
addresses.partitionBy(partitioner)

Accumulators are variables that are used for aggregating information across the executors.For example we can calculate how many records are corrupted or count events that occur during job execution for debugging purposes.
Tasks on worker nodes cannot access the accumulators value. Accumulators are write-only variables , this allows accumulators to be implemented efficiently, without having to communicate every update.
using accumulators is not the only solution to solve these problems , It is possible to aggregate values from an entire RDD back to the driver program using actions like reduce or reduceByKey.
But sometimes we prefer a simpleway to aggregate values that in the process of transforming a RDD are generated at a different scale or granularity than that of the RDD itself.

Out of the box, Spark supports several accumulators of types such as Double and Long.Spark also allows users to define custom accumulator types and custom aggregation opeartions such as finding the maximum of the accumulated values instead of adding them.
You will need to extend the AccumulatorV2 abstract class to define your custom accumulators.

Broadcast variables: Broadcast variables allow the programmer to keep a read-only variable cached on each machine rather than shipping a copy of it with tasks.
They can be used for example to give every node , a copy of a large input dataset in an efficient manner.All broadcast variables will be kept at all the worker nodes for use in one or more spark operations.

Procedure of using broadcast variables:: Create a broadcast variable T by calling SparkContext.broadcast() on an object of type T.
The broadcast variable can be any type as long as it's serializable because the broadcast needs to be passed from the driver program to all the worker in the Spark cluster across the wire.
You can also broadcast a custom java object, just make sure the object implements the serializable interface.
The variable will be sent to each node only once and should be treated as read-only meaning updates will not be propagated to other nodes.
The value of broadcast can be accessed by calling the value method in each node.

Problems if not using broadcast variables?
Spark automatically sends all variables refrenced in our closures to the worker nodes , but spark will send it seperately for each operation.
we can potentially use the same variable in multiple parallel opeartions but spark will send it seperately for each opeartion.
This can lead to some performance issue if the size of data to transfer is significant.
  
Spark SQL:: Spark Sql introduces a tabular data abstraction called DataFrame since Spark 1.3 , a dataframe is a data abstraction or a domain specific language for working with structured and semi-structured data.
DataFrames store data in a more efficient manner than native RDDs taking advantage of their schema.
It uses the immutable in-memory,resilient,distributed and parallel capabilities of RDD, and applies a structure called schema to the data allowing Spark to manage the schema and only pass data between nodes, in a much more efficient way than using java serialization.
Unlike an RDD data is organized into named columns like a table in a relational database.
Dataset:: A dataset is a set of structured data,not necessarily a row but it could be of a particular type.Java and spark will know the type of the data in dataset at compile time.
Dataset takes on two distinct APIs characteristics a strongly-typed api and an untyped api.
Consider DataFrame as untyped view of a Dataset,which is a dataset of row where a row is generic untyped JVM object.
Dataset by contrast is a collection of strongly typed jvm object.
Catalyst Optimizer: Spark Sql uses an optimizer called Catalyst to optimize all the queries written both in Spark SQL and DataFrame DSL.
This optimizer makes queries run much faster than their RDD counterparts.
The catalyst is a modular library which is built as a rule-based system. Each rule in the framework focuses on the specific optimization.For example rule like constantfolding focuses on removing constant expression from the query.
SparkSql catalyst optimizer can do more of the heavy-lifting for us to optimize the join performance , using spark sql join we have to give up some of our control.
For example Spark Sql can sometimes push down or re-order operations to make the joins more efficient.The downside is that we don't have controls over the partitioner for Datasets.
so we can't manually avoid shuffles as we did with core spark joins.

The standard SQL join types are supported by spark sql can be specified as the JoinType when performin join.
public Dataset<Row> join(Dataset<?> right,Column joinExprs,String joinType)
markerSpaceDataset.join(postCodeDataset,col("postcode"),left_outer);
Spark SQL join types:
inner,outer,left_outer,right_outer,left_semi.

Row:: Row objects represent records inside dataset and are simply fixed-length array of fields. Row objects have a number of getter functions to obtain the value of each field given its index.The get method takes a column number and returns us an object type; we are responsible for casting the object to the correct type.
Object field7 = row.get(7);
For primitive and boxed types there is a get type method which returns the value of that type.
long field1=row.getLong(1);
boolean field2=row.getBoolean(2);

Encoders:: Dataset API has the concept of encoders which translate between JVM representations which are Java objects and Spark's internal binary format.
Spark has built-in encoders such as integer encoder or long encoder which are very advanced in that they generate bytecode to interact with off-heap data and provide on-demand
access to individual attributes without having to de-serialize an entire object.
Encoders.INT();
Encoders.LONG();
The encoder is used to convert a JVM object of type response to and from the internal spark SQL representation.
Supported field types:
all primitive types(int,long etc)
all boxed types(Integer,Long etc)
list and arrays
Not supported field types :-> optional fields and map.
The class must have a setter method with zero arg-constructor

Both dataset and dataframes are built on top of RDD. RDD is the primary user facing API in Spark. At the core , RDD is an immutable distributed collection of elements of your data partitioned across nodes in your cluster that can be operated in parallel with low-level api that offers transformations and actions.
using RDDs when:
low-level transformation,actions and control on our dataset are needed. Unstructured data , such as media streams of text.
optimization and performance benefits available with Datasets are not needed , need to manipulate our data with functional programming constructs than domain specific expressions.
Use Datasets when:
Rich semantics,high-level abstractions and domain specific apis are needed. Our processing requires aggregation,averages,sum,sql queries and columnar access on semi-structured data.
We want a higher degree of type-safety at compile time,typed JVM objects.

Spark SQL has some built-in optimzations such as predicate push-down which allows Spark SQL to move some parts of our query down to the engine we are querying.
Caching : If you find yourself performing some queries ortransformations on a dataset repeatedly we should consider caching the dataset which can be done by calling the cache method on the dataset.
responseDataset.cache();
When caching a dataset , Spark Sql uses an in-memory columnar storage for the dataset.
If our subsequent queries depend only on subsets of the data, spark sql will minimize the data read and automatically tune compression to reduce garbage collection pressure and memory usage.

Configure Spark Properties
spark.sql.codegen
SparkSession session=SparkSession.builder().config("spark.sql.codegen",false).getOrCreate();
It will ask spark sql to compile each query to Java byte code before executing it.
This codegen option could make long queries or repeated queries substantially faster, as Spark generates specific code to run them.For short queries or some non-repeated ad-hoc queries this option could add unnecessary overhead as Spark has to run a compiler for each query.
It's recommended to use codegen option for workflows which involves large queries or with the same repeated query.

spark.sql.inMemoryColumnarStorage.batchSize
When caching dataset, Spark groups together the records in batches of the size given by this option and compresses each batch.
The default batch size is 1000.
SparkSession session=SparkSession.builder().config("spark.sql.inMemoryColumnarStorage.batchSize",10000).getOrCreate();
Having a larger batch size can improve memory utilization and compression.
A batch with a large number of records might be hard to build up in memory and can lead to an OutOfMemoryError.

Cluster Manager: The cluster manager is a pluggable component in spark. Spark is packaged with built-in cluster manager called the standalone cluster manager.
There are other types of Spark Manager master such as :-
Hadoop Yarn :- A resource management and scheduling tool for a Hadoop MapReduce cluster.
Apache Mesos :- Centralized fault-tolerant cluster manager and global resource manager for your entire data center.
The cluster manager abstracts away the uderlying cluster environment so that you can use the same unified high-level spark api to write spark program which can run on different clusters.
Spark provides a single script you can use to submit your application to it called spark-submit.

package spark application && use spark-submit.
Steps:-
1. Download Spark distribution to our local box.
2. Export our Spark application to a jar file.
./gradlew jar
./gradlew clean jar
3. Submit our application to our local spark cluster through spark-submit script. 
./spark-2.1.0-bin-hadoop2.7/bin/spark-submit StackOverFlowSurvey.jar

Running Spark Applications on a cluster.

The user submits an application using spark-submit. spark-submit launches the driver program and invokes the main method specified by the user.
The driver program contacts the cluster manager to ask for resources to start executors.The cluster manager launches executors on behalf of the driver program.
The driver process runs through the user application. Based on the RDD or dataset operations in the program, the driver sends work to executors in form of tasks
Tasks are run on executor processes to compute and save results.
If the driver's main method exits or it calls SparkContext.stop() , it will terminate the executors.
Spark submit options:
./bin/spark-submit --executor-memory 20G --total-executor-cores 100 abc.jar

Running on Amazon EMR.
Amazon EMR cluster provides a managed Hadoop framework that makes it easy,fast and cost-effective to process vast amounts of data across dynamically scalable Amazon EC2 instances.
We are going to run our spark application on top of the Hadoop cluster and we will put the input data source into the S3.
S3 doesn't allows space in file name
s3n://bucket-name/filename -> input option
upload the jar file as well to s3
SparkSession session = SparkSession.builder().appName("StackOverFlowSurvey").getOrCreate();--> remove the master{local} option 

Now option the emr session from putty using ssh
copy the jar file to the linux instance.
aws s3 cp s3://bucket-name/jarfile .
by default the spark-submit script is added to the classpath of emr-instance.
spark-submit StackOverFlowSurvey.jar