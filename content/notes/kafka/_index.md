---
title: "Kafka"
authors: ["gio"]
---

#### Fixing `Name or service not known`

By default kafka will use machine hostname to advertise its address to listeners. See [this](https://github.com/confluentinc/confluent-kafka-go/issues/70) issue for details.

If kafka box cannot be accessed from other box using the hostname, you may need to modify `advertised.listeners` on its `server.properties` file then restart it.

Multiple listeners can be specified, separated by comma. Example of valid `advertised.listeners` value:

```
PLAINTEXT://myhost:9092,TRACE://:9091 PLAINTEXT://0.0.0.0:9092, TRACE://localhost:9093
```

#### Kafka on Low Memory Environment

https://groups.google.com/forum/#!topic/confluent-platform/KA-RxeVIstA

#### Using kafkacat to peek into a topic

```

kafkacat -b 172.28.22.36:9092 -t goresto-api_logs -p 0 -o -10

Read the last 2000 messages from 'syslog' topic, then exit
$ kafkacat -C -b mybroker -t syslog -p 0 -o -2000 -e
```

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

#### Monitoring Lag using Burrow

See [here](https://github.com/linkedin/Burrow)

## Running Kafka on Production

#### Common Issues

- Zookeeper out of sync
- Unregistered broker
- Under replicated partition
- Kafka disk is full

#### Optimizing Kafka

If we can afford missing messages, we can do this to increase throughput:
- Leader ACK only when producing message
- Unclean leader election, unclean node can become leader

Fronting kafka architecture:
- https://blog.gojekengineering.com/kafka-4066a4ea8d0d

#### Monitoring & Tools

- Burrow (to monitor lag)
- Jolokia
- Kafka Manager
- Zookeeper CLI
