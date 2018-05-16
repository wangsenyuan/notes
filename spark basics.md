1. What is Spark
  Spark is a fast, easy-to-use and flexible data processing framework. It has an advanced execution engine supporting cyclic data flow and in-memory computing. Spark can run on Hadoop, standalone or in the cloud and is capable of accessing diverse data sources including HDFS, HBase, Cassandra and others.

2. Spark v.s Hadoop
  1. Data in Memory and Data in Disk
  2. DAG flow and MapReduce

3. Key Features
  1. Spark consists of RDD’s (Resilient Distributed Datasets), which can be cached across computing nodes in a cluster.
  2. Spark supports multiple analytic tools that are used for interactive query analysis , real-time analysis and graph processing
  3. Good for Java and Scala developers as Spark imitates Scala’s collection API and functional style
  4. Single library can perform SQL, graph analytics and streaming.

4. Spark Archtecture Overview
  1. Overview
  ![Overview](./Role-of-worker-in-Spark-Architecture.png?raw=true "Archtecture Overview")

  2. Driver
    Spark Driver is the program that runs on the master node of the machine and declares transformations and actions on data RDDs. In simple terms, driver in Spark creates SparkContext, connected to a given Spark Master.

    1. Driver schedules the job execution and negotiates with cluster manager.
    2. RDDs are translated into execution graph which splits the graph into multiple stages.
    3. Driver stores metadata about RDDs and their partitions.

  2. Worker
    Worker is a distributed agent responsible for task execution. Every Spark application has its own worker process.
    1. Worker performs all data processing

  3. Cluster Manager
    Cluster manager acquires resources on Spark cluster and allocates it to Spark job. 3 types of cluster managers which a Spark application can use to allocate and deallocate physical resources like client Spark jobs, CPU memory etc. Choosing a cluster manager depends on the application as all cluster managers give different scheduling capabilities.

4. RDD
  RDD is short for Resilient Distribution Datasets – a fault-tolerant collection of operational elements that run parallel. The partitioned data in RDD is immutable and distributed;
  RDD is partitioned into logical units, that can be processed in executors parallelly;
  There are two types of operations for RDD: transformation and action;
  Transformations, like map, filter, perform on one RDD, and result in another RDD; and they are lazy, they will not executed until an Action happen;
  Actions, its excution is the result of all the dependent transformations, and bring data back into Driver; typical actions, reduce, take, count;
  RDD transformations will be lineage-ed; translated into DAG, stages; then dispatched to executors; once failure, the lineage can be used to recover the processing;