---
title: "Kafka"
authors: ["gio"]
---

#### Useful commands

```
./kafka-consumer-groups.sh  --list --bootstrap-server <KAFKA BROKER>:9092
./kafka-consumer-groups.sh --describe --group <CONSUMER GROUP> --bootstrap-server <KAFKA BROKER>:9092

kafkacat -b 172.28.22.36:9092 -t goresto-api_logs -p 0 -o -10

# Read the last 2000 messages from 'syslog' topic, then exit
kafkacat -C -b mybroker -t syslog -p 0 -o -2000 -e
```

#### Fixing `Name or service not known`

By default kafka will use machine hostname to advertise its address to listeners. See [this](https://github.com/confluentinc/confluent-kafka-go/issues/70) issue for details.

If kafka box cannot be accessed from other box using the hostname, you may need to modify `advertised.listeners` on its `server.properties` file then restart it.

Multiple listeners can be specified, separated by comma. Example of valid `advertised.listeners` value:

```
PLAINTEXT://myhost:9092,TRACE://:9091 PLAINTEXT://0.0.0.0:9092, TRACE://localhost:9093
```

#### Kafka on Low Memory Environment

https://groups.google.com/forum/#!topic/confluent-platform/KA-RxeVIstA

#### kafka-manager

Installing kafka-manager manually

```
sudo apt-get install openjdk-8-jdk

sudo cp /etc/profile /etc/profile_backup 
echo 'export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-amd64' | sudo tee -a /etc/profile
source /etc/profile

wget https://downloads.lightbend.com/scala/2.12.6/scala-2.12.6.deb
sudo dpkg -i scala-2.12.6.deb
scala -version

echo "deb https://dl.bintray.com/sbt/debian /" | sudo tee -a /etc/apt/sources.list.d/sbt.list
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 2EE0EA64E40A89B84B2DF73499E82A75642AC823
sudo apt-get update
sudo apt-get install sbt

git clone https://github.com/yahoo/kafka-manager.git
cd ./kafka-manager
sbt clean dist

mv ./target/universal/kafka-manager-1.3.3.8.zip ~/
unzip kafka-manager-1.3.3.8.zip
rm kafka-manager-1.3.3.8.zip
cd kafka-manager-1.3.3.8
```

## Running Kafka on Production

#### Common Issues

- Zookeeper out of sync
- Unregistered broker
- Under replicated partition
- Kafka disk is full

#### CommitFailedException: Commit cannot be completed since the group has already rebalanced and assigned the partitions to another member

If we encountered the above issues, it is likely because the time a consumer take to process all the records that are fetched after `poll()` action took longer than `max.poll.inernal.ms` configuration. This means the consumer do not call `poll()` frequently enough that it is considered inactive by the broker.

In order to fix this issues, there are several thing that we can do (in order of priority):  

1. Improve records processing latency if possible
2. Reduce `max.poll.records` (consumer config) to limit the total records returned from a single call to `poll()`
3. Increase `max.poll.interval.ms` (consumer config) to allow more time between calls to `poll()`. Note that this might reduce system responsiveness to do rebalancing.

At the same time, different thread in consumer is also sending a periodic heartbeat to server that can be configured via `heartbeat.interval.ms`. We need to make sure that value is lower than `session.timeout.ms`.

For use cases where message processing time varies unpredictably, neither of above options may be sufficient. The recommended way to handle these cases is to move message processing to another thread, which allows the consumer to continue calling poll while the processor is still working. Some care must be taken to ensure that committed offsets do not get ahead of the actual position. 

Typically, you must disable automatic commits and manually commit processed offsets for records only after the thread has finished handling them (depending on the delivery semantics you need). Note also that you will need to pause the partition so that no new records are received from poll until after thread has finished handling those previously returned. 

References:

- [Kafka Consumer - Detecting Consumer Failures](https://kafka.apache.org/10/javadoc/org/apache/kafka/clients/consumer/KafkaConsumer.html)
- [KIP-62: Allow consumer to send heartbeats from a background thread](https://cwiki.apache.org/confluence/display/KAFKA/KIP-62%3A+Allow+consumer+to+send+heartbeats+from+a+background+thread)

#### Optimizing Kafka

If we can afford missing messages, we can do this to increase throughput:
- Leader ACK only when producing message
- Unclean leader election, unclean node can become leader

Fronting kafka architecture:
- https://blog.gojekengineering.com/kafka-4066a4ea8d0d

Misc:
- https://www.confluent.io/blog/how-choose-number-topics-partitions-kafka-cluster
- https://hiya.com/blog/2018/06/07/hiyas-best-practices-around-kafka-consistency-and-availability/
- https://engineering.linkedin.com/kafka/benchmarking-apache-kafka-2-million-writes-second-three-cheap-machines
