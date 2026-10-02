# Apache Kafka

1) Kafka Cluster
   A group of Kafka Brokers working together.

2) Kafka Broker
   A Kafka server/node that stores and serves topic partitions.
   A broker can contain multiple partitions from multiple topics.

3) Topic
   A logical collection/category of messages, divided into partitions.
   Consumers consume messages from its partitions.

4) Partition
   The actual append-only log (WAL-like data structure) where Kafka
   stores messages/records.
   
5) Consumer
   A process/application that reads/consumes messages from topic partitions.

6) Consumer Group
   A collection of consumers that work together to consume messages
   from one or more topics.

7) Producer
   A process/application that publishes records to Kafka topics.

8) Relationship between Partition, Topic, Consumer and Consumer Group

   a. A consumer can read multiple partitions of a topic and can also
      consume from multiple topics.

   b. Within a traditional consumer group, one partition can be assigned
      to only one consumer at a time.

   c. The same partition can be consumed by multiple consumers if they
      belong to different consumer groups.

   d. With Kafka Share Groups (Kafka 4.2+), multiple consumers within
      the same group can consume from the same partition.

9) Kafka Delivery Semantics

   At-most-once
   → Consumer commits offset BEFORE processing.
   → Drawback: Message may be lost.

   At-least-once
   → Consumer commits offset AFTER processing.
   → Drawback: Message may be processed multiple times (duplicates).

   Exactly-once
   → Kafka output + consumed offset are handled atomically
     using Kafka transactions.
   → Idempotent producers + transactions provide Kafka's
     exactly-once semantics.

10) Kafka Acknowledgements

   Producer → Kafka setting.
   Do not confuse with consumer offset commits.

   acks=0
   → Producer doesn't wait for ACK.
   → Fastest, but message loss is possible.

   acks=1
   → Leader ACKs after writing.
   → Leader failure before replication can cause data loss.

   acks=all
   → ACK after required ISR replicas acknowledge.
   → Strongest durability.

11) Kafka Message / Record Structure

   Key + Value + Headers + Timestamp

   Topic + Partition + Offset are Kafka record metadata/location,
   not part of the record payload itself.

12) Important Kafka APIs

   KafkaTemplate → Producer (send)
   @KafkaListener → Consumer
   ConsumerRecord → topic/partition/offset/key/value
   Acknowledgment → Offset commit
   KafkaConsumer → poll/commit/seek
   KafkaAdmin/NewTopic → Topic management

13) Consumer Rebalancing

   When a consumer joins, leaves, or crashes, Kafka redistributes
   partitions among consumers in that consumer group.

   Example:
   P1 → C1
   P2 → C2
   P3 → C3

   C3 goes down → Rebalance

   P1 → C1
   P2 → C2
   P3 → C2

   Rules:
   → One consumer can read multiple partitions.
   → One partition → only one consumer within the same traditional
     consumer group.
   → More consumers than partitions → extra consumers remain idle.
   → More partitions than consumers → consumers get multiple partitions.

14) Backpressure

   When a consumer cannot process messages as fast as they arrive,
   control/slow down consumption or processing to prevent overload.

15) Offset

   A position in a partition that identifies a record.

   Consumer uses offsets to track its progress.

   Committing an offset means:
   "I have successfully processed records up to this position."


16) Replication
   Each partition can have multiple replicas across brokers.
   One replica = Leader, others = Followers.

17) ISR (In-Sync Replicas)
   Replicas that are sufficiently caught up with the partition leader.
   Used with acks=all and min.insync.replicas.

18) Leader Election
   If a partition leader fails, Kafka elects another eligible replica
   as the new leader.

19) Partition Ordering
   Kafka guarantees message ordering only within a partition,
   NOT across the entire topic.

20) Partitioning
   Producer decides which partition receives a record.
   → Explicit partition → use that partition.
   → Key provided → partitioner determines partition.
   → No key → partitioner distributes records.

21) Consumer Polling
   Consumer calls poll() to fetch records from Kafka.
   max.poll.records controls records returned per poll.

22) Consumer Offset Commit
   Auto commit → Kafka periodically commits offsets.
   Manual commit → application controls when offsets are committed.
   commitSync() → waits for commit result.
   commitAsync() → asynchronous commit.

23) Retention
   Kafka does NOT delete messages just because a consumer has read them.
   Messages are deleted based on retention policies
   (time/size).

24) Consumer Lag
   Difference between the latest available offset and the
   consumer's committed/processed offset.
   
   High lag → consumer is falling behind.

25) Replication Factor
   Number of replicas maintained for each partition.

   RF = 3
   → 1 Leader + 2 Followers

26) min.insync.replicas
   Minimum number of ISR replicas required for a successful
   write when acks=all.

27) Consumer Scaling
   Partitions determine maximum parallelism for traditional
   consumer groups.

   3 partitions + 5 consumers
   → only 3 consumers can actively consume.

28) Dead Letter Topic (DLT)
   Messages that repeatedly fail processing can be sent to a
   separate topic for later investigation/reprocessing.

29) Retry
   Failed messages can be retried immediately or sent to
   retry topics with delayed processing.

30) Kafka Transactions
   Used to atomically produce records and commit consumed offsets.
   Important for Kafka-to-Kafka exactly-once processing.

31) Schema / Serialization
   Producer serializes objects → bytes.
   Consumer deserializes bytes → objects.

   Common:
   JSON, Avro, Protobuf.

32) Consumer Lag Monitoring
   Monitor:
   → Current/latest offset
   → Consumer committed offset
   → Lag
