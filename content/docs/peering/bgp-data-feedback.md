---
title: BGP Data Feedback
weight: 3
prev: /docs/peering/routing-configurations/
next: /docs/peering/infrastructure/
---

One of PeerLab's unique features is that all routing data from AS215011 is collected and made publicly available. This allows you to observe how your BGP announcements propagate, analyze routing dynamics, and use these insights to refine your experiments.

All data is accessible via ClickHouse SQL queries. See the [datasets](/docs/datasets/) section for access credentials and general documentation.

## Verify Your Announcements

After starting your PeerLab environment and establishing BGP sessions, confirm that your prefix announcements are being received by the IXP servers.

### Check Announcement Status

```sh
curl -X POST "https://nxthdr.dev/api/query/" \
  -u "read:read" \
  -H "Content-Type: text/plain" \
  -d "SELECT
    time_received_ns,
    peer_addr,
    prefix_addr,
    prefix_len,
    announced,
    as_path,
    next_hop
FROM bmp.updates
WHERE peer_asn = YOUR_ASN
ORDER BY time_received_ns DESC
LIMIT 20 FORMAT CSVWithNames"
```

The `announced` field indicates whether the update is an announcement (`true`) or a withdrawal (`false`).

### Track Announcement and Withdrawal History

Monitor the lifecycle of a specific prefix to see when it was announced and withdrawn:

```sh
curl -X POST "https://nxthdr.dev/api/query/" \
  -u "read:read" \
  -H "Content-Type: text/plain" \
  -d "SELECT
    toDateTime(time_received_ns / 1000000000) AS time,
    peer_addr,
    announced,
    as_path
FROM bmp.updates
WHERE peer_asn = YOUR_ASN
  AND prefix_addr = toIPv6('YOUR_PREFIX')
  AND prefix_len = 48
ORDER BY time_received_ns DESC
LIMIT 50 FORMAT CSVWithNames"
```

This is useful for verifying that lease expiration correctly triggers withdrawals.

## Analyze AS Path Propagation

### Observe AS Path Diversity

See how your routes are learned by different peers with different AS paths:

```sh
curl -X POST "https://nxthdr.dev/api/query/" \
  -u "read:read" \
  -H "Content-Type: text/plain" \
  -d "SELECT
    peer_addr,
    peer_asn,
    as_path,
    count(*) AS updates
FROM bmp.updates
WHERE has(as_path, YOUR_ASN)
  AND announced = true
  AND time_received_ns >= now() - INTERVAL 1 HOUR
GROUP BY peer_addr, peer_asn, as_path
ORDER BY updates DESC
LIMIT 20 FORMAT CSVWithNames"
```

### Measure AS Path Prepending Effectiveness

