# Kafka Security

The course wants to use EC2 to setup Kafka.
I think is unnecessary and I can do it locally, so I'm using docker compose to setup my kafka env

## Running

Simply use `docker compose up -d`. It should pull and run both zookeeper and kafka.

The compose also includes [kcat](https://github.com/edenhill/kcat) already configured to use as consumer and producer for testing.

Run it through compose as you would use it normally:

```bash
# Create a topic
# You can likely skip this though
docker compose exec kafka kafka-topics.sh --bootstrap-server kafka:9092 --topic trashcan --create --if-not-exists

# Produce
docker compose exec -it kafka kafka-console-producer.sh --bootstrap-server kafka:9092 --producer.config /opt/bitnami/kafka/config/producer.properties --topic trashcan

# Consume
docker compose exec kafka kafka-console-consumer.sh --topic trashcan --bootstrap-server kafka:9092 --consumer.config /opt/bitnami/kafka/config/consumer.properties --from-beginning

## Using kcat

# List topics
docker compose run --rm kcat -L

# Write to topic
docker compose run --rm -T kcat -t trashcan -K: -P <<MESSAGES
1:{"id":1,"value":7657}
2:{"id":2,"value":13}
MESSAGES

# Consume
docker compose run --rm kcat -C -t trashcan
```

### kafka-ui

The compose also includes a UI in `localhost:8080` , its not up by default, to use it run:

```bash
docker compose up kafka-ui -d
```

You can use it to consume/produce messages if command line is not your thing.

## Original readme

Setup and configure Security functionality in Kafka.  
This repo uses OpenSource Apache Kafka deployed on EC2 (AWS).  

[goto setup Kafka](./Setup-Kafka)

[goto setup Kerberos](./Setup-Kerberos)

[goto setup SSL](./Setup-SSL)

