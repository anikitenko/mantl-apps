This demo instructions should be applicable to any MANTL cluster.

### Description
*Generate pseudo human activity data and save it to HDFS*

Application generates pseudo data of human activity in order to be further analysed. As a basis data model taken from [here](http://www.cis.fordham.edu/wisdm/dataset.php)
Data can be saved to HDFS or Cassandra:

* If decided to save into HDFS - output result will be a set of directories, 1 per user. However additional empty directories can be produced, so it's recommended to use [file aggregator](https://github.com/CiscoCloud/mantl-apps/tree/master/useful-apps/spark-useful-apps/file-aggregator) to further data usage.
* For Cassandra output - make sure table of kind **CREATE TABLE users (user_id int,activity text,timestamp bigint,acc_x double,acc_y double,acc_z double, PRIMARY KEY ((user_id,activity),timestamp));** has been created prior to application start.


Data format follows convention:
```
[user],[activity],[timestamp],[x-acceleration],[y-accel],[z-accel]
```

Representative data example:
```
1,Downstairs,96688372472,-5.470834779769724,-1.5037017960233374,-7.311290105459875
```
Fields description:

* user - nominal 1.. - will be incrementer for each consequent batch;
* activity - nominal { Walking, Jogging, Sitting, Standing, Upstairs, Downstairs }
* timestamp - numeric, generated long, signifies accelerometer device uptime in nanoseconds, sampling gap between activities is set to 200 ms, 100 ms - between feature windows, 50 +- 3% ms - for intra window timestamps
* x-acceleration - numeric, floating-point values between -20 .. 20 formed by function a*sin(x) + b (a,b - random integers from some range depending on axis and activity) for Walking, Jogging, Upstairs, Downstairs activities and  c +- d (c - some constant value, d - small fraction of c) for Sitting and Standing activities.
* y-acceleration - -//-
* z-acceleration - -//-


### Requirements
Tested on:

* Java 1.7.0_67
* Maven 3.3.3 (project assembly)
* Spark 1.3.0 (job runtime)

## Prerequisites

* Make sure your user has home directory on HDFS (or any other directory with write permissions).

The below steps are based on the assumptions:

1) user's home directory on HDFS is `/user/$USER` where `$USER` is the current user logged on to the gateway host.

2) Output Cassandra table has already been created with `CREATE TABLE <table> (user_id int,activity text,timestamp bigint,acc_x double,acc_y double,acc_z double, PRIMARY KEY ((user_id,activity),timestamp));`

3) Output directory: `hdfs:///user/$USER/data/hatgen-output`

 * Login to a gateway host and run the below commands:
 
```
$ git clone https://github.com/CiscoCloud/mantl-apps.git
$ cd mantl-apps/useful-apps/spark-useful-apps/hat/hat-data-generator
$ mvn clean install -Pprod
ls -al target/hat-data-generator-1.0.jar
```
 
 * Copy jar to HDFS:
 
```
hadoop fs -mkdir -p mantl-apps/useful-apps
hadoop fs -put -f target/hat-data-generator-1.0.jar mantl-apps/useful-apps
hadoop fs -ls mantl-apps/useful-apps/hat-data-generator-1.0.jar
```

## Run Application

* Provide required parameters and run application.

### Configure
Application can be configured from command line providing parameters

* **--out** - required property, it can be either directory URI to upload generator output examples:
```
--out hdfs:///user/examples/files-out
--out 192.168.159.128:9042:keyspace:table
```
* **--amount** - required, amount of data in megabytes to be generated, example:
```
--amount 100
```
* --frequency - optional, batch frequency in seconds, default is set to 3 seconds, example:
```
--frequency 5
```
* --batchsize - optional, amount of data in megabytes to be generated per batch, affects number of output directories, default: 1, example:
```
--batchsize 10
```
* --blocksize - optional, HDFS file block size in Mb, default: governed by dfs.blocksize Hadoop option, example:
```
--blocksize 32
```
* --master - optional, specify it to point out spark master URI, by default will be governed by --master option of spark-submit command which would be required in case not providing it application, example: 
```
--master spark://localhost:7077
```
* --name - optional, Application display name, default: Cassandra-to-HDFS , example:
```
--name "Cassandra-to-HDFS"
```
* --print - optional, enable/disable batch contents to be printed to console output, default: false, example:
```
--print true
```
* *--help* - can be used to view usage options from command line.

For HDFS output:

``` 
/opt/spark/bin/spark-submit --master spark://mi-worker-003:7077 --deploy-mode cluster --class com.cisco.mantl.hat.gen.HATGenDriver hdfs:///user/$USER/mantl-apps/useful-apps/hat-data-generator-1.0.jar --out hdfs:///user/$USER/data/hatgen-output --amount 10 --frequency 1 --batchsize 10 --print true
``` 

For Cassandra output:

``` 
/opt/spark/bin/spark-submit --master spark://mi-worker-003:7077 --deploy-mode cluster --class com.cisco.mantl.hat.gen.HATGenDriver hdfs:///user/$USER/mantl-apps/useful-apps/hat-data-generator-1.0.jar --out cassandra-mantl-node.service.consul:9042:mykeyspace:users --amount 10 --frequency 1 --batchsize 10 --print true
```

* Kill application on worker node after making sure it stopped spawning data (required due to the bug). This can be done via stdout log, --print true must be set for the job.
```
ps ax | grep HATGenDriver
kill -9 <process_id>
```

* Check the output

For HDFS:

``` 
hadoop fs -ls -h hdfs:///user/$USER/data/hatgen-output
```

Directory should contain number of directories with content files, some of them however will be empty. For further convenient usage aggregate data using [file aggregator](https://github.com/CiscoCloud/mantl-apps/tree/master/useful-apps/spark-useful-apps/file-aggregator)

For Cassandra:

``` 
USE <keyspace>;
SELECT COUNT(*) FROM <table>;
```

Count number should be displayed, approx 13*10^3 per 1 Mb.