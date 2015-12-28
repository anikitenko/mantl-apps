## This is a Spark-based DistCP application.

# Building

`mvn clean package`

* swift.conf example:
```
swift.endpoint=https://us-texas-3.cloud.cisco.com:5000/v2.0
swift.username=admin
swift.password=p@ssw0rd
swift.tenant=CIS-DataStore-US-TEXAS-3
```

## Copy your compiled spark-distcp-with-dependencies.jar to HDFS

	hadoop fs -mkdir -p hdfs://hdfs/mantl-apps/demo-applications
	hadoop fs -put target/spark-distcp-with-dependencies.jar hdfs://hdfs/mantl-apps/demo-applications/

### Configure
Configuration contains config file (swift.conf - for customizing Swift Storage endpoint and credentials) and command-line parameters

* **--swiftEndpoint** - required property for Swift Storage(API url). Replacement for swift.endpoint property from swift.conf
* **--swiftUsername** - username for Swift API. Replacement for swift.username property from swift.conf
* **--swiftPassword** - password for Swift API. Replacement for swift.password property from swift.conf
* **--swiftTenant** - tenant for Swift API. Replacement for swift.tenant property from swift.conf

* **--swiftConf** - or you can use config-file without external parameters (swiftEndpoint|swiftUsername|swiftPasswordswiftTenant) or with some of it

* **-m** - is a required property, and set of the execution mode: fromSwiftToHdfs|fromHdfsToSwift|fromHdfstoHdfs. example:
```
-m fromSwiftToHdfs
```
* **--swiftContainer** - specify a Swift Storage container. required with "fromSwiftToHdfs" and "fromHdfsToSwift" execution modes. example:
```
--swiftContainer sasa
```
* **--swiftUri** - specify an object name (URI-object). required with "fromSwiftToHdfs" and "fromHdfsToSwift" execution modes. example:
```
--swiftUri "tmp/tmp1.txt"
```
* **--hdfsUri** - specify a file on HDFS. required with "fromSwiftToHdfs" and "fromHdfsToSwift" execution modes. example:
```
--hdfsUri "/demo-applications/tmp.txt"
```

* **--hdfsSrc** - specify a source file on HDFS for copy. required with "fromHdfsToHdfs" execution mode. example:
```
--hdfsSrc "/demo-applications/tmp1.txt"
```
* **--hdfsDst** - specify a destination file on HDFS for copy. required with "fromHdfsToHdfs" execution mode. example:
```
--hdfsDst "/demo-applications/tmp2.txt"
```

## Run application
Run by submitting it to Spark via command line, providing mandatory parameters, example:

**Copy object from Swift Storage to HDFS**
```
/opt/spark/bin/spark-submit --master spark://mi-worker-003:7077 --deploy-mode cluster --conf spark.mesos.coarse=true \
--class com.cisco.mantl.SparkDistcp hdfs:///demo-applications/spark-distcp-with-dependencies.jar \
--swiftConf "./swift.conf" --swiftContainer test1 --swiftUri "tmp/tmp1.txt" --hdfsUri "/demo-applications/tmp.txt" -m fromSwiftToHdfs
```

**Copy file from HDFS to Swift Storage**
```
/opt/spark/bin/spark-submit --master spark://mi-worker-003:7077 --deploy-mode cluster --conf spark.mesos.coarse=true \
--class com.cisco.mantl.SparkDistcp hdfs:///demo-applications/spark-distcp-with-dependencies.jar \
--swiftConf "./swift.conf" --swiftContainer test1 --swiftUri "tmp/tmp1.txt" --hdfsUri "/demo-applications/tmp1.txt" -m fromHdfsToSwift
```

**Copy file on HDFS**
```
/opt/spark/bin/spark-submit --master spark://mi-worker-003:7077 --deploy-mode cluster --conf spark.mesos.coarse=true \
--class com.cisco.mantl.SparkDistcp hdfs:///demo-applications/spark-distcp-with-dependencies.jar \
--hdfsSrc "/demo-applications/tmp1.txt" --hdfsDst "/demo-applications/tmp6.txt" -m fromHdfsToHdfs
```