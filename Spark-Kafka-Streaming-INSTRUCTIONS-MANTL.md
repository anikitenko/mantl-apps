This demo instructions should be applicable to any MANTL cluster.

## Prerequisites

Below steps are based on the assumptions:

1) User's home directory on HDFS is `/user/$USER` where `$USER` is the current user logged on to the gateway host.
In case different directory is planned to be used, make sure write permission is available.

2) Kafka topic with messages is available - `test-topic` with `3` partitions

3) Create target Cassandra table, use [streaming_sample.cql](https://github.com/CiscoCloud/mantl-apps/blob/master/demo-apps/spark-demo-apps/sparkstreaming-kafka-cassandra-simple/streaming_maple.cql)

* Login to gateway host and run the below commands:

```
git clone https://github.com/CiscoCloud/mantl-apps
cd mantl-apps/demo-apps/spark-demo-apps/sparkstreaming-kafka-cassandra-simple
mvn clean install
ls -al target/sparkstreaming-kafka-cassandra-simple-with-dependencies.jar
```

* Copy jar to HDFS:
```
hadoop fs -mkdir -p mantl-apps/demo-apps/
hadoop fs -put -f target/sparkstreaming-kafka-cassandra-simple-with-dependencies.jar mantl-apps/demo-apps
hadoop fs -ls mantl-apps/demo-apps/sparkstreaming-kafka-cassandra-simple-with-dependencies.jar
```

## Run Spark Streaming application

```
/opt/spark/bin/spark-submit --master spark://mi-worker-003:7077 --deploy-mode cluster --class com.cisco.mantl.demo.streaming.KafkaStream \
hdfs:///user/$USER/mantl-apps/demo-apps/sparkstreaming-kafka-cassandra-simple-with-dependencies.jar \
-z zookeeper.service.consul:2181 \
-g my-consumer-group \
-t test-topic \
-n 1 \
-a cassandra-mantl-node.service.consul \
-p 9042
```

## Check the result

```
/opt/apache-cassandra-2.2.4/bin/cqlsh cassandra-mantl-node.service.consul 9042 --protocolversion=3 --cqlversion=3.2.0
USE streamingsample;
SELECT * FROM streaming_sample;
```