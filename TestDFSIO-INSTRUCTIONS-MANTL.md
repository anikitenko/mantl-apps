# TestDFSIO for Spark.

## Prerequisites

* Make sure your user have home directory on HDFS (or any other directory with write permissions).

The below steps are based on the assumption that user's home directory on HDFS is `/user/$USER` where `$USER` is the current user logged on to the gateway host.

* Login to a gateway host and run the below commands:

```
git clone https://github.com/CiscoCloud/mantl-apps
cd mantl-apps/benchmarking-apps/spark-benchmarking-apps/spark-test-dfsio
mvn clean package
ls -al target/spark-test-dfsio-with-dependencies.jar
```

# Running

`cd` to your your spark install.

## Copy your compiled spark-test-dfsio-with-dependencies.jar to HDFS

	hadoop fs -mkdir -p hdfs://hdfs/mantl-apps/benchmarking-apps
	hadoop fs -mkdir hdfs://hdfs/test
	hadoop fs -put target/spark-test-dfsio-with-dependencies.jar hdfs://hdfs/mantl-apps/benchmarking-apps/

## Testing WRITE performance

    /opt/spark/bin/spark-submit --master spark://mi-worker-003:7077 --deploy-mode cluster \
	--class com.cisco.dfsio.test.Runner hdfs:///mantl-apps/benchmarking-apps/spark-test-dfsio-with-dependencies.jar \
	--file test/testdfsio-write --nFiles 10 --fSize 200000000 -m write --log test/testdfsio-write/testHdfsIO-WRITE.log

## Testing READ performance

	/opt/spark/bin/spark-submit --master spark://mi-worker-003:7077 --deploy-mode cluster \
	--class com.cisco.dfsio.test.Runner hdfs:///mantl-apps/benchmarking-apps/spark-test-dfsio-with-dependencies.jar \
	--file data/testdfsio-write --nFiles 10 --fSize 200000000 -m read --log data/testdfsio-write/testHdfsIO-READ.log