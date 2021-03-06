# Kafka-Distributed-Cluster
Apache Kafka is a distributed streaming platform. Additional tools and services have been created around it to build and manage an Event Streaming Platform.<br/>
This will provide a template for real world distributed kafka cluster deployment. Components will be in a *physically distributed* and *distinct servers*. Since monitoring your kafka cluster is tedious yet a very important task, instructions for integrating frameworks and UIs from **Confluent Inc**, **lenses.io** and other Open source projects are also provided to facilitate the management and configuration of the cluster.     
## Getting started 
These instructions will get you a copy of the project up and running for development and testing purposes.
### Prerequisites
#### - docker
Get the Docker engine community edition following the steps in the official documentation [here](https://docs.docker.com/install/linux/docker-ce/ubuntu/).<br/>
This Setup was tested on Ubuntu 18.04. 
#### - docker-compose 
Install docker compose which relies on docker engine following the steps [here](https://docs.docker.com/compose/install/).
### Docker Images Version
- Zookeeper: 3.5
- Kafka: 2.3.0 (Confluent 5.3.1)
- kafka connect: Confluent 5.3.1
- Schema-registry: Confluent 5.3.1
- KSQL-server: Confluent 5.3.1
- KSQL-CLI: Confluent 5.3.1
- REST-Proxy: Confluent 5.3.1
- Kafka-topics-ui: 0.9.4
- kafka-connect-ui: 0.9.4
- Schema-registry-ui: 0.9.4
- Zoonavigator: 0.7.1
### Setup 
To deploy services, run in each node of the cluster the following command after changing to the directory where you have copied the appropriate compose file with the necessary changes: 
```
docker-compose up -d 
```
Or you can refer to this [docker-compose.yml](docker-compose.yml) containing all the services. 
## Distributed Cluster Setup
### Deploy services
Replace the following settings in the compose files [here](./distributed_cluster_setup) with the corresponding IP addresses in each node of the cluster:
- **zookeeper-1**, **zookeeper-2** and **zookeeper-3** in **ZOO_SERVERS** environment variable.
- **IP_ADDRESS_SERVER_1**, **IP_ADDRESS_SERVER_2** and **IP_ADDRESS_SERVER_3** in **KAFKA_ADVERTISED_LISTENERS**, **KAFKA_ZOOKEEPER_CONNECT**, **CONNECT_BOOTSTRAP_SERVERS**, **CONNECT_ZOOKEEPER_CONNECT** environment variables.

### Tests
**1- Kafka Cluster** <br/>
Using **Kafkacat utility** (see the following [link](https://github.com/edenhill/kafkacat) for installation) and by running the following command on one of the nodes (change the IP Address and Port number based on the chosen node) we can check that our cluster of 3 brokers was created successfully:
```
kafkacat -b IP_Address_of_the_broker_node:Port_Number -L 
```
The output of this command is a list of 3 brokers with the corresponding IP Addresses and some topics that were automatically created.<br/> <br/>
**2- Kafka Connect** <br/>
We will create a simple source file stream connector to test **kafka-connect** service.<br/><br/>
**2.1- Standalone mode**
- Create a new kafka topic
```
docker exec -it kafka-1 bash 
```
```
kafka-topics --create --topic test-standalone --partitions 3 --replication-factor 1 --zookeeper zookeeper-1:2181
```
- Create the files [worker.properties](./file-stream-connector/standalone_mode/worker.properties) and [file-stream-standalone.properties](./file-stream-connector/standalone_mode/file-stream-standalone.properties) and update them with your servers' IP addresses
- Copy the previously created files inside the **kafka-connect** container using the following commands
```
docker cp worker.properties kafka-connect-1:/
```
```
docker cp file-stream-standalone.properties kafka-connect:/
```
- Run the connector in standalone mode
```
docker exec -it kafka-connect bash 
```
```
connect-standalone worker.properties file-stream-standalone.properties
````
- Start a console consumer in **kafka-1** broker
```
docker exec -it kafka-1 bash 
```
```
kafka-connect-consumer --topic test-standalone --from-beginning --bootstrap-server kafka-1:9092
````
- Create a new file *demo-file.txt* inside the **kafka-connect** container and put some data into it 
```
touch demo-file.txt
```
```
echo "hi" >> demo-file.txt
```
- You should now see your data in the console consumer

**2.2- Distributed mode**
<br/> 
- Create the file [file-stream-distributed.properties](./file-stream-connector/distributed_mode/file-stream-distributed.properties) and update it with your servers' IP addresses</br></br>
- Copy this file inside the **kafka-connect** container
```
docker cp file-stream-distributed.properties kafka-connect:/
```
- Run the connector in distributed mode
````
connect-distributed file-stream-distributed.properties 
````
- Start a console consumer in **kafka-1** broker
```
docker exec -it kafka-1 bash 
```
```
kafka-connect-consumer --topic test-distributed --from-beginning --bootstrap-server kafka-1:9092
```
- Put some data into *demo-file.txt*
```
echo "hi again" >> demo-file.txt
```
- You should now see your data in the console consumer

**2.3- Rest API**
- Run the following command to create a new file stream connector in distributed mode
```
curl -s -X POST -H "Content-Type: application/json" --data '{"name": "file-stream-demo-distributed-2", "config":{"connector.class":"org.apache.kafka.connect.file.FileStreamSourceConnector","key.converter.schemas.enable":"true","file":"demo-file.txt","tasks.max":"1","value.converter.schemas.enable":"true","name":"file-stream-demo-distributed-2","topic":"test-distributed","value.converter":"org.apache.kafka.connect.json.JsonConverter","key.converter":"org.apache.kafka.connect.json.JsonConverter"}}' http://kafka-connect-1:8083/connectors
```
- List connectors available on your worker (you should see your newly created connector *file-stream-demo-distributed-2*)
```
curl -s kafka-connect-1:8083/connector-plugins
```
- Get connector status (it shoud be in state *Running*)
```
curl -s kafka-connect-1:8083/connectors/file-stream-demo-distributed-2/status
```
- Start a console consumer in **kafka-1** broker
```
docker exec -it kafka-1 bash 
```
```
kafka-connect-consumer --topic test-distributed --from-beginning --bootstrap-server kafka-1:9092
```
- Put some data into *demo-file.txt*
```
echo "rest api test" >> demo-file.txt
```
- You should now see your data in the console consumer
## Confluent Schema registry, KSQL and REST Proxy
### Deploy services
Replace the following settings in the extended compose files [here](./confluent_schema_registry_ksql_rest_proxy) **IP_ADDRESS_SERVER_1**, **IP_ADDRESS_SERVER_2** and **IP_ADDRESS_SERVER_3** with the corresponding IP addresses in each node of the cluster 
## Lenses.io UIs
The following components could be deployed in a seperate server dedicated to the monitoring and management of the cluster which we will refer to as ***SERVER_4***
### Topics UI
Add the following service definition with the appropriate IP address and deploy it
```
  kafka-topics-ui:
      image: landoop/kafka-topics-ui:0.9.4
      hostname: kafka-topics-ui
      container_name: kafka-topics-ui
      ports:
        - "8002:8000"
      environment:
        KAFKA_REST_PROXY_URL: "http://IP_ADDRESS_SERVER_1:8082"
        PROXY: "true"
```
Access this url ``http://IP_ADDRESS_SERVER_4:8002`` and you should be prompted with the topics interface 
### Connect UI
Add the following service definition with the appropriate IP addresses and deploy it
```
  kafka-connect-ui:
      image: landoop/kafka-connect-ui:0.9.4
      hostname: kafka-connect-ui
      container_name: kafka-connect-ui
      ports:
        - "8003:8000"
      environment:
        CONNECT_URL: "http://IP_ADDRESS_SERVER_1:8083,http://IP_ADDRESS_SERVER_2:8083,http://IP_ADDRESS_SERVER_3:8083"
        PROXY: "true"
```
Access this url `` http://IP_ADDRESS_SERVER_4:8003`` and you should be prompted with the connect interface 
### Schema Registry UI
Add the following service definition with the appropriate IP addresses and deploy it
```
  schema-registry-ui:
      image: landoop/schema-registry-ui:0.9.4
      hostname: schema-registry-ui
      container_name: schema-registry-ui
      ports:
        - "8001:8000"
      environment:
        CONNECT_URL: "http://IP_ADDRESS_SERVER_1:8081,http://IP_ADDRESS_SERVER_2:8081,http://IP_ADDRESS_SERVER_3:8081"
        PROXY: "true"
```
Access this url `` http://IP_ADDRESS_SERVER_4:8001`` and you should be prompted with the schema registry interface 
## Kafka-Manager
This is a [tool](https://github.com/yahoo/kafka-manager) for managing Apache Kafka.<br/>
Add the following service definition with the appropriate IP addresses and chosen password then deploy it 
```
  kafka-manager:
    image: "qnib/plain-kafka-manager"
    container_name: kafka-manager
    environment:
      ZOOKEEPER_HOSTS: "IP_ADDRESS_SERVER_1:2181,IP_ADDRESS_SERVER_2:2181,IP_ADDRESS_SERVER_3:2181"
      APPLICATION_SECRET: PUT_YOUR_SECRET_HERE
    restart: always
```
Access this url `` http://IP_ADDRESS_SERVER_4:9000`` and you should be prompted with the kafka-manager interface
## Zoonavigator
This is a web-based [ZooKeeper UI](https://github.com/elkozmon/zoonavigator). <br/>
Add the following service definition and deploy it
```
  zoonavigator:
    image: "elkozmon/zoonavigator:0.7.1"
    container_name: zoonavigator
    environment:
      HTTP_PORT: 9001
    restart: unless-stopped
```
Access this url `` http://IP_ADDRESS_SERVER_4:9001`` and you should be prompted with the zoonavigator interface
## Author 
* **Firas Esbai** 
## Licence 
This project is licenced under the [Apache-2.0 Licence](https://www.apache.org/licenses/LICENSE-2.0)
