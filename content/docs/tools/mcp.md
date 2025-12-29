---
title: MCP Servers
weight: 3
next: /docs/internals/
---

Since **nxthdr** datasets are stored in ClickHouse, you can use the official MCP server for ClickHouse [mcp-clickhouse](https://github.com/ClickHouse/mcp-clickhouse) to query the data.

This provides a convenient way to access and analyze the datasets using natural language queries. You can obviously combine this MCP server with others such as the [mcp-server-ipinfo](https://github.com/briandconnelly/mcp-server-ipinfo) to enrich your queries with geolocation and other IP-related information.

## Installation

```yaml
{
    "servers": {
        "mcp-clickhouse-nxthdr": {
            "command": "uv",
            "args": [
                "run",
                "--with",
                "mcp-clickhouse",
                "--python",
                "3.10",
                "mcp-clickhouse"
            ],
            "env": {
                "CLICKHOUSE_HOST": "clickhouse.nxthdr.dev",
                "CLICKHOUSE_PORT": "443",
                "CLICKHOUSE_USER": "read",
                "CLICKHOUSE_PASSWORD": "read",
                "CLICKHOUSE_SECURE": "true",
                "CLICKHOUSE_VERIFY": "true",
                "CLICKHOUSE_CONNECT_TIMEOUT": "30",
                "CLICKHOUSE_SEND_RECEIVE_TIMEOUT": "30"
            }
        }
    },
}
```

## Example Usage

```
Please give me the top 5 prefixes with the highest number of communities
for the last 24 hours.
```

Response using Claude Sonnet 4 (as of July 2025):

```
Top 5 Prefixes by Community Count
2400:cb00:648::/48 - 162 communities

Peer ASN: 47498
This is a Cloudflare IPv6 prefix (2400:cb00::/32 is Cloudflare's APAC range)
::ffff:188.95.92.0/24 - 116 communities

Peer ASN: 34307 (appears to be a transit provider in Brazil/Latin America)
This is an IPv4-mapped IPv6 address for 188.95.92.0/24
2804:4870:2f00::/40 - 107 communities

Peer ASN: 34307
Brazilian IPv6 prefix
2804:5bbc:4000::/34 - 107 communities

Peer ASN: 34307
Brazilian IPv6 prefix
2804:523c:3ffe::/48 - 107 communities

Peer ASN: 34307
Brazilian IPv6 prefix

Key Observations:

Cloudflare dominates: The prefix with the most communities (162)
belongs to Cloudflare,which makes sense given their global
CDN infrastructure and complex traffic engineering requirements.

Brazilian transit provider: ASN 34307 appears to be a major
transit provider in Brazil, responsible for 4 out of the
top 5 prefixes with high community counts.

Community usage patterns: The high number of communities
suggests sophisticated traffic engineering and policy control,
particularly for large content providers and transit networks.

These prefixes likely use BGP communities for advanced traffic
engineering, load balancing, and policy control across their networks
and peering relationships.
```
