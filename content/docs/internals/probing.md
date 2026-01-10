---
title: Probing Platform
weight: 3
prev: /docs/internals/peering/
next: /docs/internals/pipeline
---

Saimiris is an internet-scale active measurements pipeline that enables distributed network probing from multiple vantage points. The platform performs traceroute and ping measurements to analyze Internet paths, latency, and topology.

## Architecture Overview

The Saimiris platform consists of three main components working together to orchestrate and execute measurements:

```
Client → Gateway (API) → Kafka → Agents → Kafka → ClickHouse
                ↓
           PostgreSQL
```

## Components

### Saimiris Agent

The [Saimiris](https://github.com/nxthdr/saimiris) agent performs the actual measurements. It runs on distributed servers (probing vantage points) and continuously listens for probe requests.

**Core Functionality**:
- Consumes probe specifications from Kafka topic (`saimiris-probes`)
- Executes measurements using the caracat library
- Produces measurement results to Kafka topic (`saimiris-replies`)
- Supports multiple probing instances per agent (anycast and unicast)

**Probing Capabilities**:
- **Protocols**: UDP, ICMP, ICMPv6
- **Source addressing**: Configurable IPv4/IPv6 prefixes per instance
- **Integrity checking**: Optional probe validation
- **Rate limiting**: Configurable probing rate per instance

**Architecture**:
The agent uses a multi-threaded architecture:
- **Consumer thread**: Reads probe specifications from Kafka
- **Sender threads**: One per caracat instance, sends probes using raw sockets
- **Receiver thread**: Captures ICMP replies using pcap
- **Producer thread**: Serializes and sends results to Kafka
- **Gateway client**: Periodic health checks and configuration updates

**Configuration Example**:
```yaml
agent:
  id: "vltcdg01"
  metrics_address: "0.0.0.0:8080"

caracat:
  - name: anycast
    instance_id: 0
    src_ipv6_prefix: "2a0e:97c0:8a0::/48"
    probing_rate: 10000
  - name: unicast
    instance_id: 1
    src_ipv6_prefix: "2001:db8::/48"
    probing_rate: 10000

kafka:
  brokers: "redpanda.nxthdr.dev:9092"
  in_topics: saimiris-probes
  out_topic: saimiris-replies
```

### Saimiris Gateway

The [Saimiris Gateway](https://github.com/nxthdr/saimiris-gateway) is a REST API service that manages the measurement pipeline. It provides authentication, quota management, and agent coordination.

**API Surfaces**:

**Client API** (JWT authenticated):
- `GET /api/user/me`: Retrieve user probe daily usage statistics
- `GET /api/user/prefixes`: List user prefixes per agent
- `POST /api/probes`: Submit probes for measurement
- `GET /api/measurements/{id}/status`: Get measurement status

**Agent API** (agent key authenticated):
- `POST /agent-api/agent/register`: Register a new agent
- `POST /agent-api/agent/{id}/config`: Update agent configuration
- `POST /agent-api/agent/{id}/health`: Update agent health status
- `POST /agent-api/agent/{id}/measurement/{id}/status`: Update measurement status

**Public API**:
- `GET /api/agents`: List all agents
- `GET /api/agent/{id}`: Get agent details
- `GET /api/agent/{id}/config`: Get agent configuration
- `GET /api/agent/{id}/health`: Get agent health status

**Key Features**:
- **Quota management**: Daily probe limits per user (SHA256 hashed identifiers)
- **Agent management**: Registration, configuration, and health monitoring
- **Measurement tracking**: Status updates and completion tracking
- **Prefix validation**: Ensures probes use authorized source prefixes
- **Auto-migration**: Database schema automatically created and updated

**Data Storage**:
PostgreSQL database stores:
- User probe quotas and daily usage
- Agent registrations and configurations
- Measurement metadata and status
- Source prefix allocations per agent

### Saimiris Client

The client component submits measurement campaigns. It reads probe specifications and distributes them to agents via Kafka.

The Saimiris client is designed to operate independently from the Gateway. While the Gateway provides orchestration, authentication, and quota management for the nxthdr platform, the core Saimiris client and agent can be deployed standalone with just Kafka for custom measurement pipelines.

**Usage**:
```bash
cat probes.txt | saimiris client --config=saimiris.yml agent1:ip1,agent2:ip2
```

**Probe Format**:
Probes use the [caracal](https://dioptra-io.github.io/caracal/usage/) format:
```
2001:db8::1,24000,33434,24,UDP
2001:db8::2,24000,33435,24,UDP
```

Fields: destination IP, source port, destination port, TTL, protocol

**Agent Targeting**:
The client supports targeting specific agents with optional source IP specification:
- `agent1:192.0.2.1,agent2:[2001:db8::1]`: Probes sent from specified IPs
- `agent1,agent2`: Agent selects source IP based on prefix configuration

## Data Pipeline

### Serialization Format

Measurements use Cap'n Proto for efficient binary serialization.

**Probe Schema**:
```capnp
struct Probe {
    dstAddr      :Data;
    srcPort      :UInt16;
    dstPort      :UInt16;
    ttl          :UInt8;
    protocol     :Protocol;
}
```

**Reply Schema**:
```capnp
struct Reply {
    timeReceivedNs      :UInt64;
    agentId             :Text;
    replySrcAddr        :Data;
    replyDstAddr        :Data;
    replyId             :UInt16;
    replySize           :UInt16;
    replyTtl            :UInt8;
    replyQuotedTtl      :UInt8;
    replyProtocol       :UInt8;
    replyIcmpType       :UInt8;
    replyIcmpCode       :UInt8;
    replyMplsLabel      :List(Mpls);
    probeSrcAddr        :Data;
    probeDstAddr        :Data;
    probeId             :UInt16;
    probeSize           :UInt16;
    probeTtl            :UInt8;
    probeProtocol       :UInt8;
    probeSrcPort        :UInt16;
    probeDstPort        :UInt16;
    rtt                 :UInt16;
}
```

### Kafka Topics

**`saimiris-probes`**: Gateway publishes probe specifications consumed by agents. Each message contains a batch of probes targeting specific agents.

**`saimiris-replies`**: Agents publish measurement results consumed by ClickHouse. Each message contains a single probe reply with timing and path information.

### ClickHouse Storage

The pipeline uses three ClickHouse tables in the `saimiris` database:

1. `from_kafka`: Kafka engine table consuming Cap'n Proto messages
2. `replies`: MergeTree table storing measurement results with 7-day TTL
3. `from_kafka_mv`: Materialized view transforming data from Kafka to storage format

**Storage Schema**:
Results are partitioned by date and ordered by timestamp, agent, protocol, and flow tuple (source/destination IPs and ports, TTL) for efficient querying of measurement campaigns.

**Key Fields**:
- Timing: receive timestamp, RTT
- Reply information: source/destination, ICMP type/code, TTL, MPLS labels
- Probe information: source/destination, ports, protocol, TTL
- Agent identifier for vantage point tracking

## Deployment

### Probing Servers

Probing agents run on Vultr instances distributed across multiple geographic locations. Each server:
- Runs the Saimiris agent as a Docker container
- Has dedicated IPv6 prefixes for measurements
- Supports both anycast and unicast source addressing
- Connects to core infrastructure via WireGuard tunnels

**Prefix Allocation**:
- Anycast prefix: `2a0e:97c0:8a0::/48` (shared across all agents)
- Unicast prefixes: Per-agent /48 subnets for unique source addressing

### Gateway Service

The gateway runs on the core server alongside other nxthdr services:
- PostgreSQL database for state management
- Redpanda (Kafka) for message streaming
- Auth0 integration for user authentication
- Prometheus metrics for monitoring

## Use Cases

**Path Discovery**: Traceroute measurements reveal Internet routing paths, identifying transit networks and peering relationships.

**Latency Analysis**: RTT measurements quantify network performance from different vantage points.

**Topology Mapping**: Systematic probing discovers router-level Internet topology and MPLS tunnels.

**Anycast Analysis**: Measurements from distributed vantage points reveal anycast catchment areas and routing behavior.

**Load Balancing Detection**: Per-flow probing identifies load balancers and ECMP routing.

## Security and Privacy

**Authentication**: Multi-layer authentication protects the measurement infrastructure:
- Gateway API uses Auth0 JWT tokens for users
- Agent API uses shared secret keys for agent authentication
- User identifiers are SHA256 hashed for privacy

**Quota Management**: Daily probe limits prevent abuse and ensure fair resource allocation across users.

**Source Validation**: Gateway validates that requested source IPs match agent prefix allocations.

**Network Isolation**: Agents connect to core infrastructure via encrypted WireGuard tunnels.

## Observability

**Agent Metrics**: Prometheus metrics exposed on port 8080:
- Probes sent/received counters
- Kafka message production rates
- Integrity check failures
- Sender/receiver thread statistics

**Gateway Metrics**: API request rates, authentication failures, quota violations, and database query performance.

**Health Monitoring**: Agents periodically report health status to gateway, enabling detection of offline or degraded vantage points.