If you applied AS path prepending (see [Routing Configurations](/docs/peering/routing-configurations/#as-path-prepending)), verify its effect:

```sh
curl -X POST "https://nxthdr.dev/api/query/" \
  -u "read:read" \
  -H "Content-Type: text/plain" \
  -d "SELECT
    peer_addr,
    as_path,
    length(as_path) AS path_length
FROM bmp.updates
WHERE has(as_path, YOUR_ASN)
  AND announced = true
  AND time_received_ns >= now() - INTERVAL 1 HOUR
ORDER BY time_received_ns DESC
LIMIT 20 FORMAT CSVWithNames"
```

You should see your ASN appearing multiple times in the `as_path` for peers where prepending is applied.

## Monitor Community Propagation

### Check Communities on Your Announcements

If you tagged your routes with BGP communities, verify they propagate:

```sh
curl -X POST "https://nxthdr.dev/api/query/" \
  -u "read:read" \
  -H "Content-Type: text/plain" \
  -d "SELECT
    time_received_ns,
    peer_addr,
    prefix_addr,
    prefix_len,
    communities,
    large_communities
FROM bmp.updates
WHERE peer_asn = YOUR_ASN
  AND announced = true
  AND time_received_ns >= now() - INTERVAL 1 HOUR
ORDER BY time_received_ns DESC
LIMIT 20 FORMAT CSVWithNames"
```

### Find Routes with Specific Communities

Query the dataset for routes carrying a specific community value:

```sh
curl -X POST "https://nxthdr.dev/api/query/" \
  -u "read:read" \
  -H "Content-Type: text/plain" \
  -d "SELECT
    peer_addr,
    prefix_addr,
    prefix_len,
    as_path,
    communities
FROM bmp.updates
WHERE hasAll(communities, [(YOUR_ASN, 100)])
  AND announced = true
  AND time_received_ns >= now() - INTERVAL 1 HOUR
ORDER BY time_received_ns DESC
LIMIT 20 FORMAT CSVWithNames"
```

## Analyze Routing Dynamics

### BGP Update Frequency

Measure how many BGP updates your prefix generates over time, which helps identify route instability:

```sh
curl -X POST "https://nxthdr.dev/api/query/" \
  -u "read:read" \
  -H "Content-Type: text/plain" \
  -d "SELECT
    toStartOfMinute(toDateTime(time_received_ns / 1000000000)) AS minute,
    countIf(announced = true) AS announcements,
    countIf(announced = false) AS withdrawals
FROM bmp.updates
WHERE peer_asn = YOUR_ASN
  AND time_received_ns >= now() - INTERVAL 1 HOUR
GROUP BY minute
ORDER BY minute DESC
FORMAT CSVWithNames"
```

A healthy prefix should show a single announcement per peer with no withdrawals (except during intentional changes).

### Compare Announcements Across IXPs

If you announce from multiple IXP peers, compare how each one receives your routes:

```sh
curl -X POST "https://nxthdr.dev/api/query/" \
  -u "read:read" \
  -H "Content-Type: text/plain" \
  -d "SELECT
    peer_addr,
    countIf(announced = true) AS announcements,
    countIf(announced = false) AS withdrawals,
    max(toDateTime(time_received_ns / 1000000000)) AS last_update
FROM bmp.updates
WHERE peer_asn = YOUR_ASN
  AND time_received_ns >= now() - INTERVAL 1 DAY
GROUP BY peer_addr
ORDER BY last_update DESC
FORMAT CSVWithNames"
```

## Combine with Traffic Data

You can cross-reference BGP data with sFlow traffic data to observe how routing changes affect actual traffic patterns.

### Traffic to Your Prefix

Check if traffic is reaching your announced prefix:

```sh
curl -X POST "https://nxthdr.dev/api/query/" \
  -u "read:read" \
  -H "Content-Type: text/plain" \
  -d "SELECT
    toStartOfMinute(toDateTime(time_received_ns / 1000000000)) AS minute,
    sum(bytes * sampling_rate) AS estimated_bytes,
    count(*) AS flow_samples
FROM flows.records
WHERE isIPAddressInRange(dst_addr, 'YOUR_PREFIX/48')
  AND time_received_ns >= now() - INTERVAL 1 HOUR
GROUP BY minute
ORDER BY minute DESC
FORMAT CSVWithNames"
```

### Correlate Routing Changes with Traffic Shifts

By running BGP and traffic queries over the same time window, you can observe how announcing or withdrawing a prefix affects the volume and source of traffic you receive. This is particularly useful for studying:

- **Convergence time**: How quickly traffic arrives after a new announcement
- **Withdrawal impact**: How fast traffic drops after a withdrawal
- **Prepending effects**: Whether AS path prepending shifts traffic between IXPs

## Combine with Probing Data

If you also use [Saimiris](/docs/probing/) (the nxthdr probing platform), you can study how your routing changes affect reachability and latency.

### Measure Reachability to Your Prefix

Submit probes targeting your announced prefix and observe how path changes affect latency:

```sh
curl -X POST "https://nxthdr.dev/api/query/" \
  -u "read:read" \
  -H "Content-Type: text/plain" \
  -d "SELECT
    agent_id,
    probe_dst_addr,
    round(avg(rtt), 2) AS avg_rtt_us,
    count(*) AS probes
FROM saimiris.replies
WHERE isIPAddressInRange(probe_dst_addr, 'YOUR_PREFIX/48')
  AND time_received_ns >= now() - INTERVAL 1 HOUR
GROUP BY agent_id, probe_dst_addr
ORDER BY avg_rtt_us DESC
FORMAT CSVWithNames"
```

This enables joint peering and probing studies — one of the unique research capabilities of the nxthdr platform.

## Experiment Workflow

A typical experiment cycle using BGP data feedback:

1. **Configure** your routing policy in BIRD (see [Routing Configurations](/docs/peering/routing-configurations/))
2. **Announce** your prefix and wait for BGP convergence (typically seconds to minutes)
3. **Verify** the announcement using the queries above
4. **Observe** traffic patterns and community propagation
5. **Adjust** your configuration based on the data (e.g., add prepending, change communities)
6. **Repeat** and compare results across iterations

All historical data remains available in the datasets, so you can revisit and analyze past experiments at any time.
