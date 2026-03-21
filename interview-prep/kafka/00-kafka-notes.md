---

# 🚀 Apache Kafka — Complete Fundamentals Notes

---

# 🌊 1. What is Kafka? (Core Idea)

## 🔹 Event Streaming

Event streaming means:

* Capturing data in real-time from different sources
* Storing it durably for later use
* Processing and reacting to it in real-time or later

👉 Example:

* Payment happens → event generated → multiple systems react (fraud, analytics, notifications)

---

## 🔹 Definition

👉 **Apache Kafka is a distributed event streaming platform**

---

## 🔹 Kafka Capabilities (Very Important)

1. Publish (write) and subscribe (read) event streams
2. Store events durably and reliably
3. Process events in real-time or retrospectively

---

## 🔹 Key Characteristics

* Distributed
* Fault-tolerant
* Scalable (elastic)
* High-throughput
* Secure

---

## 🎯 Interview Answer

Kafka is a distributed event streaming platform that allows applications to publish, store, and process streams of events in real time. It is designed for high throughput, fault tolerance, and scalability, and is widely used in event-driven architectures.

---

# 🏗️ 2. Kafka Architecture (Big Picture)

```text
Producer → Kafka Cluster → Consumer
```

Inside Kafka:

```text
Cluster → Brokers → Topics → Partitions → Events
```

---

# 🖥️ 3. Server Side (Cluster, Broker, Connect)

## 🔹 Kafka Cluster

* Kafka runs as a **cluster of servers**
* Can run on:

  * Bare metal
  * VMs
  * On-prem
  * Cloud

---

## 🔹 Broker

👉 A **single Kafka server**

* Stores data
* Handles read/write requests

---

## 🔹 Cluster

👉 Group of brokers working together

* Provides scalability + fault tolerance
* One broker acts as **controller (in KRaft mode)**

---

## 🔹 Kafka Connect

👉 Tool to integrate Kafka with external systems

### Why it exists

* Avoid tight coupling
* Avoid custom integration code
* Avoid DB overload

---

### 🔹 Data Flow

**Import into Kafka**

* Databases (MySQL, Postgres)
* File systems (logs, CSV)

**Export from Kafka**

* Elasticsearch (search)
* S3 (storage)
* Data warehouse (analytics)

---

### 🔹 Why Kafka Connect is important

* Decouples systems
* Enables real-time streaming
* Supports replay
* Eliminates bottlenecks

---

## 🎯 Interview Answer

Kafka Connect is a framework that allows Kafka to integrate with external systems like databases and data warehouses. It enables continuous data streaming without custom code and helps decouple systems.

---

# 💻 4. Client Side (Producers & Consumers)

## 🔹 Producers

* Write events to Kafka

## 🔹 Consumers

* Read and process events

---

## 🔹 Key Property

👉 **Fully decoupled**

* Producer doesn’t care who consumes
* Consumer doesn’t care who produces

---

## 🎯 Interview Answer

Producers send data to Kafka topics, while consumers read and process that data. Both are decoupled, enabling scalable and independent microservices.

---

# 📩 5. Events (Message)

👉 Smallest unit in Kafka

---

## 🔹 Structure

* Key
* Value
* Timestamp
* Headers (optional)

---

## 🔹 Example

```json
{
  "key": "orderId-123",
  "value": "order placed",
  "timestamp": "2026-03-22"
}
```

---

## 🎯 Interview Answer

An event in Kafka is a record representing something that happened, consisting of a key, value, timestamp, and optional metadata.

---

# 📂 6. Topic

👉 Logical grouping of events

---

## 🔹 Example

* `order-events`
* `payment-events`

---

## 🔹 Key Points

* Multi-producer & multi-consumer
* Like a folder
* Events are NOT deleted after consumption

---

## 🎯 Interview Answer

A topic is a logical stream of events where producers publish data and consumers subscribe to read it.

---

# 📊 7. Partition (🔥 VERY IMPORTANT)

👉 Topics are split into partitions

---

## 🔹 Why partitions?

