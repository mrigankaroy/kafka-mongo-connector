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

Setup kafka with below command [docker-compose.yml](https://github.com/mrigankaroy/kafka-mongo-connector/blob/main/docker-compose.yml)

```sh
$ docker compose up -d
```

## 4. Debezium source connector Setup

We need to install debezium source connector

```sh
$ docker run -it --rm --name connect --network library-network -p 8083:8083 -e GROUP_ID=1 -e CONFIG_STORAGE_TOPIC=my_connect_configs -e OFFSET_STORAGE_TOPIC=my_connect_offsets -e STATUS_STORAGE_TOPIC=my_connect_statuses -e BOOTSTRAP_SERVERS=kafka:9092 --link zookeeper:zookeeper --link kafka:kafka --link postgres:postgres --link mongo:mongo debezium/connect:1.3

```

### 5. Testing

Create new mongo database

```sh
$ docker exec -it mongo mongo

$ use user-database
$ db.test_data.insert({"name":"Test data2"})
```

Create a source connector [Ref](https://debezium.io/documentation/reference/connectors/mongodb.html#mongodb-connector-properties)

```sh
curl --location --request POST 'http://localhost:8083/connectors' \
--header 'Content-Type: application/json' \
--data-raw '{
  "name": "test-connector", 
  "config": {
    "connector.class": "io.debezium.connector.mongodb.MongoDbConnector", 
    "mongodb.hosts": "rs0/mongo:27017", 
    "mongodb.name": "libraryMongo"
  }
}'
```

Check newly created topic in kafka

```sh
$ bin/kafka-topics.sh --list --zookeeper zookeeper:2181

```

Monitor messages in kafka topic

```sh
$ bin/kafka-console-consumer.sh --topic libraryMongo.user-database.test_data --from-beginning --bootstrap-server kafka:9092

```
