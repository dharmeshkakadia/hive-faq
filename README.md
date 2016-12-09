# hive-notes

These are some of the common patterns I have found useful while solving Hive deployements managed by Ambari (such as HDInsight).

## My query fails

### What is the error?

### 

## My query is slow


## FAQ

### How do I check my cluster capacity?

### How many containers I can run in parallel?

### How do I check DAG counters for a query?
Go to Tez View. Find the DAG and click on Dag counters tab.

### How do I check that amount of data being read/writted/shuffled by a query?
Go to Dag counters and look for HDFS_BYTES_READ/HDFS_BYTES_WRITTEN/SHUFFLE_BYTES.

### How do I download all YARN container logs?
``yarn logs -applicationId <application_id> > logs``

### How do I download YARN container logs for a particular container?
``yarn logs -applicationId <application_id> -containerId <container_id> > containerlogs``

### How do I download YARN container logs for all Application Master (AM) container?
``yarn logs -applicationId <application_id> -am ALL > amlogs``

### How do I download YARN container logs for only the latest Application Master (AM) container?
``yarn logs -applicationId <application_id> -am -1 > latestamlogs``

### How do I download YARN container logs for first 2 Application Master (AM) container?
``yarn logs -applicationId <application_id> -am 1,2 > first2amlogs``

### Where is hive client logs?
``/tmp/<username>/hive.log``

### Where is hive metastore logs?
``/var/log/hive/hivemetastore.log``

### Where is hiveserver logs?
``/var/log/hive/hiveserver2.log``

### How to I start hive shell with debug logging enabled on console?
hive -hiveconf hive.root.logger=ALL,console

### How do I use tez engine?
``set hive.execution.engine=tez``

### How do I download tez DAG data?
``hadoop jar /usr/hdp/current/tez-client/tez-history-parser-*.jar org.apache.tez.history.ATSImportTool -downloadDir . -dagId <DagId>``

### How do I get CrticalPath for a Tez DAG?
``hadoop jar /usr/hdp/current/tez-client/tez-job-analyzer-*.jar CriticalPath --saveResults --dagId <DagId> --eventFileName <DagData.zip>``

### What are the other analyzers available for Tez DAG?
You can list all the available analyzers with ``hadoop jar /usr/hdp/current/tez-client/tez-job-analyzer-*.jar``. Currently it has the following analyzers available:

  * ContainerReuseAnalyzer: Print container reuse details in a DAG
  * CriticalPath: Find the critical path of a DAG
  * LocalityAnalyzer: Print locality details in a DAG
  * ShuffleTimeAnalyzer: Analyze the shuffle time details in a DAG
  * SkewAnalyzer: Analyze the skew details in a DAG
  * SlowNodeAnalyzer: Print node details in a DAG
  * SlowTaskIdentifier: Print slow task details in a DAG
  * SlowestVertexAnalyzer: Print slowest vertex details in a DAG
  * SpillAnalyzer: Print spill details in a DAG
  * TaskAssignmentAnalyzer: Print task-to-node assignment details of a DAG
  * TaskConcurrencyAnalyzer: Print the task concurrency details in a DAG
  * VertexLevelCriticalPathAnalyzer: Find critical path at vertex level in a DAG

### How do I kill an application?
``yarn application -kill <application_id>``

This will kill any query that is being run as part of the Tez session also.

### How do I check currently running application on the cluster?
``yarn top``

### How do I specify the database while starting hive?
``hive -database <databaseName>``

### How do I specify a config or variable while starting hive?
``hive -hiveconf a=b``

### How do I disable mapjoin?
``set hive.auto.convert.join=false``

### How do I decide which joins are converted to mapjoins?
set ``hive.auto.convert.join.noconditionaltask.size`` to value of the largest join you want to convert to mapjoin. For example, ``set hive.auto.convert.join.noconditionaltask.size=1000000`` will convert all joins to mapjoins where hive "estimates" the sum of smallest n-1 tables in n-join is less then 1MB.

### How do I decide the right container size for my workload?
Magic ! 

### My cluster disk space is filled what do I do?
Go to Ambari -> HDFS. Check the values of ``Disk Usage (DFS Used)`` and ``Disk Usage (Non DFS Used)`` and decide which one is taking most of the space on the cluster. If most data is used for,

1. DFS : You have lot of data on HDFS, which is consuming disk space. The solution is to delete the data from the HDFS that you no longer require. FOr example, you can use ``hdfs dfs -rm -r -skipTrash hdfs://mycluster/tmp`` to remove the /tmp HDFS directory.
2. Non DFS : You have lot of intermediate job data that is consuming the disk space. You have to kill the application(s) responsible for writing lots of data. You can check the values of FILE_BYTES_WRITTEN counter to infer which is the bad application(s). Once you have killed the bad application(s), the cluster should become healthy in few minutes. If you re-run the hive query without fixing it to not to produce large intermediate data, the cluster will again end up in the same situation. One of the very common example of queries which result into a cross join. If you do ``explain`` of the query, the cross join would be flagged as [WARNING].

In both cases, using a larger cluster or cluster with more disk spaces on nodes can help mitigate the problem for short time.

### How do I use connect automatically when starting beeline?
``beeline -u "jdbc:hive2://localhost:10001/;transportMode=http" -n <usernmae> -p <password>``

### How do I connect to a different database(mydb) when starting beeline?
``beeline -u "jdbc:hive2://localhost:10001/mydb;transportMode=http" -n <usernmae> -p <password>``
