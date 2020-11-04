# kafka-mongo-connector
Kafka MongoDB source connector example - Docker, Kafka, MongoDB and Debezium source connector

This document will help to setup MongoDB, Apache Kafka, Debezium mongodb source connector on Docker

## 1. Create docker network

```sh
$ docker network create --driver bridge library-network
```
We can check you newly created network lists and status with below commands.

```sh
$ docker network ls
$ docker network inspect library-network
```
## 2. MongoDB Setup

We need to install two mongo db replica instances.

```sh
$ docker run -it -d -p 27017:27017 --name mongo --network=library-network mongo mongod --replSet library-mongodb-replicaset
```

```sh
$ docker run -it -d -p 27018:27017 --name mongo-rp-1 --network=library-network mongo mongod --replSet library-mongodb-replicaset
```

We need to configure replica set

Login to PRIMARY mongodb console

```sh
$ docker exec -it mongo mongo
```
Execute below commands
```sh
$ config={"_id": "library-mongodb-replicaset","members": [{"_id": 0,"host": "mongo:27017"},{"_id": 1,"host": "mongo-rp-1:27017"}]}

$ rs.initiate(config)
```
Login to SECONDARY mongodb console

```sh
$ docker exec -it mongo-rp-1 mongo
```
Execute below commands
```sh
$ rs.secondaryOk()
```

## 3. Kafka Setup

Setup kafka with below command [Details docker-compose.yml]
