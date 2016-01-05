This demo instructions should be applicable to any MANTL cluster.

## Prerequisites

Below steps are based on the assumptions:

1) User's home directory on HDFS is `/user/$USER` where `$USER` is the current user logged on to the gateway host.
In case different directory is planned to be used, make sure write permission is available.

2) Kafka topic with messages is available - `test-topic` with `3` partitions

3) Zookeeper quorum is: `zookeeper.service.consul:2181`

4) Kafka brokers are: mi-worker-004.cisco.com:9092,mi-worker-003.cisco.com:9093

* Login to gateway host and run the below commands:

```
git clone https://github.com/CiscoCloud/mantl-apps
cd mantl-apps/demo-apps/spark-demo-apps/sparkstreaming-kafka-hdfs-simple
mvn clean install
ls -al target/sparkstreaming-kafka-hdfs-simple-with-dependencies.jar
```

* Copy jar to HDFS:
```
hadoop fs -mkdir -p mantl-apps/demo-apps/
hadoop fs -put -f target/sparkstreaming-kafka-hdfs-simple-with-dependencies.jar mantl-apps/demo-apps
hadoop fs -ls mantl-apps/demo-apps/sparkstreaming-kafka-hdfs-simple-with-dependencies.jar
```

* *Optional step:* Create and configure Kafka topic if required:
```
 /opt/kafka_2.9.1-0.8.2.2/bin/kafka-topics.sh --create --zookeeper zookeeper.service.consul:2181 --replication-factor 1 --partitions 3 --topic test-topic
```

* *Optional step:* Start Kafka producer:
```
 /opt/kafka_2.9.1-0.8.2.2/bin/kafka-console-producer.sh --broker-list mi-worker-004.cisco.com:9092,mi-worker-003.cisco.com:9093 --topic test-topic
```

## Run Application

* Remove previous output directory:
```
hadoop fs -rm -r -f -skipTrash data/demo-apps/spark-demo-apps/sparkstreaming-kafka-hdfs-simple
```

* Create output directory:
```
hadoop fs -mkdir -p data/demo-apps/spark-demo-apps/sparkstreaming-kafka-hdfs-simple
```

* Run spark job:
```
/opt/spark/bin/spark-submit --master spark://mi-worker-003:7077 --deploy-mode cluster --class com.cisco.mantl.demo.streaming.KafkaStream \
hdfs:///user/$USER/mantl-apps/demo-apps/sparkstreaming-kafka-hdfs-simple-with-dependencies.jar \
-z zookeeper.service.consul:2181 \
-g my-consumer-group \
-t test-topic \
-n 3 \
-o hdfs:///user/$USER/data/demo-apps/spark-demo-apps/sparkstreaming-kafka-hdfs-simple/demo
```

* *Optional step:* Supply data into Kafka producer

* Check the result in the output directory:

```
hadoop fs -ls -h "/user/$USER/data/demo-apps/spark-demo-apps/sparkstreaming-kafka-hdfs-simple/*/*"
```

```
watch -n 10 'hadoop fs -ls -h "/user/$USER/data/demo-apps/spark-demo-apps/sparkstreaming-kafka-hdfs-simple/demo-*/part-*" | wc -l '
```

* Stop application by killing it on worker node:

```
ps ax | grep KafkaStream
kill -9 <process_id>
```