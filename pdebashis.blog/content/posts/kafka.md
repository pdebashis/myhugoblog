---
title: "Quick Notes : Kafka"
date: 2021-06-12T17:59:25+05:30
---

# Introduction

**Consumer**  Subscribe to a topic and pull messages 

**Producer** Send a message record to a topic 

**Topic** Has a consumer instance 

**Topic Partition** Have individual consumer instance 

**leader** is the node responsible for all reads and writes for the given partition. Each node will be the leader for a randomly selected portion of the partitions. 

**replicas** is the list of nodes that replicate the log for this partition regardless of whether they are the leader or even if they are currently alive. 

**isr** is the set of "in-sync" replicas. This is the subset of the replicas list that is currently alive and caught-up to the leader. 

# Startup

Start zookeeper and kafka 

Windows
    .\bin\windows\zookeeper-server-start.bat .\config\zookeeper.properties 
    .\bin\windows\kafka-server-start.bat .\config\server.properties 

Linux 
    bin/zookeeper-server-start.sh config/zookeeper.properties 
    bin/kafka-server-start.sh config/server.properties 


# Connectivity

Kafka vendor to share details for Authentication based on the method proposed. 

KafkaAccount Name 
bootstrap.servers(IP:Port combination) 
group.id with permission on required topics 
TopicNames  
jaas.conf file 
krb5.conf file 
Keytab file 

Consumer server ip's required for whitelisting against group id. 

# Kafka APIs

Producer API – Permits an application to publish streams of records. 

Consumer API – Permits an application to subscribe to topics and processes streams of records. 

Connector API – Executes the reusable producer and consumer APIs that can link the topics to the existing applications. 

Streams API – This API converts the input streams to output and produces the result. 

# Commands

List topics 

.\bin\windows\kafka-topics.bat --list --bootstrap-server localhost:9092 

bin/kafka-topics.sh --list --bootstrap-server localhost:9092 

 

Create Topic 

.\bin\windows\kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic test 

bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 3 --topic cms_cx_base_shared 

 

Delete Topic 

.\bin\windows\kafka-run-class.bat kafka.admin.TopicCommand --delete --topic test --zookeeper localhost:2181 

bin/kafka-topics.sh --delete --topic cms_cx_base_shared --zookeeper localhost:2181 

 

Produce and Consume 

.\bin\windows\kafka-console-producer.sh --broker-list localhost:9092 --topic test 

 

.\bin\windows\kafka-console-consumer.sh --bootstrap-server server1:9092 --topic test --from-beginning 

 

bin/kafka-console-producer.sh --broker-list localhost:9092 --topic cms_cx_base_shared --property parse.key=true --property key.separator="|" 

 

 

bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning 

 

cat web.json | /home/ushas/adaptor/dialog/dialog/kafka_2.11-1.0.0/bin/kafka-console-producer.sh --broker-list localhost:9092 --topic pelatro --property "parse.key=true" --property "key.separator=:" 

 

Describe Topics 

.\bin\windows\kafka-topics.bat --describe --bootstrap-server localhost:9092 --topic my-replicated-topic 

bin/kafka-topics.sh --describe --bootstrap-server localhost:9092 --topic cms_cx_base_shared 