* Parallel processing
* Scalability
* High throughput

---

## 🔹 Internal Working

* Each partition = **append-only log**
* Messages are immutable
* Stored sequentially

---

## 🔹 Ordering Rule

👉 Ordering is guaranteed **only within a partition**

---

## 🎯 Interview Answer

Partitions are the units of parallelism in Kafka, where each partition is an ordered, immutable sequence of records enabling scalability.

---

# 🔑 8. Partition Key & Partitioner

## 🔹 Partition Key

* Determines partition placement

---

## 🔹 Rule

* Same key → same partition
* Ensures ordering

---

## 🔹 Partitioner Logic

```text
if key present:
    hash(key) % partitions
else:
    round-robin
```

---

## 🎯 Interview Answer

The partition key ensures that related messages go to the same partition, maintaining order, while the partitioner decides how messages are distributed across partitions.

---

# 🔢 9. Offset

👉 Unique ID of message inside a partition

---

## 🔹 Key Points

* Per partition
* Sequential
* Assigned automatically

---

## 🎯 Interview Answer

Offset is a unique identifier for each message within a partition and helps consumers track their position.

---

# 👥 10. Consumer Group (🔥 MOST ASKED)

👉 Group of consumers reading a topic

---

## 🔹 Rule

👉 One partition → one consumer (per group)

---

## 🔹 Why?

* No duplicate processing
* Load balancing

---

## 🎯 Interview Answer

A consumer group allows multiple consumers to share the processing of a topic, where each partition is assigned to only one consumer.

---

# 📍 11. Consumer Offset

👉 Tracks last consumed message

---

## 🔹 Stored in

```text
__consumer_offsets topic
```

---

## 🔹 Types

* Auto commit
* Manual commit (recommended)

---

## 🎯 Interview Answer

Consumer offset tracks the last processed message, allowing recovery and reprocessing in case of failures.

---

# 💾 12. Segments (Internal Storage)

👉 Partition is divided into segments (files)

---

## 🔹 Example

```text
partition-0/
   segment1.log
   segment2.log
```

---

## 🔹 Why?

* Efficient disk usage
* Easier cleanup

---

## 🎯 Interview Answer

Kafka stores partition data in smaller segment files, enabling efficient storage management and faster cleanup.

---

# 🧹 13. Retention Policy

👉 Controls how long data is stored

---

## 🔹 Types

* Time-based (e.g., 7 days)
* Size-based (e.g., 1GB)

---

## 🔹 Important

👉 Kafka DOES NOT delete after consumption

---

## 🎯 Interview Answer

Kafka retains data based on time or size policies, independent of consumption, allowing replay of events.

---

# 🔁 14. Replication Factor (🔥 CRITICAL)

👉 Number of copies of data

---

## 🔹 Example

```text
RF = 3 → 1 leader + 2 followers
```

---

## 🔹 Internal Working

* Leader handles read/write
* Followers replicate
* ISR (in-sync replicas)

---

## 🔹 Failure Handling

* Leader dies → follower becomes leader

---

## 🎯 Interview Answer

Replication factor ensures fault tolerance by maintaining multiple copies of data across brokers, with one leader and multiple followers.

---

# ⚙️ 15. Kafka Storage (KRaft Mode)

## 🔹 Cluster ID

```bash
kafka-storage.sh random-uuid
```

👉 Generates unique cluster ID

---

## 🔹 Format Storage

```bash
kafka-storage.sh format --standalone
```

👉 Initializes metadata and storage

---

## 🔹 Start Kafka

```bash
kafka-server-start.sh server.properties
```

---

## 🎯 Explanation

These steps generate a cluster ID, initialize storage with metadata, and start the Kafka broker in KRaft mode.

---

# 📁 16. Log Directories

## 🔹 Important Config

```properties
log.dirs=... (broker data)
metadata.log.dir=... (controller metadata)
```

---

## 🔹 Combined Mode

```properties
process.roles=broker,controller
```

---

## 🔹 Why Separate?

