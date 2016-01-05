This demo instructions should be applicable to any MANTL cluster.

## Prerequisites

Below steps are based on these assumptions:

1) User's home directory on HDFS is `/user/$USER` where `$USER` is the current user logged on to the gateway host.
In case different directory is planned to be used, make sure write permission is available.

2) Input dataset is in `hdfs:///data/sasa-logs-without-recursion` directory. Recursive directory read IS SUPPORTED for the application, but here we use directory that does not contain recutsive folders ( since previously we demonstrate file aggregation using  [File aggregator](https://github.com/CiscoCloud/mantl-apps/blob/master/useful-apps/spark-useful-apps/file-aggregator) application )

3) Zookeeper quorum is: `zookeeper.service.consul:2181`

4) Kafka brokers are: `mi-worker-004.cisco.com:9092,mi-worker-003.cisco.com:9093`

Btw brokers could be added by running the following commands:
```
jre/bin/java -Xmx256m -jar kafka-mesos-0.9.2.0.jar broker add 0 --api=http://mi-worker-003.cisco.com:25341 --port 9092
jre/bin/java -Xmx256m -jar kafka-mesos-0.9.2.0.jar broker add 1 --api=http://mi-worker-003.cisco.com:25341 --port 9093
```

* Login to a gateway host and run the below commands:

```
git clone https://github.com/CiscoCloud/mantl-apps
cd mantl-apps/demo-apps/hadoop-demo-apps/fs-kafka-producer
mvn clean install -Pprod
ls -al target/kafka_producer-1.0.jar
```

* *Optional step:* create and configure Kafka topic if required:
```
 /opt/kafka_2.9.1-0.8.2.2/bin/kafka-topics.sh --create --zookeeper zookeeper.service.consul:2181 --replication-factor 1 --partitions 3 --topic test-topic
```

```
 /opt/kafka_2.9.1-0.8.2.2/bin/kafka-topics.sh --describe --zookeeper zookeeper.service.consul:2181 --topic test-topic
 ```

## Run Kafka Producer application

* Start application (as a background process):

```
nohup hadoop jar target/fs-kafka-producer-1.0.jar com.cisco.mantl.kafka.KProdDriver --brokers mi-worker-004.cisco.com:9092,mi-worker-003.cisco.com:9093 --topic test-topic --inputDir hdfs:///data/sasa-logs-without-recursion --zkhost zookeeper.service.consul:2181 &
```

* Optional step: start Kafka consumer to verify that messages are going to the topic:
```
/opt/kafka_2.9.1-0.8.2.2/bin/kafka-console-consumer.sh --zookeeper zookeeper.service.consul:2181 --topic test-topic
```

* Check the result in the consumer terminal window. It should look something like this:
```
Thread-19;Time-432681059397966;Line-<content>
Thread-19;Time-432683833523908;Line-<content>
...
```