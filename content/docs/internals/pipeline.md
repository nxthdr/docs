---
title: Data Pipelines
weight: 2
prev: /docs/internals/infrastructure/
next: /docs/internals/storage
---

The nxthdr platform processes network telemetry data through two main pipelines: BGP routing updates and network flow data. Both pipelines follow a similar architecture pattern, collecting data from network devices, streaming it through Kafka, and storing it in ClickHouse for analysis.

## BGP Pipeline

The BGP pipeline collects routing updates from BIRD routers using the BGP Monitoring Protocol (BMP) and makes them available for analysis.

### Architecture

```
BIRD Routers → Risotto → Redpanda (Kafka) → ClickHouse
```

### Components

**Risotto Collector**

[Risotto](https://github.com/nxthdr/risotto) is a high-performance BMP collector written in Rust that processes BMP messages from routers and publishes BGP updates to Kafka. It listens on port 4000 for incoming BMP connections and exposes Prometheus metrics on port 8080.

Key features:
- **State management**: Maintains a representation of connected routers, BGP peers, and announced prefixes
- **Deduplication**: Filters duplicate announcements from BMP session resets
- **Synthetic withdraws**: Generates missing withdraw messages when BGP sessions go down
- **Stateless mode**: Can be configured to stream updates without state management

**Data Curation**

Risotto addresses two common challenges with BMP data:

1. **Duplicate announcements** occur when BMP sessions reset and routers resend all active prefixes. Risotto checks each update against its state and filters duplicates.

2. **Missing withdraws** happen when BGP sessions terminate and routers fail to send withdraw messages, or when the collector experiences downtime. Risotto generates synthetic withdraws using stored prefix state for downed peers.

When Risotto restarts, it infers missing withdraws from the initial Peer Up sequence, ensuring database consistency. Some duplicate announcements may be replayed after restart, but the database state remains accurate.

**Serialization Format**

BGP updates are serialized using [Cap'n Proto](https://capnproto.org/) before being sent to Kafka. The schema includes:

- Metadata: timestamps, router address/port, peer information
- Prefix information: address, length, announcement/withdrawal flag
- BGP attributes: origin, AS path, next hop, communities, MED, local preference, etc.
- Flags: post-policy, adj-rib-out, synthetic withdraw indicators

**Kafka Topic**

Updates are published to the `risotto-updates` topic in Redpanda (Kafka-compatible). This decouples collection from storage and enables multiple consumers to process the same data stream.

**ClickHouse Storage**

The pipeline uses three ClickHouse tables in the `bmp` database:

1. `from_kafka`: Kafka engine table that consumes Cap'n Proto messages
2. `updates`: MergeTree table storing processed updates with 7-day TTL
3. `from_kafka_mv`: Materialized view transforming data from Kafka to storage format

The storage schema converts field names to snake_case, adds insertion timestamps, and partitions data by date for efficient querying and automatic expiration.

## Flows Pipeline

The flows pipeline collects network flow data using sFlow v5 and stores sampled traffic information for analysis.

### Architecture

```
Network Devices → Pesto → Redpanda (Kafka) → ClickHouse
```

### Components

**Pesto Collector**

[Pesto](https://github.com/nxthdr/pesto) is a high-performance sFlow v5 collector written in Rust that receives sFlow datagrams via UDP, parses them, and forwards them to Kafka. By default, it listens on port 6343 for sFlow datagrams.

Key features:
- **UDP receiver**: Handles high-volume sFlow datagram streams
- **Flow-only processing**: Processes flow samples only (counter samples are filtered)
- **IPv6 support**: All flows are normalized to IPv6 (IPv4 addresses are mapped to IPv6)
- **Cap'n Proto serialization**: Efficient binary format for Kafka messages

**Flow Sample Processing**

Pesto processes sFlow flow samples and extracts:

- Datagram metadata: agent address/port, sequence numbers, uptime
- Sample metadata: sampling rate, sample pool, drops, interfaces
- Flow data: source/destination IPs and ports, protocol, packet length, TCP flags, ToS

Counter samples are serialized internally but filtered at the producer level before sending to Kafka. This design follows the approach used by goflow2 and accommodates ClickHouse's limitation with Cap'n Proto union types.

**Serialization Format**

Flow records use a flat Cap'n Proto schema compatible with ClickHouse. All fields are at the root level with no nested structures or unions. IPv4 addresses are converted to IPv4-mapped IPv6 addresses for consistent storage.

**Kafka Topic**

Flow records are published to the `pesto-sflow` topic in Redpanda, enabling downstream consumers to process flow data independently.

**ClickHouse Storage**

The pipeline uses three ClickHouse tables in the `flows` database:

1. `from_kafka`: Kafka engine table consuming Cap'n Proto messages
2. `records`: MergeTree table storing flow records with 7-day TTL
3. `from_kafka_mv`: Materialized view transforming and aggregating data

The materialized view calculates estimated bytes and packets based on sampling rates, converts timestamps, and filters out invalid samples (sampling rate = 0). Data is partitioned by date and ordered by timestamp and IP addresses for efficient querying.

## Common Patterns

Both pipelines share several design patterns:

**Streaming Architecture**: Data flows through Kafka/Redpanda, decoupling collection from storage and enabling multiple consumers.

**Cap'n Proto Serialization**: Binary format provides efficient serialization with schema evolution support.

**ClickHouse Integration**: Kafka engine tables consume messages directly, materialized views transform data, and MergeTree tables provide efficient storage with automatic TTL-based expiration.

**High Performance**: Rust implementations provide low latency and high throughput for processing network telemetry at scale.

**Observability**: All collectors expose Prometheus metrics for monitoring performance and statistics.
