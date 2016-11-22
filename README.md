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

### How do I download YARN container logs?

### How do I download hive client logs?

### How do I download hive metastore logs?

### How do I download hiveserver logs?

### How to I start hive shell with debug logging enabled on console?
hive -hiveconf hive.root.logger=ALL,console

### How do I use tez engine?
set hive.execution.engine=tez

### How do I download tez DAG data?
hadoop jar /usr/hdp/current/tez-client/tez-history-parser-*.jar org.apache.tez.history.ATSImportTool -downloadDir . -dagId \<DagId\>

### How do I get CrticalPath for a Tez DAG?
hadoop jar /usr/hdp/current/tez-client/tez-job-analyzer-*.jar CriticalPath --saveResults --dagId \<DagId\> --eventFileName \<DagData.zip\>

### How do I kill an application?
yarn application -kill \<ApIpd\>

This will kill any query that is being run as part of the Tez session also.

### How do I check currently running application on the cluster?
yarn top

### How do I specify the database while starting hive?
hive -database \<databaseName\>

### How do I specify a config or variable while starting hive?
hive -hiveconf a=b

### How do I disable mapjoin?
set hive.auto.convert.join=false

### How do I decide which joins are converted to mapjoins?
set hive.auto.convert.join.noconditionaltask.size to value of the largest join you want to convert to mapjoin. For example, set hive.auto.convert.join.noconditionaltask.size=10000000 will convert all joins to mapjoins where hive "estimates" the sum of smallest n-1 tables in n-join is less then 1MB.
