# CS3219-OTOT-TaskD

## Demo 1: Pub-Sub messaging system using Apache Kafka

1. Clone the repository and make sure Docker is started.

2. Change directory to `/kafka-cluster`.

```bash
cd kafka-cluster
```

3. Create a file named `.env` in this directory and copy the content of `.env.example` to `.env`.

4. Run the following command to find the IP address of the Docker host

```bash
docker run alpine /sbin/ip route|awk '/default/ { print $3 }'
```

Reference: https://stackoverflow.com/questions/22944631/how-to-get-the-ip-address-of-the-docker-host-from-inside-a-docker-container

5. Copy the IP address printed in the terminal to the `HOST` field of `.env`.

6. Start the Kafka cluster by running the following command:

```bash
docker-compose up
```

7. Wait for the initialization to complete and this might take a while. The initialization should be completed when the default topic (‘new_topic’) is created and initialised properly.

8. Start a new terminal, run the following command to create a producer that connects to node `kafka1` and publishes messages to `new_topic`. You can connect to any of the Kafka nodes, i.e. you can replace `kafka1` in the following command with `kafka2` or `kafka3` if you want. You can then start to type a message and press `Enter` when you finish typing.

```bash
docker run --interactive --network kafka-cluster_default confluentinc/cp-kafkacat kafkacat -b kafka1:9092 -t new_topic -P
```

9. Start another new terminal, run the following command to create a consumer that connects to node `kafka1` and subscribes to `new_topic`. Similarly, you can connect to any of the Kafka nodes, i.e. you can replace `kafka1` in the following command with `kafka2` or `kafka3` if you want. You should now see the previous message sent by the producer.

```bash
docker run --tty --network kafka-cluster_default confluentinc/cp-kafkacat kafkacat -b kafka1:9092 -t new_topic -C
```

10. Switch back to the terminal with the producer running, type another message and press Enter when you finish. Switch to the terminal with the consumer running, the new message should show up in the terminal. This shows that the pub-sub messaging works in real-time.

11. Press `ctrl-D` to stop the producer.

12. Press `ctrl-C` to stop the consumer.

13. Run the following command to stop the Kafka cluster:

```bash
docker-compose down
```

## Demo 2: Management of the failure of the master node

1. Repeat **step 1 - 7 in Demo 1** to set up the Kafka cluster.

2. Run the following command to list the metadata of all topics. You can connect to any of the Kafka nodes, i.e. you can replace `kafka1` in the following command with `kafka2` or `kafka3` if you want.

```bash
docker run --tty --network kafka-cluster_default confluentinc/cp-kafkacat kafkacat -b kafka1:9092 -L
```

3. Look for topic `new_topic`, note down the broker ID of its leader.

4. Repeat **step 8 – 10 in Demo 1** to publish a few messages to the topic `new_topic`. For the purpose of this demo, connect to the master node. Later we will show that all these messages are not lost even the master node dies.

5. Start a new terminal and run the following command to kill the master node (i.e. leader) of `new_topic`. If the broker ID of `new_topic`’s leader in the previous step is `1`, replace [container name] with `kafka1`.

```bash
docker kill [container name]
```

You should see errors in the producer and consumer connecting to this node as the node gets killed. You can kill the producer and consumer with commands specified in **step 11 - 12 in Demo 1**.

6. Run the following command to list the metadata of all topics again. Note that you can connect to any of two remaining nodes but you cannot connect to the killed node any more. You should be able to see that the killed node is not present in the list of brokers and a new master node (i.e. leader) has been elected for `new_topic` to replace the previous master node.

```bash
docker run --tty --network kafka-cluster_default confluentinc/cp-kafkacat kafkacat -b kafka1:9092 -L
```

7. Repeat **step 8 - 10 in Demo 1** to verify that the pub-sub messaging functionality works as before. Note that you should only connect to the alive nodes. Moreover, the previous messages published in **step 4 in Demo 2** are still displayed in the terminal with consumer running.

8. Repeat **step 11 – 13 in Demo 1** to stop the producer, consumer and the Kafka cluster.

## References

- https://hub.docker.com/_/zookeeper
- https://github.com/wurstmeister/kafka-docker/wiki/Connectivity
