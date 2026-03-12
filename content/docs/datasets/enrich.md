---
title: Data Enrichment
weight: 1
prev: /docs/datasets/
---

It is useful to enrich the different dataset data with contextual information such as geolocation, ASN details and network metadata.

## IPinfo

We use [IPinfo](https://ipinfo.io/)'s free Country & ASN database to enrich IP address data. The data is updated daily and loaded into ClickHouse using [`mmdb-to-clickhouse`](https://github.com/maxmouchet/mmdb-to-clickhouse), which creates optimized `ip_trie` dictionaries for fast lookups.

### Lookup Function

Use the `country_asn()` function to enrich any IP address:

```sh
curl -X POST "https://nxthdr.dev/api/query/" \
  -u "read:read" \
  -H "Content-Type: text/plain" \
  -d "SELECT
  country_asn(toIPv6('8.8.8.8'), 'as_name') AS as_name,
  country_asn(toIPv6('8.8.8.8'), 'country') AS country,
  country_asn(toIPv6('8.8.8.8'), 'continent') AS continent
FORMAT CSVWithNames"
```

Returns: `Google LLC, US, NA`

### Available Attributes

| Attribute        | Description              | Example          |
|------------------|--------------------------|------------------|
| `as_name`        | AS organization name     | `Google LLC`     |
| `as_domain`      | AS domain                | `google.com`     |
| `asn`            | Autonomous System Number | `15169`          |
| `country`        | Country code (ISO 3166)  | `US`             |
| `country_name`   | Full country name        | `United States`  |
| `continent`      | Continent code           | `NA`             |
| `continent_name` | Full continent name      | `North America`  |

### ASN-to-Name Dictionary

For direct ASN number lookups (when you have an ASN but not an IP), use the `asn_to_name` dictionary:

```sh
curl -X POST "https://nxthdr.dev/api/query/" \
  -u "read:read" \
  -H "Content-Type: text/plain" \
  -d "SELECT dictGet('ipinfo.asn_to_name', 'as_name', toUInt64(13335)) AS as_name
FORMAT CSVWithNames"
```

Returns: `Cloudflare, Inc.`

This is derived from the same Country & ASN database and is useful when working with ASN values from BGP data (e.g., AS paths).

### Enrichment Examples

Enrich probing results with geolocation:

```sh
curl -X POST "https://nxthdr.dev/api/query/" \
  -u "read:read" \
  -H "Content-Type: text/plain" \
  -d "SELECT
  probe_dst_addr,
  country_asn(probe_dst_addr, 'country') AS country,
  country_asn(probe_dst_addr, 'as_name') AS as_name,
  round(avg(rtt), 2) AS avg_rtt_us
FROM saimiris.replies
WHERE time_received_ns >= now() - INTERVAL 1 HOUR
GROUP BY probe_dst_addr
ORDER BY avg_rtt_us DESC
LIMIT 10 FORMAT CSVWithNames"
```

Enrich BGP updates with AS names:

```sh
curl -X POST "https://nxthdr.dev/api/query/" \
  -u "read:read" \
  -H "Content-Type: text/plain" \
  -d "WITH as_path[1] AS asn
SELECT
  asn,
  dictGet('ipinfo.asn_to_name', 'as_name', asn) AS as_name,
  count(*) AS updates
FROM bmp.updates
WHERE time_received_ns >= now() - INTERVAL 1 DAY AND asn != 0
GROUP BY asn
ORDER BY updates DESC
LIMIT 10 FORMAT CSVWithNames"
```

### Data Source

The enrichment data comes from IPinfo's free [Country & ASN database](https://ipinfo.io/products/free-ip-database), licensed under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/). It covers both IPv4 and IPv6 addresses and is updated daily.
