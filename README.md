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
