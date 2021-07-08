# MongoDB Kafka Connector and RedPanda

This docker compose spins up the following:
- MongoDB single node replica set
- Kafka Connect
- RedPanda 

In this example, we will spin up a [RedPanda](https://vectorized.io/redpanda) streaming service, configure a single node MongoDB Replica set and configure the MongoDB Connector for Apache Kafka connector.  The connector will be configured to read from the Stocks.StockData collection and write data to a RedPanda topic.  The connection will be configured as a sink to read from the RedPanda topic and write data into the MongoDB collection Stocks.StockDataCopy.

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

## View the status

`sh status.sh`

The status script is a quick way to enumerate the RedPanda topics and the connectors that are configured.  If you do not see any data wait a few seconds for the services to finish starting up.  Once ready, this command will show something similar to the following:
```
Kafka topics:

[
  "docker-connect-offsets",
  "docker-connect-status",
  "docker-connect-configs"
]

The status of the connectors:


Currently configured connectors

[]


Version of MongoDB Connector for Apache Kafka installed:

{"class":"com.mongodb.kafka.connect.MongoSinkConnector","type":"sink","version":"1.5.1"}
{"class":"com.mongodb.kafka.connect.MongoSourceConnector","type":"source","version":"1.5.1"}
```

Once our services are ready, we can configure the source and sink.


## Configure the source connector

The source.json file contains a source configuration for the MongoDB Connector for Apache Kafka.  This configuration tells Kafka Connect to read from the Stocks.StockData collection and write data into the RedPanda "Stocks.StockData" topic.

`curl -X POST -H "Content-Type: application/json" -d @source.json  http://localhost:8083/connectors`


## Configure the sink connector

The sink.json file contains a sink configuration for the MongoDB Connector for Apache Kafka.  This configuration tells Kafka Connect to read from the "Stocks.StockData" RedPanda topic and write to the MongoDB Collection, "Stocks.StockDataCopy".

`curl -X POST -H "Content-Type: application/json" -d @sink.json  http://localhost:8083/connectors`  


## Write some data to the source collection

To insert some sample data, execute the following:

`docker-compose exec mongo1 /usr/bin/mongo --host "mongodb://0.0.0.0/Stocks?replicaSet=rs0"  --eval '''db.StockData.insertOne({'test':123123})''' `


## Read the data in the sink collection

To read data from the MongoDB collection, execute the following:

`docker-compose exec mongo1 /usr/bin/mongo --host "mongodb://0.0.0.0/Stocks?replicaSet=rs0"  --eval '''db.StockDataCopy.find().pretty()''' `

If successful you'll see the test document we created in the source collection.

## View the status

`sh status.sh`

The status script is a quick way to enumerate the RedPanda topics and the connectors that are configured.  At this point, you should see the source and sink connectors you configured as well as the Stock.StockData topic.
