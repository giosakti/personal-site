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
