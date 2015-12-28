This demo instructions should be applicable to any MANTL-managed cluster.

## Prerequisites

* Make sure your user has home directory on HDFS (or any other directory with write permissions).

The below steps are based on the assumption that user's home directory on HDFS is `/user/$USER` where `$USER` is the current user logged on to the gateway host.

* Login to a gateway host and run the below commands:

```
git clone https://github.com/CiscoCloud/mantl-apps
cd mantl-apps/useful-apps/spark-useful-apps/file-aggregator
mvn clean package
ls -al target/file-aggregator-with-dependencies.jar
```

* Copy jar to HDFS:

```
hdfs dfs -mkdir -p mantl-apps/useful-apps
hdfs dfs -put -f file-aggregator-with-dependencies.jar mantl-apps/useful-apps
hdfs dfs -ls mantl-apps/useful-apps/file-aggregator-with-dependencies.jar
```

### Configure
Application can be configured from command line providing parameters

* **-i** - is a required property, specify it to point out directory URI small files to be read and aggregated into combined files, example:
```
-i hdfs://localhost/user/examples/files
```
* **-o** - is a required property, specify it to point out directory URI to upload combined files to, **NOTE:** directory should not exist, it will be created as part of the job, example:
```
-o hdfs://localhost/user/examples/files-out
```
* -m - is a optional property, specify it to point out spark master URI, by default will be governed by --master option of spark-submit command which would be required in case not providing it to File Aggregator application, example: 
```
-m spark://quickstart.cloudera:7077
```
* -n - is a optional property, Application display name, default: File Aggregator , example:
```
-n "File Agg"
```

* -f - is an optional property, Max size of single output file in Mb, default: 128 , example:
```
-f 64
```
* -b - is an optional property, HDFS file block size in Mb, default: governed by dfs.blocksize Hadoop option , example:
```
-b 64
```
* -d - is an optional property, delimiter to separate content from small input files in combined files, default: "linebreak" , example:
```
-d %
```
* -r - is an optional property, enables recursive file reads in nested input directories, default: true , example:
```
-r false
```
* *--help* - can be used to view usage options from command line.

## MODE 1 - File Aggregation with default parameters

Default parameters are:

Max output file size: 128 MB

Output file block size: dfs.blocksize

Spark application name: File Aggregator

Include input directories recursively: true

* Remove previous output directory:

```
hdfs dfs -rm -r -f -skipTrash data/file-aggregator-output
```

* Run the jar command:

```
/opt/spark/bin/spark-submit --master spark://mi-worker-003:7077 --deploy-mode cluster --class com.cisco.mantl.aggregator.AggDriver \
 hdfs:///user/$USER/mantl-apps/useful-apps/file-aggregator-with-dependencies.jar \
 -i hdfs:///data/sasa-logs/20150224/www.cisco.com \
 -o hdfs:///user/$USER/data/file-aggregator-output
```

* Check the result (comparing size of input and output directories):

```
hdfs dfs -du -s -h hdfs:///user/$USER/data/file-aggregator-output
hdfs dfs -du -s -h hdfs:///data/sasa-logs/20150224/www.cisco.com
```

## MODE 2 - File Aggregation with custom parameters

For example:

Max output file size: 1024MB

Output file block size: 256MB

* Remove previous output directory:

```
hdfs dfs -rm -r -f -skipTrash data/file-aggregator-output-custom
```

* Run the jar command:

```
/opt/spark/bin/spark-submit --master spark://mi-worker-003:7077 --deploy-mode cluster --class com.cisco.mantl.aggregator.AggDriver \
 hdfs:///user/$USER/mantl-apps/useful-apps/file-aggregator-with-dependencies.jar \
 -i hdfs:///data/sasa-logs/20150224/www.cisco.com \
 -o hdfs:///user/$USER/data/file-aggregator-output-custom \
 -f 1024 -b 256 
```

* Check the result (comparing size of input and output directories):

```
hdfs dfs -du -s -h hdfs:///user/$USER/data/file-aggregator-output-custom
hdfs dfs -du -s -h hdfs:///data/sasa-logs/20150224/www.cisco.com
```