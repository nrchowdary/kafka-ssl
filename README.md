# kafka-ssl
# docker-compose with external network



docker network create kafka --attachable
############################################
---
version: "3"
services:
  zookeeper:
    image: 'bitnami/zookeeper:latest'
    networks:
      - kafka
    restart: unless-stopped
    ports:
      - '2181:2181'
    environment:
      - ALLOW_ANONYMOUS_LOGIN=yes
  kafka:
    image: 'bitnami/kafka:latest'
    networks:
      - kafka
    restart: unless-stopped
    user: root
    ports:
      - '9092:9092'
    volumes:
      - $PWD/kafka-persistence:/bitnami/kafka
    environment:
      - KAFKA_BROKER_ID=1
      - KAFKA_LISTENERS=PLAINTEXT://:9092
      - KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://127.0.0.1:9092
      - KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181
      - ALLOW_PLAINTEXT_LISTENER=yes
    depends_on:
      - zookeeper
networks:
  kafka:
      external: true

######################################
---
version: "3"
services:
  zookeeper:
    image: 'bitnami/zookeeper:latest'
    networks:
      - kafka
    restart: unless-stopped
    ports:
      - '2181:2181'
    environment:
      - ALLOW_ANONYMOUS_LOGIN=yes
  kafka:
    image: 'bitnami/kafka:latest'
    networks:
      - kafka
    restart: unless-stopped
    user: root
    ports:
      - '9092:9092'
      - '9093:9093'
    volumes:
      - $PWD/kafka-persistence:/bitnami/kafka
    environment:
      - KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper:2181
      - ALLOW_PLAINTEXT_LISTENER=yes
      - KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CLIENT:PLAINTEXT,EXTERNAL:PLAINTEXT
      - KAFKA_CFG_LISTENERS=CLIENT://:9092,EXTERNAL://:9093
      - KAFKA_CFG_ADVERTISED_LISTENERS=CLIENT://kafka:9092,EXTERNAL://localhost:9093
      - KAFKA_INTER_BROKER_LISTENER_NAME=CLIENT
    depends_on:
      - zookeeper
networks:
  kafka:
      external: true
###################################################

docker-compose -f docker-compose.yml up -d
docker images
docker ps
docker exec -it e7d217c83d0e /bin/bash

# Important Commands
# Topic Creation
kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic test
# List Topic
kafka-topics.sh --list --bootstrap-server localhost:9092
# Describe a topic
kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic mytopic
# Creating Producers
kafka-console-producer.sh --bootstrap-server localhost:9092 --topic testtopic
# Creating Consumers
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic testtopic --from-beginning
# Listing Consumers Groups
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list
# Describe Consumers Groups
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group GROUPNAME
# Consumer Offsets  (Consumer information stored in consumer offsets by kafka)
kafka-topics.sh --bootstrap-server localhost:9092 --list

--------------------------------------
# https://github.com/bitnami/bitnami-docker-kafka
