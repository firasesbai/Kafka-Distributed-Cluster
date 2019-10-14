# Kafka-Cluster-Stack-docker
Apache Kafka is a distributed streaming platform. Additional tools and services have been created around it to build and manage an Event Streaming Platform.<br/>
This will provide a template for real world distributed kafka cluster deployment. Components will be in a *physically distributed* and *distinct servers*. Frameworks and UIs from **Confluent Inc** and **lenses.io** are as well integrated.     
## Getting started 
These instructions will get you a copy of the project up and running on your local machine for development and testing purposes.
### Prerequisites
#### - docker
Get the Docker engine community edition following the steps in the official documentation [here](https://docs.docker.com/install/linux/docker-ce/ubuntu/).<br/>
This Setup was tested on Ubuntu 18.04. 
#### - docker-compose 
Install docker compose which relies on docker engine following the steps [here](https://docs.docker.com/compose/install/).
### Setup 
To deploy services, run in each node of the cluster the following command after changing to the directory where you have copied the appropriate compose file with the necessary changes: 
```
docker-compose up -d 
```
## Distributed Cluster Setup
### Deploy services
Replace the following settings in the compose files with the corresponding IP addresses in each node of the cluster:
- **zookeeper-1**, **zookeeper-2** and **zookeeper-3** in **ZOO_SERVERS** environment variable.
- **IP_ADDRESS_SERVER_1**, **IP_ADDRESS_SERVER_2** and **IP_ADDRESS_SERVER_3** in **KAFKA_ADVERTISED_LISTENERS**, **KAFKA_ZOOKEEPER_CONNECT**, **CONNECT_BOOTSTRAP_SERVERS**, **CONNECT_ZOOKEEPER_CONNECT** environment variables.
### Tests
**1-** Using **Kafkacat** utility (see the following [link](https://github.com/edenhill/kafkacat) for installation) and by running the following command on one of the nodes (change the IP Address and Port number based on the chosen node) we can check that our cluster of 3 brokers was created successfully:
```
kafkacat -b IP_Address_of_the_broker_node:Port_Number -L 
```
The output of this command is a list of 3 brokers with the corresponding IP Addresses and some topics that were automatically created.<br/> <br/>
**2-** We will create a simple source file stream connector to test kafka-connect service.
<br/><br/>
Two methods are possible: 
<br/><br/>
  **2.1- Standalone mode**
- Create a new kafka topic
```
docker exec -it kafka-1 bash 
```
```
kafka-topics --create --topic test-standalone --partitions 3 --replication-factor 1 --zookeeper zookeeper-1:2181
```
- Create the files [worker.properties](worker.properties) and [file-stream-standalone.properties](file-stream-standalone.properties)
- Copy the previously created files inside the kafka-connect container 
```
docker cp worker.properties kafka-connect-1:/
```
```
docker cp file-stream-standalone.properties kafka-connect:/
```
 
  **2.2- Rest API**

## Authors 
* **Firas Esbai** - *Initial work* - [Firas Esbai](https://github.com/firasesbai) 
## Licence 
This project is licenced under the [Apache-2.0 Licence](https://www.apache.org/licenses/LICENSE-2.0) - see the [LICENCE](LICENCE.md) file for details.
