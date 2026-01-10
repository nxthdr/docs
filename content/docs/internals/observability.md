---
title: Observability
weight: 5
prev: /docs/internals/probing/
next: /docs/reference/
---

The nxthdr platform implements a comprehensive observability strategy to monitor infrastructure health, service performance, and system availability. The stack combines metrics, logs, and external monitoring to provide complete visibility.

## Architecture Overview

The observability infrastructure consists of multiple components working together:

```
Services → Prometheus (metrics) → Grafana (visualization)
        → Loki (logs)          ↗
        → Alloy (collection)  ↗

External: Upptime (HTTP) + UptimeRobot (ICMP)
```

## Components

### Prometheus

[Prometheus](https://prometheus.io/) is the core metrics collection and storage system. It scrapes metrics from all services and stores them as time-series data.

**Configuration**:
- Scrape interval: 30 seconds
- Retention: 15 days
- Storage: Local filesystem
- Authentication: Basic auth for remote write and queries

**Scrape Targets**:
- **Infrastructure services**: Proxy (Caddy), ClickHouse, Redpanda, PostgreSQL
- **Data collectors**: Risotto (BMP), Pesto (sFlow), Saimiris (probing)
- **Observability stack**: Prometheus itself, Loki, Grafana, Alertmanager
- **System metrics**: Node Exporter, cAdvisor
- **Application services**: PeerLab Gateway, Saimiris Gateway, nxthdr.dev

**Metrics Exposed**:
All services expose Prometheus metrics on `/metrics` endpoints:
- Request rates and latencies
- Error rates and types
- Resource utilization (CPU, memory, disk)
- Queue depths and processing rates
- Custom business metrics

### Loki

[Loki](https://grafana.com/oss/loki/) aggregates logs from all services and makes them queryable alongside metrics in Grafana.

**Architecture**:
- Single-instance deployment on core server
- Filesystem storage for chunks and indexes
- TSDB schema for efficient querying
- 7-day retention with automatic compaction

**Log Sources**:
- **Container logs**: Docker container stdout/stderr via Alloy
- **System logs**: Syslog messages via Alloy (TCP port 601, UDP port 514)
- **Application logs**: Structured logging from services

**Features**:
- Label-based indexing (no full-text indexing)
- LogQL query language
- Integration with Alertmanager for log-based alerts
- Correlation with metrics in Grafana

### Grafana

[Grafana](https://grafana.com/) provides visualization and dashboards for metrics and logs.

**Data Sources**:
- **Prometheus**: Default data source for metrics
- **Loki**: Log aggregation and search
- **ClickHouse**: Direct queries for BGP, flows, and probing data

**Dashboards**:
Pre-configured dashboards for:
- Infrastructure overview (system resources, network)
- Service-specific metrics (Risotto, Pesto, Saimiris)
- ClickHouse performance and query statistics
- Redpanda broker and topic metrics
- Container and host metrics

**Access**:
- Public URL: `https://grafana.nxthdr.dev`
- Authentication: Auth0 integration
- Role-based access control

### Alloy

[Grafana Alloy](https://grafana.com/docs/alloy/) is the telemetry collector that gathers logs and metrics from distributed servers and forwards them to the central observability stack.

**Deployment**:
Alloy runs on all servers (core, IXP, probing) as Docker containers.

**Log Collection**:
- **Container logs**: Reads Docker container logs from `/var/lib/docker/containers`
- **Syslog**: Listens on TCP/UDP for RFC5424 syslog messages
- **Processing**: Parses JSON logs, extracts labels, timestamps

**Metrics Collection**:
- **Node Exporter**: System metrics (CPU, memory, disk, network)
- **cAdvisor**: Container metrics (resource usage per container)
- **Service metrics**: Application-specific metrics from local services
- **Remote write**: Forwards all metrics to central Prometheus

**Configuration**:
- Pipeline-based configuration with stages
- Label relabeling for consistent naming
- Instance identification for multi-server deployments

### Node Exporter

[Node Exporter](https://github.com/prometheus/node_exporter) exposes hardware and OS metrics for Unix systems.

**Metrics Provided**:
- CPU usage and load averages
- Memory and swap utilization
- Disk I/O and space usage
- Network interface statistics
- Filesystem metrics
- System uptime

**Deployment**:
Runs as a Docker container on all servers, scraped by local Alloy instance.

### Alertmanager

[Alertmanager](https://prometheus.io/docs/alerting/latest/alertmanager/) handles alerts from Prometheus and routes them to notification channels.

**Configuration**:
- Grouping: Alerts grouped by name and job
- Deduplication: Prevents duplicate notifications
- Routing: All alerts sent to Discord webhook
- Repeat interval: 3 hours for ongoing alerts

**Alert Rules**:
Prometheus evaluates alert rules defined in `alerts.yml`:
- **Service availability**: Detects when core services go down
- **Resource exhaustion**: High CPU, memory, or disk usage
- **Data pipeline health**: Kafka lag, ClickHouse errors
- **Network issues**: BGP session failures, high packet loss

**Notification Channels**:
- Discord webhook for real-time alerts
- Future: Email, PagerDuty, Slack integration

### Upptime

[Upptime](https://upptime.js.org/) provides HTTP endpoint monitoring with a public status page.

**Monitoring**:
- HTTP/HTTPS endpoint availability checks
- Response time measurements
- SSL certificate expiration tracking
- Historical uptime statistics

**Status Page**:
- Public dashboard showing service status
- Incident history and response times
- Automated via GitHub Actions
- Static site hosted on GitHub Pages

**Endpoints Monitored**:
- nxthdr.dev (main website)
- Grafana, Prometheus, Loki (observability stack)
- API endpoints (PeerLab Gateway, Saimiris Gateway)
- Public services (Geofeed, CHProxy)

### UptimeRobot

[UptimeRobot](https://uptimerobot.com/) provides ICMP ping monitoring for network-level availability.

**Monitoring**:
- ICMP ping checks to core infrastructure
- Network reachability validation
- Complementary to HTTP monitoring
- Independent external monitoring

**Targets**:
- Core server IPv6 address
- IXP server addresses
- Probing server addresses
- Critical network endpoints

**Alerts**:
- Email notifications for downtime
- SMS alerts for critical failures
- Integration with status page

## Observability Patterns

### Metrics Collection

**Push vs Pull**:
- **Pull model**: Prometheus scrapes metrics from services (preferred)
- **Push model**: Alloy remote write for distributed servers

**Metric Types**:
- **Counters**: Total requests, errors, bytes processed
- **Gauges**: Current values (queue depth, active connections)
- **Histograms**: Request latencies, response sizes
- **Summaries**: Quantiles for performance analysis

### Log Aggregation

**Structured Logging**:
Services use structured logging with consistent fields:
- Timestamp, level, message
- Service name, instance, job
- Request ID for tracing
- Custom context fields

**Label Strategy**:
Loki uses labels for indexing (not full-text):
- `job`: Service type (risotto, pesto, saimiris)
- `instance`: Server hostname
- `container_name`: Docker container
- `level`: Log level (info, warn, error)

### Alerting Strategy

**Alert Severity Levels**:
- **Critical**: Service down, data loss risk
- **Warning**: Degraded performance, approaching limits
- **Info**: Notable events, configuration changes

**Alert Grouping**:
Related alerts grouped to reduce notification noise:
- By service (all Redpanda alerts together)
- By instance (all alerts from one server)
- Time-based grouping (5-minute window)

## Monitoring Coverage

### Infrastructure Layer

**System Metrics**:
- CPU, memory, disk, network utilization
- Process counts and system load
- Filesystem space and inode usage
- Network interface errors and drops

**Container Metrics**:
- Per-container resource usage
- Container restart counts
- Image pull statistics
- Docker daemon health

### Application Layer

**Service Health**:
- HTTP endpoint availability (Upptime)
- Service process uptime
- Health check endpoints
- Dependency availability

**Performance Metrics**:
- Request rates and latencies
- Error rates by type
- Queue depths and processing times
- Database query performance

### Data Pipeline Layer

**Kafka/Redpanda**:
- Topic lag and throughput
- Producer and consumer rates
- Broker health and replication
- Partition distribution

**ClickHouse**:
- Query execution times
- Insert rates and batch sizes
- Table sizes and compression ratios
- Replication lag (if applicable)

**Collectors**:
- Messages received and processed
- Parse errors and invalid data
- Buffer utilization
- Backpressure indicators

## Access and Usage

**Grafana Access**:
- URL: `https://grafana.nxthdr.dev`
- Authentication via Auth0
- Pre-built dashboards for all services
- Custom dashboard creation supported

**Prometheus Access**:
- URL: `https://prometheus.nxthdr.dev`
- Basic authentication required
- PromQL query interface
- API for programmatic access

**Loki Access**:
- URL: `https://loki.nxthdr.dev`
- Basic authentication required
- LogQL query interface
- Integrated in Grafana for log exploration

**Status Pages**:
- Upptime: Public HTTP monitoring dashboard
- UptimeRobot: ICMP monitoring status

## Retention and Storage

**Prometheus**:
- Retention: 15 days
- Storage: Local filesystem on core server
- Compaction: Automatic block compaction
- Backup: Not currently implemented

**Loki**:
- Retention: 7 days
- Storage: Filesystem chunks and indexes
- Compaction: Automatic with 2-hour delay
- Cleanup: TTL-based deletion

**Grafana**:
- Dashboard persistence: PostgreSQL
- User sessions: In-memory
- Snapshots: Stored indefinitely
