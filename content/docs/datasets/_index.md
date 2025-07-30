---
title: Datasets
weight: 4
prev: /docs/measurements/ethics/
next: /docs/datasets/enrich/
---

The **nxthdr** platform makes all collected data freely available to the research community. This section provides comprehensive documentation for accessing and querying our datasets, which include active measurement results, BGP routing data, and traffic flow information.

You can find the equivalent but more interactive documentation on our [website](https://nxthdr.dev/docs/datasets/).

### Data Licensing

These datasets are made available under the **Public Domain Dedication and License v1.0** whose full text can be found at: [http://opendatacommons.org/licenses/pddl/1.0/](http://opendatacommons.org/licenses/pddl/1.0/).

## Access & Authentication

To access the data, use basic HTTP authentication with the following public credentials:

```
Endpoint: https://nxthdr.dev/api/query/
Username: read
Password: read
```

These credentials are intentionally public and can be freely shared. Authentication exists solely to prevent automated scraping while allowing legitimate access.

You can query the datasets using:

* Command line: curl with SQL queries (examples below)
* ClickHouse client: Official CLI tool with native protocol
* Programming languages: HTTP libraries in Python, R, JavaScript, etc.
* Analytics platforms: Jupyter notebooks, R Studio, or custom applications

All queries must be compatible with ClickHouse SQL. See the examples below for each dataset to get started.


## Available Datasets

### Probing Dataset

The probing dataset is available in the `saimiris.replies` table. Active measurements using [saimiris](https://github.com/nxthdr/saimiris), whether scheduled via cron jobs or performed on demand by the users, are stored in a ClickHouse database. This data consists of traceroute-like and ping-like measurement results collected from multiple vantage points.

Each row corresponds to a measurement result, capturing the source and destination IP addresses of the sent packet, the reply, the hop count, and other relevant attributes. This dataset is ideal for network topology discovery, latency analysis, and path characterization.

**Example Query**

Find the top 10 destinations with the highest average RTT from the CDG agent in the last hour:

```sh
curl -X POST "https://nxthdr.dev/api/query/" \
  -u "read:read" \
  -H "Content-Type: text/plain" \
  -d "SELECT probe_dst_addr,
       count(*) as measurements,
       round(avg(rtt), 2) as avg_rtt_us
FROM saimiris.replies
WHERE time_received_ns >= now() - INTERVAL 1 HOUR
  AND agent_id = 'vltcdg01'
GROUP BY probe_dst_addr
ORDER BY avg_rtt_us DESC
LIMIT 10 FORMAT CSVWithNames"
```

### Peering Dataset

The peering dataset is available in the `bmp.updates` table. Each router of as215011, including those inside IXPs, sends BMP messages to [risotto](https://github.com/nxthdr/risotto), which records the updates in a ClickHouse database.

Each row corresponds to an update or a withdraw, capturing prefixes, AS paths, communities, and other attributes. This dataset is perfect for analyzing BGP routing dynamics, prefix announcements, and AS relationship studies.

**Example Query**

Example Query
Find prefixes with the most BGP communities in the last 24 hours:

```sh
curl -X POST "https://nxthdr.dev/api/query/" \
  -u "read:read" \
  -H "Content-Type: text/plain" \
  -d "WITH concat(prefix_addr, '/', prefix_len) AS prefix
SELECT prefix,
       max(length(communities)) AS n_communities
FROM bmp.updates
WHERE time_received_ns >= now() - INTERVAL 1 DAY
GROUP BY prefix
ORDER BY n_communities DESC
LIMIT 5 FORMAT CSVWithNames"
```

### Traffic Dataset

The traffic dataset is available in the `flows.records` table. Each router of as215011 sends sFlow messages to [goflow2](https://github.com/netsampler/goflow2), which records the flow samples in a ClickHouse database.

Each row represents a sampled network flow, capturing traffic statistics between source and destination endpoints. This dataset is more useful for internal troubleshooting, but is public for transparency and possible research use cases.


**Example Query**

Find the top 10 destination IP addresses by average bandwidth in the last 24 hours:

```sh
curl -X POST "https://nxthdr.dev/api/query/" \
  -u "read:read" \
  -H "Content-Type: text/plain" \
  -d "SELECT
  dst_addr,
  sum(bytes * sampling_rate) / (24 * 3600) as avg_bytes_per_second,
  count(*) as flow_records,
  sum(bytes * sampling_rate) as total_estimated_bytes
FROM flows.records
WHERE time_flow_start_ns >= now() - INTERVAL 1 DAY
GROUP BY dst_addr
ORDER BY avg_bytes_per_second DESC
LIMIT 10 FORMAT CSVWithNames"
```
