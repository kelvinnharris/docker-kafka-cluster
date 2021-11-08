# docker-kafka-cluster
docker-kafka-cluster allows to deploy a Kafka multi node cluster with Docker

## Steps to reproduce

### 1. Build the base image 
- Clone the repo and `cd` into the repo.
- `docker build -t kafka_base .`

### 2. Launch the kafka cluster
- `docker-compose up -d`

### 3.	Test for Pub-sub messaging

- Run a client 

```
docker run -it --network host kafka_base /bin/bash
```
- Create a topic

```
/bin/kafka-topics --create --zookeeper localhost:2181 --replication-factor 3 --partitions 1 --topic test
```
- Run a console producer

```
./bin/kafka-console-producer --topic test --broker-list localhost:9091,localhost:9092,localhost:9093
```

- Run a console consumer

```
./bin/kafka-console-consumer --topic test --bootstrap-server localhost:9091,localhost:9092,localhost:9093
```

-	Test out communication by typing in messages
- The messages will be reflected in the consumer side terminal

Producer:

```
root@docker-desktop:/confluent-5.2.2# ./bin/kafka-console-producer --topic test --broker-list localhost:9091,localhost:9092,localhost:9093
>a
>hello
>test
```

Consumer:

```
root@docker-desktop:/confluent-5.2.2# ./bin/kafka-console-consumer --topic test --bootstrap-server localhost:9091,localhost:9092,l  ocalhost:9093
a
hello
test
```

### 4.Testing for fault tolerance
To test for fault tolerance, we can try to stop a broker container.

```
docker stop broker1
```

A warning will be raised for both producer and consumer side. However, messages can still be sent over as failure of nodes are managed correctly.

Producer:

```
root@docker-desktop:/confluent-5.2.2# ./bin/kafka-console-producer --topic test --broker-list localhost:9091,localhost:9092,localhost:9093

>[2021-11-08 15:34:12,263] WARN [Producer clientId=console-producer] Connection to node -1 (localhost/127.0.0.1:9091) could not be established. Broker may not be available. (org.apache.kafka.clients.NetworkClient)
>new message
>hello
>test message
>ok
```

Consumer side:

```
[2021-11-08 15:33:21,285] WARN [Consumer clientId=consumer-1, groupId=console-consumer-76904] Connection to node 1 (localhost/127.  0.0.1:9091) could not be established. Broker may not be available. (org.apache.kafka.clients.NetworkClient)
new message
hello
test message
ok
^CProcessed a total of 10 messages
```

## References

- https://medium.com/@PierreKieffer/deploy-a-kafka-multi-node-cluster-with-docker-72878ddbaf96
