#  kafka-mirrormaker-azure-tutorial

This tutorial is a simple step by step instruction to reach the following goal:

*   Install a on-prem Apache Kafka
*   Configure the Apache Kafka tool mirrormaker
*   Send message produced on-prem to Microsoft Azure Event Hub

## Step 1

**Prerequisite:** 

*   Ubuntu 20.04 (on Windows 10 with WSL 2)
*   MS Visual Studio Code
*   Microsoft Azure subscription

## Step 2

**Install java on Ubuntu 20.04:**

> $ sudo apt update
> 
> $ sudo apt install openjdk-8-jdk -y
> 
> $ java -version

![](https://user-images.githubusercontent.com/51634515/108631882-74cfd680-746c-11eb-8f1f-8ba0fa6cb4b2.png)

> $ export JAVA\_HOME=/usr/lib/jvm/java-8-openjdk-amd64
> 
> $ export PATH=$PATH:$JAVA\_HOME/bin

## Step 3

**Install Apache Kafka on Ubuntu 20.04:**

> $ mkdir kafka
> 
> $ cd kafka
> 
> $ wget [https://downloads.apache.org/kafka/2.7.0/kafka_2.13-2.7.0.tgz](https://downloads.apache.org/kafka/2.7.0/kafka_2.13-2.7.0.tgz)
> 
> $ tar -zxf kafka\_2.13-2.7.0.tgz
> 
> $ sudo mv kafka\_2.13-2.7.0 /usr/local/kafka
> 
> $ sudo mkdir /tmp/kafka-logs
> 
> $ sudo mkdir /tmp/zookeeper

## Step 4

**Start Apache Zookeeper on Ubuntu 20.04:**

> $ sudo /usr/local/kafka/bin/zookeeper-server-start.sh /usr/local/kafka/config/zookeeper.properties

## Step 5

**Start Apache Kafka Server on Ubuntu 20.04:**

> $ sudo /usr/local/kafka/bin/kafka-server-start.sh /usr/local/kafka/config/server.properties

## Step 6

**Create Kafka topic on Ubuntu 20.04:**

> $ /usr/local/kafka/bin/kafka-topics.sh --create --topic quickstart-events --bootstrap-server localhost:9092
> 
> $ /usr/local/kafka/bin/kafka-topics.sh --create --topic **mymachine** \--bootstrap-server localhost:9092
> 
> $ /usr/local/kafka/bin/kafka-topics.sh --list --bootstrap-server  localhost:9092

![](https://user-images.githubusercontent.com/51634515/108633264-9aaca980-7473-11eb-8798-59a2a2c4089d.png)

## Step 7

**Microsoft Azure Event-Hub:**

Go to your Microsoft Azure subscription.

Create a resource

![](https://user-images.githubusercontent.com/51634515/108899070-47c02700-7618-11eb-92d7-1d4e53b602dc.png)

![](https://user-images.githubusercontent.com/51634515/108899264-8524b480-7618-11eb-8348-2f98ff3cfc0f.png)

![](https://user-images.githubusercontent.com/51634515/108899506-c9b05000-7618-11eb-9b0d-ec28758e5861.png)

Create

![](https://user-images.githubusercontent.com/51634515/108900276-bc479580-7619-11eb-8b20-6c586b28af66.png)

You have to create a "Standard" tier Event Hubs namespace: In this case the Kafka endpoint for the namespace is automatically enabled.

Next: Features >  Next: Tags > Next: Review + create > Create

![](https://user-images.githubusercontent.com/51634515/108902321-4264db80-761c-11eb-8eb4-609d9b64ff3f.png)

Go to resource

![](https://user-images.githubusercontent.com/51634515/108902458-67f1e500-761c-11eb-8481-3e1762524ca7.png)

Click on 

![](https://user-images.githubusercontent.com/51634515/108902538-87890d80-761c-11eb-9ecf-e6db2b5b5ce9.png)

and copy the "**Connection string–primary key**" 

e.g. Endpoint=sb://kafkaazure.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=**\[your key\]**

for later use.

## Step 8

**Configure MirrorMaker on Ubuntu 20.04:**

> $ cd /usr/local/kafka/config
> 
> $ code .

Visual Studio Code opens. Create File / New File and enter the following configuration:

> bootstrap.servers=localhost:9092
> 
> exclude.internal.topics=true
> 
> client.id=mirror\_maker\_consumer  
> group.id=mirror\_maker\_consumer

File / Save as sourceCluster1Consumer.config

![](https://user-images.githubusercontent.com/51634515/108904327-a5577200-761e-11eb-9138-18e4ec25d09f.png)

Create File / New File and enter the following configuration:

> bootstrap.servers=kafkaazure.servicebus.windows.net:9093  
> security.protocol=SASL\_SSL  
> sasl.mechanism=PLAIN  
> sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="Endpoint=sb://kafkaazure.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=**\[your key\]**;
> 
> acks=1
> 
> batch.size=50
> 
> client.id=mirror\_maker\_test\_producer

File / Save as azureClusterProducer.config

![](https://user-images.githubusercontent.com/51634515/108905062-89a09b80-761f-11eb-8f86-3d5e22213007.png)

## Step 9

**Start MirrorMaker on Ubuntu 20.04:**

> $ sudo /usr/local/kafka/bin/kafka-mirror-maker.sh --consumer.config /usr/local/kafka/config/sourceCluster1Consumer.config --num.streams 1 --producer.config /usr/local/kafka/config/azureClusterProducer.config --whitelist=".\*"

## Step 10

**Start Kafka producer on Ubuntu 20.04:**

> $ sudo /usr/local/kafka/bin/kafka-console-producer.sh --topic mymachine --bootstrap-server localhost:9092

Enter the following value:

> {"sensor\_id": 1,"ltime": 1613204245,"temp": 5.5,"status": 1}

![](https://user-images.githubusercontent.com/51634515/108909932-7abce780-7625-11eb-9f79-d2a494a01a45.png)

## Step 11

**Start Kafka consumer on Ubuntu 20.04:**

> $ sudo /usr/local/kafka/bin/kafka-console-consumer.sh --topic mymachine --from-beginning --bootstrap-server localhost:9092

![](https://user-images.githubusercontent.com/51634515/108909983-87414000-7625-11eb-912b-150d8416f73f.png)

## Step 12

**Microsoft Azure Event-Hub:**

In the Event Hubs Namespace go to Event Hubs:

An Event Hub with the name **mymachine** ist generated automatically. Click on the mymachine Event Hub:

![](https://user-images.githubusercontent.com/51634515/108910267-e99a4080-7625-11eb-9fa3-99f290f76a91.png)

Go to mymachine Event Hubs Instance. 

Congratulations: Your first on-prem produced Kafka message has been transmitted to Microsoft Azure as you can see for example in "Messages".

![](https://user-images.githubusercontent.com/51634515/108910548-51508b80-7626-11eb-9d8c-a9ed336ddba7.png)
