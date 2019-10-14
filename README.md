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

## Authors 
* **Firas Esbai** - *Initial work* - [Firas Esbai](https://github.com/firasesbai) 
## Licence 
This project is licenced under the [Apache-2.0 Licence](https://www.apache.org/licenses/LICENSE-2.0) - see the [LICENCE](LICENCE.md) file for details.
