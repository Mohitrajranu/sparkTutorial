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
