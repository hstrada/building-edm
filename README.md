# Building Event-Driven Microservices: Leveraging Organizational Data at Scale

## Book Reference

https://www.amazon.com.br/Building-Event-Driven-Microservices-Adam-Bellemare/dp/1492057894

## Docker

```bash
docker-compose up -d
```

## Registrando Connector

```curl
curl --request POST \
  --url http://localhost:8083/connectors/ \
  --header 'Content-Type: application/json' \
  --data '{
    "name": "inventory-connector",  
    "config": {  
			"connector.class": "io.debezium.connector.mysql.MySqlConnector",
			"tasks.max": "1",  
			"database.hostname": "mysql",  
			"database.port": "3306",
			"database.user": "root",
			"database.password": "debezium",
			"database.server.id": "184054",  
			"topic.prefix": "dbserver1",  
			"database.include.list": "inventory",  
			"schema.history.internal.kafka.bootstrap.servers": "kafka:9092",  
			"schema.history.internal.kafka.topic": "schema-changes.inventory"  
    }
}'
```

### Connectors

*Listing*

```curl
curl --request GET \
  --url http://localhost:8083/connectors/
```

*Expected Result*

```json
["inventory-connector"]
```

*Inventory*

```curl
curl --request GET \
  --url http://localhost:8083/connectors/inventory-connector
```

*Expected Result*

```json
{
	"name": "inventory-connector",
	"config": {
		"connector.class": "io.debezium.connector.mysql.MySqlConnector",
		"database.user": "root",
		"topic.prefix": "dbserver1",
		"schema.history.internal.kafka.topic": "schema-changes.inventory",
		"database.server.id": "184054",
		"tasks.max": "1",
		"database.hostname": "mysql",
		"database.password": "debezium",
		"name": "inventory-connector",
		"schema.history.internal.kafka.bootstrap.servers": "kafka:9092",
		"database.port": "3306",
		"database.include.list": "inventory"
	},
	"tasks": [
		{
			"connector": "inventory-connector",
			"task": 0
		}
	],
	"type": "source"
}
```

## Updating Value -> MySQL

> UPDATE customers SET first_name='Anne' WHERE id=1004;

*Inside docker*

> ./kafka-console-consumer.sh --bootstrap-server kafka:9092 --topic dbserver1.inventory.customers

{"before":{"id":1004,"first_name":"Anne Marie","last_name":"Kretchmar","email":"annek@noanswer.org"},"after":{"id":1004,"first_name":"Anne","last_name":"Kretchmar","email":"annek@noanswer.org"},"source":{"version":"2.1.0.Final","connector":"mysql","name":"dbserver1","ts_ms":1672097856000,"snapshot":"false","db":"inventory","sequence":null,"table":"customers","server_id":223344,"gtid":null,"file":"mysql-bin.000003","pos":802,"row":0,"thread":204,"query":null},"op":"u","ts_ms":1672097856037,"transaction":null}

docker run -it --rm --network building-edm_default --name watcher --link zookeeper:zookeeper --link kafka:kafka quay.io/debezium/kafka:2.1 watch-topic -a -k dbserver1.inventory.customers
