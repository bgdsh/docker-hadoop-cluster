# Hadoop cluster using Docker

This repository contains Docker file to build a Docker image with Hadoop, Spark, HBase, Hive, Zookeeper and Kafka. The accompanying scripts can be used to start and stop the clusters easily.

## Pull the image

The image is released as an official Docker image from Docker automated build repository - you can always pull or refer the image when launching containers.

``` sh
docker pull karthikmlore/hadoop_cluster
```

## Build the image

If you would like to try directly from the Dockerfile you can build the image as:

``` sh
docker build --rm --no-cache -t karthikmlore/hadoop_cluster .
```

## Create network, containers and start cluster

### Through script

You can use the start_cluster.sh and stop_cluster.sh scripts to start and stop the hadoop cluster using bash or Windows Powershell.

* Default is 1 namenode with 2 datanodes (up to 8 datanodes currently possible, to add more edit "/usr/local/hadoop/etc/hadoop/slaves" and restart the cluster)
* Each node takes 1GB memory and 2 virtual cpu cores

``` sh
sh start_cluster.sh 2
sh stop_cluster.sh
```

### Manual procedure

#### Create bridge network

``` sh
docker network create --driver bridge hadoop
```

#### Create and start containers

Create a namenode container with the Docker image you have just built or pulled

``` sh
docker create -it -p 8088:8088 -p 50070:50070 -p 50075:50075 -p 2122:2122  --net hadoop --name namenode --hostname namenode --memory 1024m --cpus 2 karthikmlore/hadoop_cluster
```

Create and start datanode containers with the Docker image you have just built or pulled (up to 8 datanodes currently possible, to add more edit "/usr/local/hadoop/etc/hadoop/slaves" and restart the cluster)

``` sh
docker run -itd --name datanode1 --net hadoop --hostname datanode1 --memory 1024m --cpus 2 karthikmlore/hadoop_cluster
docker run -itd --name datanode2 --net hadoop --hostname datanode2 --memory 1024m --cpus 2 karthikmlore/hadoop_cluster
...
```

Start namenode container

``` sh
docker start namenode
```

#### Start cluster

``` sh
docker exec -it namenode //etc//bootstrap.sh start_cluster
```

After few minutes, you should be able to view Resource Manager UI at

`http://<host>:8088`

You should be able to access the HDFS UI at

`http://<host>:50070`

### Credentials

You can connect through SSH and SFTP clients to the namenode of the cluster using port 2122

``` yaml
Username: hdpuser
Password: hdppassword
```

#### Miscellaneous information

* You can login as root user into namenode using "docker exec -it namenode bash"
* To start HBase manually, log in as root (as described above) and executing the command "$HBASE_HOME/bin/start-hbase.sh"
* To start Kafka manually, log in as root (as described above) and executing the command "$KAFKA_HOME/bin/kafka-server-start.sh -daemon $KAFKA_HOME/config/server.properties"
* Kafka topics can be created by "hdpuser" with root privileges

``` sh
sudo $KAFKA_HOME/bin/kafka-topics.sh --create --zookeeper namenode:2181 --replication-factor 1 --partitions 1 --topic test
```

### Known issues

* Spark application master is not reachable from host system
* HBase and Kafka services do not start automatically sometimes (increasing memory of the container might solve this issue)
* No proper PySpark setup
* Unable to get Hive to work on Tez (current default MapReduce)

## Credits

* SequenceIQ - [https://github.com/sequenceiq](https://github.com/sequenceiq)
* Luciano Resende - [https://github.com/lresende](https://github.com/lresende)
