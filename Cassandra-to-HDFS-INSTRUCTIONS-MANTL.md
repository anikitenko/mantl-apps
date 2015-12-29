This demo instructions should be applicable to any MANTL cluster.

### Description
*Dump table contents to HDFS*

Cassandra to HDFS application reads Cassandra table contents and stores them as a number of files on HDFS.
In case composite partition key discovered each table partition will be stored under separate directory.  

### Requirements
Tested on:

* Java 1.7.0_67
* Maven 3.3.3 (project assembly)
* Spark 1.3.0 (job runtime)
* Cassandra 2.2.3

## Prerequisites

* Make sure your user has home directory on HDFS (or any other directory with write permissions).

The below steps are based on the assumptions:

1) user's home directory on HDFS is `/user/$USER` where `$USER` is the current user logged on to the gateway host.

2) Data is already present in Cassandra table

3) Output directory: `hdfs:///user/$USER/data/cassandra-output`
 
 * Login to a gateway host and run the below commands:
 
```
$ git clone https://github.com/CiscoCloud/mantl-apps.git
$ cd mantl-apps/useful-apps/spark-useful-apps/cassandra-to-hdfs
$ mvn clean install -Pprod
ls -al target/cassandra-to-hdfs-1.0.jar
```
 
 * Copy jar to HDFS:
 
```
hdfs dfs -mkdir -p mantl-apps/useful-apps
hdfs dfs -put -f target/cassandra-to-hdfs-1.0.jar mantl-apps/useful-apps
hdfs dfs -ls mantl-apps/useful-apps/cassandra-to-hdfs-1.0.jar
```

### Configure
Application can be configured from command line providing parameters

* **--host** - required, specify it to point out Cassandra host, example:
```
--host localhost:9042
```
* **--table** - required, specify it to point out Cassadnra table full name including keyspace, example:
```
--table keyspace.table
```
* **--out** - required, specify it to point out directory URI to upload cassandra table contents, example:
```
--out hdfs://localhost/user/examples/files-out"
```
* --bsize - optional, HDFS file block size in Mb, default: governed by dfs.blocksize Hadoop option, example:
```
--bsize 32
```
* --master - optional, specify it to point out spark master URI, by default will be governed by --master option of spark-submit command which would be required in case not providing it application, example: 
```
--master spark://localhost:7077
```
* --name - optional, Application display name, default: Cassandra-to-HDFS , example:
```
--name "Cassandra-to-HDFS"
```
* *--help* - can be used to view usage options from command line.

 
## Run application

 * Output directory should not exist, remove target if applicable
 
 * Provide required parameters and run application.
``` 
/opt/spark/bin/spark-submit --master spark://mi-worker-003:7077 --deploy-mode cluster --class com.cisco.mantl.cassandra.CHDriver \
hdfs:///user/$USER/mantl-apps/useful-apps/cassandra-to-hdfs-1.0.jar -h cassandra-mantl-node.service.consul:9042 -t <keyspace>.<table> \
-o hdfs:///user/$USER/data/cassandra-output
```
 * Check `hdfs:///user/$USER/data/cassandra-output` for the output. In case target table had a composite partition key output should be divided into subfolders like part1-part2-etc.