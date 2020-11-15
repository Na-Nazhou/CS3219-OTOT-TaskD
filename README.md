# CS3219-OTOT-TaskD

## Demo1: Pub-Sub Messaging with Kafka Cluser

1. Make sure Docker is started

2. Change directory to `/kafka-cluster` and copy the content of `.env.example` to `.env`

```bash
cd kafka-cluster
```

3. Run the following command to find the IP address of the Docker host

```bash
docker run alpine /sbin/ip route|awk '/default/ { print $3 }'
```

Reference: https://stackoverflow.com/questions/22944631/how-to-get-the-ip-address-of-the-docker-host-from-inside-a-docker-container

4. Copy the IP address to the HOST field of `.env`.

5. Spin up the Docker containers

```bash
docker-compose up
```

6. Wait for initialization to complete

7. Start a new terminal, run the following command to start a producer which connects to node `kafka1`, you can replace it with `kafka2` or `kafka3`. Type a message and press `Enter`

```bash
docker run --interactive --network kafka-cluster_default confluentinc/cp-kafkacat kafkacat -b kafka1:9092 -t new_topic -P
```

To stop the producer, type `ctrl-D`

8. Start another terminal, run the following command to start a consumer which connects to node `kafka1`, you can replace it with `kafka2` or `kafka3`. You should see the previous message sent by the producer

```bash
docker run --tty --network kafka-cluster_default confluentinc/cp-kafkacat kafkacat -b kafka2:9092 -t new_topic -C
```

To stop the consumer, type `ctrl-C`

9. Run the following command to stop the Docker containers

```bash
docker-compose down
```

## Demo2: Failover

1. Repeat step 1 - 6 in **Demo1** to spin up the Docker containers

2. List all topics, find the master node of topic `new_topic`

```bash
docker run --tty --network kafka-cluster_default confluentinc/cp-kafkacat kafkacat -b kafka1:9092 -L
```

3. Run the following command to kill the master node

```bash
docker kill [master Kafka node container]
```

4. Run the following command to verify that a new leader has been elected

```bash
docker run --tty --network kafka-cluster_default confluentinc/cp-kafkacat kafkacat -b kafka1:9092 -L
```

5. Repeat step 7 - 8 in **Demo1** to start a producer and a consumer (Note: do not connect to the previous master node)

6. Verify that the producer can still send message to the topic and the consumer can consume the sent messages as before, i.e. the messaging functionality still works as before.

7. Run the following command to stop the Docker containers

```bash
docker-compose down
```
