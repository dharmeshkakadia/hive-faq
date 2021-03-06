# Hive Frequently Asked Questions

These are some of the common patterns and questions I have found useful while solving Hive deployements managed by Ambari (such as HDInsight).

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
``hive -hiveconf hive.root.logger=ALL,console``

### How do I use tez engine?
``set hive.execution.engine=tez``

### How do I download tez DAG data?
From commandline:

``hadoop jar /usr/hdp/current/tez-client/tez-history-parser-*.jar org.apache.tez.history.ATSImportTool -downloadDir . -dagId <DagId>``

From Ambari tez view:

Go to Ambari --> Go to Tez view (hidden under tiles icon in upper right corner) --> Click on the dag you are interested in --> Click on Download data.

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
From the command line:

``yarn application -kill <application_id>``

From the Yarn UI:

Go to Ambari --> Yarn UI --> click on application that you want to kill --> Click on "kill application" towards top left.

This will kill any query that is being run as part of the Tez session also.

### How do I check currently running application on the cluster?
``yarn top``

### How to I export hive metastore and import it on new cluster?
Run the following where you want to export the metastore:
```
for d in `hive -e "show databases"`; do echo "create database $d; use $d;" >> alltables.sql ; for t in `hive --database $d -e "show tables"` ; do ddl=`hive --database $d -e "show create table $t"`; echo "$ddl ;" >> alltables.sql ; echo "$ddl" | grep -q "PARTITIONED\s*BY" && echo "MSCK REPAIR TABLE $t ;" >> alltables.sql ; done; done
```

This will generate a ``allatables.sql`` file. Copy this file to new cluster and run the following on new cluster:
```
hive -f alltables.sql
```
This assumes that data path on new cluster are same as on old. If not, you can manually edit the generated ``alltables.sql`` file to reflect any changes.

### How do I specify the database while starting hive?
``hive -database <databaseName>``

### How do I specify a config or variable while starting hive?
``hive -hiveconf a=b``

### How do I list all columns and their types for a table?
Within hive/beeline shell run ``desc <tablename>``

### How do I list all properties of a table?
Within hive/beeline shell run ``desc extended <tablename>``. The output includes all the column names, their types, data format, compression properties, table statistics, columns statistics etc. 

### How do I disable mapjoin?
``set hive.auto.convert.join=false``

### How do I decide which joins are converted to mapjoins?
set ``hive.auto.convert.join.noconditionaltask.size`` to value of the largest join you want to convert to mapjoin. For example, ``set hive.auto.convert.join.noconditionaltask.size=1000000`` will convert all joins to mapjoins where hive "estimates" the sum of smallest n-1 tables in n-join is less then 1MB.

### How do I decide the right container size for my workload?
Magic ! 

