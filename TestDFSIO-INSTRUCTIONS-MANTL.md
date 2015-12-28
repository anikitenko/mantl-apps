# TestDFSIO for Spark.
## This instruction should be applicable to any Mesos managed cluster

## This is a Spark-based TestDFSIO application.

# Building

`mvn clean package`

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