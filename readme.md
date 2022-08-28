# Kafka Security

The course wants to use EC2 to setup Kafka.
I think is unnecessary and I can do it locally, so I'm using docker compose to setup my kafka env.
The original files were moved to [./original_course_files](./original_course_files/).

## Requirements

There are no requirements when running the version `plain` of compose as those have no encryption.

However, the default compose uses Kafka with TLS only, so we need to generate a jks truststore and keystore for the server and also clients.

Some default keys are stored in this repo as they are used only for local/dev usage, as well as their passwords.
If you need/want to generate a new root ca and keys you can use the following commands:

```bash
# Install Java as keytool comes with the jre and is required for the key generation. In ubuntu you can do:
apt install default-jre

# This scripts will generate all the necessary files for the Kafka server ( and some clients )
# When asked for the FQDN ( or your name ) make sure to use the server hostname as defined in the docker-compose.yaml.
./kafka-generate-ssl.sh

# Once the files are generated you can start the server and kafka-ui.
# To use kcat you will need to convert to PEM format.
# First convert to P12
keytool -importkeystore -srckeystore ./keystore/kafka.keystore.jks -destkeystore ./keystore/kafka.keystore.p12 -srcstoretype jks -deststoretype pkcs12

# From the P12 file, grab the private key
openssl pkcs12 -in keystore/kafka.keystore.p12 -nodes -nocerts -out ./keystore/private_key.pem

# Now grab the certificates
openssl pkcs12 -in ./keystore/kafka.keystore.p12 -nodes -nokeys -out ./keystore/keystore.pem
```

## Running

Simply use `docker compose up -d`. It should pull and run both zookeeper and kafka.

The compose also includes [kcat](https://github.com/edenhill/kcat) already configured to use as consumer and producer for testing.

Run it through compose as you would use it normally:

```bash
# Create a topic
# You can likely skip this though
docker compose exec kafka kafka-topics.sh --bootstrap-server kafka.local:9093 --topic trashcan --create --if-not-exists

# Produce
docker compose exec -it kafka kafka-console-producer.sh --bootstrap-server kafka.local:9093 --producer.config /opt/bitnami/kafka/config/producer.properties --topic trashcan

# Consume
docker compose exec kafka kafka-console-consumer.sh --topic trashcan --bootstrap-server kafka.local:9093 --consumer.config /opt/bitnami/kafka/config/consumer.properties --from-beginning

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

The compose also includes a UI in `localhost:8080`, its not up by default, to use it run:

```bash
docker compose up kafka-ui -d
```

You can use it to consume/produce messages if command line is not your thing.