### Does hive support Transactions?
Yes. Look [here](https://cwiki.apache.org/confluence/display/Hive/Hive+Transactions)

### How do I change the container size for hive on tez?
Set ``hive.tez.container.size`` to the desired value in MB. While changing the container size you should make sure that values of ``hive.tez.java.opts`` (0.8\*hive.tez.container.size) and ``tez.runtime.io.sort.mb`` (0.4\*hive.tez.container.size) are adjusted accordingly. 

### How do I turn off vectorization in hive?
set ``hive.vectorized.execution.enabled`` to ``false``. Hive vectorization has been source of many many errors and usually the error trace when the query fails, has the vectorization operator name.

### My cluster disk space is filled what do I do?
Go to Ambari -> HDFS. Check the values of ``Disk Usage (DFS Used)`` and ``Disk Usage (Non DFS Used)`` and decide which one is taking most of the space on the cluster. If most data is used for,

1. DFS : You have lot of data on HDFS, which is consuming disk space. The solution is to delete the data from the HDFS that you no longer require. FOr example, you can use ``hdfs dfs -rm -r -skipTrash hdfs://mycluster/tmp`` to remove the /tmp HDFS directory.
2. Non DFS : You have lot of intermediate job data that is consuming the disk space. You have to kill the application(s) responsible for writing lots of data. You can check the values of FILE_BYTES_WRITTEN counter to infer which is the bad application(s). Once you have killed the bad application(s), the cluster should become healthy in few minutes. If you re-run the hive query without fixing it to not to produce large intermediate data, the cluster will again end up in the same situation. One of the very common example of queries which result into a cross join. If you do ``explain`` of the query, the cross join would be flagged as [WARNING].

In both cases, using a larger cluster or cluster with more disk spaces on nodes can help mitigate the problem for short time.

### How do I use connect automatically when starting beeline?
``beeline -u "jdbc:hive2://localhost:10001/;transportMode=http" -n <usernmae> -p <password>``

### How do I connect to a different database(mydb) when starting beeline?
``beeline -u "jdbc:hive2://localhost:10001/mydb;transportMode=http" -n <usernmae> -p <password>``

### How do I set hive scrath directory?
``set hive.exec.scratchdir=/tmp/hdiuser``

## How do I configure the trash interval for HDFS?
Trash interval decides how long the data deleted from HDFS is kept around. You can configure it with ``fs.trash.interval``. The deleted data is kept under ``.Trash/Current`` directory on HDFS and can be restored from there.

### How does tez decide how many map tasks to generaete?

### How does tez decide how many reducer tasks to generate?

### How do turn on compression for intermediate output?

### How do I specify compression codec for intermediate output?

### How do I use [Hplsql](http://www.hplsql.org/)?
Use ``/usr/hdp/current/hive-server2-hive2/bin/hplsql``

### How do I know version of various components?
``hdp-select``

### How do I check the version of hadoop?
``hadoop version`` should return detailed version information as follows:

```
Hadoop 2.7.3.2.5.2.1-1
Subversion git@github.com:hortonworks/hadoop.git -r f3d31dd3dc216fff8acd4da01075656131947e1d
Compiled by jenkins on 2016-11-07T18:55Z
Compiled with protoc 2.5.0
From source with checksum a6c8c64bfe734b01a46c7c480ebc7ff
This command was run using /usr/hdp/2.5.2.1-1/hadoop/hadoop-common-2.7.3.2.5.2.1-1.jar
```

### How do I check if native compression codecs are properly setup?
``hadoop checknative -a`` should list output similar to

```
Native library checking:
hadoop:  true /usr/hdp/2.5.2.1-1/hadoop/lib/native/libhadoop.so.1.0.0
zlib:    true /lib/x86_64-linux-gnu/libz.so.1
snappy:  true /usr/lib/x86_64-linux-gnu/libsnappy.so.1
lz4:     true revision:99
bzip2:   true /lib/x86_64-linux-gnu/libbz2.so.1
openssl: true /usr/lib/x86_64-linux-gnu/libcrypto.so
```

### How do I dump metadata of an ORC file?
``hive --orcfiledump <path-to-ORC-file>``

### How do I dump content of an ORC file?
``hive --orcfiledump -d <path-to-ORC-file>``

### How do I specify the ORC compression?
``orc.compress``. Possible values are ``NONE``, ``ZLIB`` and ``SNAPPY``. ``ZLIB`` is a good default choice.

### How do I specify ORC strip size?
``orc.stripe.size``

### How do I specify ORC encoding strategy for integers?
``hive.exec.orc.encoding.strategy``. Possible values are ``SPEED`` and ``COMPRESSION``.

### Where can I find more information about Hive ORC configurations?
here : https://orc.apache.org/docs/hive-config.html

### How can I list all the effective configurations?
run ``set``.

### How do I get the query plan?
run ``explain `` followed by the query string.

### How do I get machine readble query plan?
run ``explain formatted `` followed by the query string.

### How do I get very detailed query plans?
run ``explain extended `` followed by the query string. This includes things like what partitions were pruned.

### How do I know if the a perticular join in the query is a map join or a merge join?
The query plan will have ``Map Join Operator`` and ``Merge Join Operarator`` to indicate that.

### How do I specify ORC split strategy?
set ``hive.exec.orc.split.strategy``. Possible values are ``BI``, ``ETL`` and ``HYBIRD``.

### Where are all hive configurations listed?
here : https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties
