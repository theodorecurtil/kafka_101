We show how to run a local Kafka cluster using [Docker](https://www.docker.com/) containers. We will also show how to produce and consume events from Kafka using the CLI.

## The Infrastructure

The cluster is set up using [Confluent](https://hub.docker.com/u/confluentinc) images. In particular, we set up 4 services:

1. Zookeeper
2. Kafka Server (the broker)
3. Confluent Schema Registry (for use in later article...)
4. Confluent Control Center (the UI to interact with the cluster)

Note that Kafka 3.4 introduces the capability to move a Kafka cluster from Zookeeper to KRaft mode. At the time this article is written, Confluent still has not released the new Docker image with Kafka 3.4. As such, we still use Zookeeper in this tutorial. For a discussion on how about Zookeeper and KRaft, refer to [this article](https://www.confluent.io/blog/kafka-without-zookeeper-a-sneak-peek/).

### Zookeeper

Confluent configurations available [here](https://docs.confluent.io/platform/current/installation/docker/config-reference.html#zk-configuration).

We need to tell Zookeper on which port to listen to connections from clients, in our case Apache Kafka. This is configured with the key `ZOOKEEPER_CLIENT_PORT`. Once this port is chosen, expose the corresponding port in the container. `ZOOKEEPER_TICK_TIME` in milliseconds, is the basic unit of time used by Zookeeper.

```
version: '3.3'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:7.2.1
    hostname: zookeeper
    container_name: zookeeper
    ports:
      - "2181:2181"
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
  broker:
    ...
  schema-registry:
    ...
  control-center:
    ...
```

### Kafka Server

```
version: '3.3'
services:
  zookeeper:
    ...
  broker:
    image: confluentinc/cp-server:7.2.1
    hostname: broker
    container_name: broker
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
      - "9101:9101"
      - "29092:29092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: 'zookeeper:2181'
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_LOG4J_LOGGERS: org.apache.zookeeper=ERROR,org.apache.kafka=ERROR, kafka=ERROR, kafka.cluster=ERROR,kafka.controller=ERROR, kafka.coordinator=ERROR,kafka.log=ERROR,kafka.server=ERROR,kafka.zookeeper=ERROR,state.change.logger=ERROR
      KAFKA_LOG4J_ROOT_LOGLEVEL: ERROR
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://broker:29092,PLAINTEXT_HOST://localhost:9092
      KAFKA_METRIC_REPORTERS: io.confluent.metrics.reporter.ConfluentMetricsReporter
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      KAFKA_JMX_PORT: 9101
      KAFKA_JMX_HOSTNAME: localhost
      KAFKA_CONFLUENT_SCHEMA_REGISTRY_URL: http://schema-registry:8081
      CONFLUENT_METRICS_REPORTER_BOOTSTRAP_SERVERS: broker:29092
      CONFLUENT_METRICS_REPORTER_TOPIC_REPLICAS: 1
      CONFLUENT_METRICS_ENABLE: 'true'
      CONFLUENT_SUPPORT_CUSTOMER_ID: 'anonymous'
      KAFKA_CONFLUENT_LICENSE_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_CONFLUENT_BALANCER_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
  schema-registry:
    ...
  control-center:
    ...
```