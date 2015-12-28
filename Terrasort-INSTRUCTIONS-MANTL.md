# TeraSort benchmark for Spark.
## This instruction should be applicable to any Mesos managed cluster

## This is a Spark-based TeraSort application.

# Building

`mvn install`

# Running

`cd` to your your spark install.

## Copy your compiled terasort.jar to HDFS

	hadoop fs -mkdir -p hdfs://hdfs/mantl-apps/benchmarking-apps
	hadoop fs -mkdir hdfs://hdfs/test
	hadoop fs -put target/spark-terasort-1.0-jar-with-dependencies.jar hdfs://hdfs/mantl-apps/benchmarking-apps/

## Generate data

    ./bin/spark-submit --master spark://mi-worker-003:25317 --deploy-mode cluster --class com.cisco.mantl.terasort.TeraGen \
    hdfs://hdfs/mantl-apps/benchmarking-apps/spark-terasort-1.0-jar-with-dependencies.jar 1G hdfs://hdfs/test/terainput

## Sort the data
    ./bin/spark-submit --master spark://mi-worker-003:25317 --deploy-mode cluster --class com.cisco.mantl.terasort.TeraSort \
    hdfs://hdfs/mantl-apps/benchmarking-apps/spark-terasort-1.0-jar-with-dependencies.jar hdfs://hdfs/test/teraoutput

## Validate the data
    ./bin/spark-submit --master spark://mi-worker-003:25317 --deploy-mode cluster --class com.cisco.mantl.terasort.TeraValidate \
    hdfs://hdfs/mantl-apps/benchmarking-apps/spark-terasort-1.0-jar-with-dependencies.jar hdfs://hdfs/test/teravalidate
