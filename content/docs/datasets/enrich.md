---
title: Data Enrichment
weight: 1
prev: /docs/datasets/
---

It is useful to enrich the different dataset data with contextual information such as geolocation, ASN details and network metadata.

## IPinfo

We recommend [IPinfo](https://ipinfo.io/) to enrich IP address data with geolocation and ASN information.

As for now, we directly provide ipinfo's ASN to AS name mapping functionality through our own database. Use the dictionary for fast lookups:

```sh
curl -X POST "https://nxthdr.dev/api/query/" \
  -u "read:read" \
  -H "Content-Type: text/plain" \
  -d "SELECT dictGet('ipinfo.asn_asname', 'as_name', toUInt32(13335)) AS as_name"
```

Returns: `Cloudflare, Inc.`
