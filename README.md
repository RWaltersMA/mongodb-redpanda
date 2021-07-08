

# MongoDB Kafka Connector and RedPanda

This docker compose spins up the following:
- MongoDB single node replica set
- Kafka Connect
- RedPanda 

In this example, we will spin up a RedPanda service, configure a single node MongoDB Replica set and configure the MongoDB Connector for Apache Kafka connector.  The connector will be configured to read from the Stocks.StockData collection and write data to a RedPanda topic.  The connection will be configured as a sink to read from the RedPanda topic and write data into the MongoDB collection Stocks.StockDataCopy.

To launch execute the shell script `sh run.sh`.

If successful you'll see the summary of the services running

```
==============================================================================================================

The following services are running:

MongoDB 1-node replica set on port 27017
Redpanda on 8082 (Redpanda proxy on 8083)
Kafka Connect on 8083

Status of kafka connectors:
sh status.h

To tear down the environment and stop these serivces:
docker-compose-down -v

==============================================================================================================
```

## Configure the source connector

curl -X POST -H "Content-Type: application/json" -d @source.json  http://localhost:8083/connectors

## Configure the sink connector

curl -X POST -H "Content-Type: application/json" -d @sink.json  http://localhost:8083/connectors  

## Write some data to the source collection
docker-compose exec mongo1 /usr/bin/mongo --host "mongodb://0.0.0.0/Stocks?replicaSet=rs0"  --eval '''db.StockData.insertOne({'test':123123})'''

## Read the data in the sink collection
docker-compose exec mongo1 /usr/bin/mongo --host "mongodb://0.0.0.0/Stocks?replicaSet=rs0"  --eval '''db.StockDataCopy.find().pretty()''' 