* Better performance
* Reliability
* Production best practice

---

# 🧪 17. Basic Commands

## 🔹 Create Topic

```bash
kafka-topics.sh --create --topic test-topic --bootstrap-server localhost:9092
```

## 🔹 Describe Topic

```bash
kafka-topics.sh --describe --topic test-topic --bootstrap-server localhost:9092
```

## 🔹 Producer

```bash
kafka-console-producer.sh --topic test-topic --bootstrap-server localhost:9092
```

## 🔹 Consumer

```bash
kafka-console-consumer.sh --topic test-topic --from-beginning --bootstrap-server localhost:9092
```

---

# 🔄 18. End-to-End Flow (VERY IMPORTANT)

1. Producer sends event
2. Partitioner selects partition
3. Event appended to partition
4. Offset assigned
5. Replicated to followers
6. Consumer group reads
7. Offset committed

---

# 🧠 19. Whiteboard Summary (REVISION)

```text
Kafka = Distributed Append-only Log

Broker → Server
Cluster → Group of brokers
Topic → Logical stream
Partition → Parallel ordered log
Offset → Message ID
Consumer Group → Load balancing
Replication → Fault tolerance
Retention → Data lifecycle
```

---

# 🔥 20. Real-World Use Cases

* Payment processing
* Fraud detection
* Log aggregation
* Microservices communication
* Real-time analytics

---

# 🎯 Final Interview Master Answer

Kafka is a distributed event streaming platform that works as an append-only log system. Data is organized into topics and partitions for scalability and ordering. Producers publish events, and consumers read them using offsets. Kafka ensures fault tolerance through replication and enables parallel processing using consumer groups, making it ideal for real-time, event-driven architectures.

---

```
Kafka
Kafka is a distributed event streaming platform that allows applications to publish, store, and process streams of events in real time. It is designed for high throughput, fault tolerance, and scalability, and is widely used in event-driven architectures.

Event
An event is a record consisting of a key, value, timestamp, and optional headers representing a state change or action.

Producer
A producer is a client that publishes events to Kafka topics.

Consumer
A consumer is a client that subscribes to topics and processes events from Kafka.

Topic
A topic is a logical category or stream where events are stored and accessed by producers and consumers.

Partition
A partition is an ordered, immutable, append-only log that enables parallelism and scalability within a topic.

Partition Key
The partition key determines the target partition for a message, ensuring that messages with the same key go to the same partition.

Partitioner
The partitioner is the component that assigns messages to partitions using key-based hashing or round-robin logic.

Offset
An offset is a unique sequential identifier assigned to each message within a partition.

Consumer Group
A consumer group is a set of consumers that share the workload of consuming a topic, with each partition assigned to one consumer.

Consumer Offset
Consumer offset is the position of the last consumed message tracked per partition for each consumer group.

Broker
A broker is a Kafka server responsible for storing data and serving client requests.

Cluster
A cluster is a group of Kafka brokers working together to provide scalability and fault tolerance.

Replication Factor
Replication factor defines the number of copies of each partition distributed across brokers for fault tolerance.

Leader & Follower
The leader handles all read/write operations for a partition, while followers replicate data and take over on failure.

ISR (In-Sync Replicas)
ISR is the set of replicas that are fully synchronized with the leader and eligible for leader election.

Segment
A segment is a smaller log file that makes up a partition and enables efficient storage and cleanup.

Retention Policy
Retention policy defines how long Kafka retains messages based on time or size, independent of consumption.

Kafka Connect
Kafka Connect is a framework for streaming data between Kafka and external systems using configurable connectors.

KRaft
KRaft is Kafka’s internal consensus mechanism that replaces Zookeeper for managing cluster metadata.

log.dirs
log.dirs specifies the directory where Kafka stores partition data logs.

metadata.log.dir
metadata.log.dir specifies the directory where Kafka stores cluster metadata in KRaft mode.

End-to-End Flow
Kafka processes data by producing events to partitions, assigning offsets, replicating data, and allowing consumers to read and commit offsets.
```