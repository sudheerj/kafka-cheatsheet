# kafka-cheatsheet

## Install Kafka with Zookeeper on Windows
1. Install WSL2 using `wsl --install`

2. Install Java JDK version 11

3. Download Apache Kafka from https://kafka.apache.org/downloads under Binary Downloads section or use wget command

    ```bash
    wget https://archive.apache.org/dist/kafka/3.8.0/kafka_2.13-3.8.0.tgz
    ```
    Add below system run command in `.bashrc`

    ```bash
    PATH="$PATH:~/kafka_2.13-3.8.0/bin"
    ```

4. Extract the contents on WSL2 and setup the $PATH environment variables to easy access the Kafka binaries
   ```bash
   tar xzf kafka_2.13-3.0.0.tgz
   mv kafka_2.13-3.0.0 ~
   ```

5. Start Zookeeper using the binaries in WSL2
   ```bash
   zookeeper-server-start.sh ~/kafka_2.13-3.8.0/config/zookeeper.properties
   ```

6. Start Kafka using the binaries in an another process in WSL2

   ```bash
   kafka-server-start.sh ~/kafka_2.13-3.8.0/config/server.properties
   ```

## Kafka Topics commands

 1. Create:

      ```bash
      kafka-topics.sh --bootstrap-server localhost:9092 --topic first_topic --create

      kafka-topics.sh --bootstrap-server localhost:9092 --topic second_topic --create --partitions 3

      kafka-topics.sh --bootstrap-server localhost:9092 --topic third_topic --create --partitions 3 --replication-factor 1
      ```

 2. List:
   
      ```bash
      kafka-topics.sh --bootstrap-server localhost:9092 --list 
      ```
 3. Describe:
   
      ```bash
      kafka-topics.sh --bootstrap-server localhost:9092 --topic first_topic --describe 
      ```
 4. Delete:
   
      ```bash
      kafka-topics.sh --bootstrap-server localhost:9092 --topic first_topic --delete // works only when delete.topic.enable=true
      ```

## Kafka Console Producer commands

   1. Basic producing:
   
      ```bash
      kafka-console-producer.sh --bootstrap-server localhost:9092 --topic first_topic
      >Hello
      >May name is Sudheer
      >This is a kafka demo
      ```

   2. Producer with properties:

      ```bash
       kafka-console-producer.sh --bootstrap-server localhost:9092 --topic first_topic --producer-property acks=all
      >The producer supports properties through arguments
      >Let's broker acknowledge the message
      ```

   3. Producting to non-existing topic:

      ```bash
      kafka-console-producer.sh --bootstrap-server localhost:9092 --topic new_topic
      > hello new topic!
      ```
   4. Producting with keys:
   
      ```bash
      kafka-console-producer.sh --bootstrap-server localhost:9092 --topic first_topic --property parse.key=true --property key.separator=*
      >veggies: potato
      >fruits: banana
      ```

## Kafka Console Consumer commands

1. Consuming messages:

   ```bash
   kafka-topics.sh --bootstrap-server localhost:9092 --topic fourth_topic --create --partitions 3
   kafka-console-producer.sh --bootstrap-server localhost:9092 --producer-property partitioner.class=org.apache.kafka.clients.producer.RoundRobinPartitioner --topic fourth_topic
   >one
   >two
   >three

   kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic fourth_topic
   one
   two
   three
   ```

2. Consuming from beginning:

   ```bash
   kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic fourth_topic --from-beginning
   one
   three
   two
   ```

3. Message formatting with key, value, partitioning and timestamp info:

   ```bash
   kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic fourth_topic --formatter kafka.tools.DefaultMessageFormatter --property print.timestamp=true --property print.key=true --property print.value=true --property print.partition=true --from-beginning

   CreateTime:1724466964735        Partition:1     null    one
   CreateTime:1724466968330        Partition:0     null    three
   CreateTime:1724466966554        Partition:2     null    two
   ``` 

## Kafka Console Consumer group commands

**Prerequisites:** Create a new topic with 3 partitions

```bash
kafka-topics.sh --bootstrap-server localhost:9092 --topic fourth_topic --create --partitions 3
```

1. Create a first consumer in consumer group:
```bash
root@Sonu:~# kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic fourth_topic --group my-first-group
```

2. Create a producer to produce messages:
```bash
kafka-console-producer.sh --bootstrap-server localhost:9092 --producer-property partitioner.class=org.apache.kafka.clients.producer.RoundRobinPartitioner --topic fourth_topic
>one
>two
>three
>four
```

3. Create more consumers in a group and message spread:
```bash
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic fourth_topic --group my-first-group
one
four
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic fourth_topic --group my-first-group
three
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic fourth_topic --group my-first-group
two
```

4. Create a new group with from beginning option:
```bash
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic fourth_topic --group my-fourth-group --from-beginning
two
one
four
three
```

## Kafka Consumer Group commands
1. List consumer groups
   
```bash
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list
my-fourth-group
my-first-group
my-third-group
my-second-group
```

2. Describe a consumer group

```bash
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group my-first-group

GROUP           TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID                                           HOST            CLIENT-ID
my-first-group  fourth_topic    0          3               3               0               console-consumer-e6eb8120-3613-4b28-b722-6917474e8d46 /127.0.0.1      console-consumer
my-first-group  fourth_topic    1          3               3               0               console-consumer-e6eb8120-3613-4b28-b722-6917474e8d46 /127.0.0.1      console-consumer
my-first-group  fourth_topic    2          2               2               0               console-consumer-e91f28d3-403b-45b4-89a7-1fb60566fe95 /127.0.0.1      console-consumer
```

3. Reset offsets

```bash
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group my-first-group --reset-offsets --to-earliest --topic fourth_topic --execute

WARN [AdminClient clientId=adminclient-1] The DescribeTopicPartitions API is not supported, using Metadata API to describe topics. (org.apache.kafka.clients.admin.KafkaAdminClient)

GROUP                          TOPIC                          PARTITION  NEW-OFFSET     
my-first-group                 fourth_topic                   0          0              
my-first-group                 fourth_topic                   1          0              
my-first-group                 fourth_topic                   2          0             
```

