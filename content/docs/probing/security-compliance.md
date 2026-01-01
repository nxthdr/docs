---
title: Security & Compliance
weight: 4
prev: /docs/probing/source-address
next: /docs/probing/limitation
---

When using Saimiris, you must conduct measurements responsibly and ethically. For complete details on requirements and prohibited activities, see our [terms of service](/docs/reference/terms/).

Key requirements include:

* No denial of service attacks or flooding
* No network intrusion or unauthorized security testing
* No espionage or surveillance activities
* Respect for target networks and abuse complaints
* Compliance with applicable laws and regulations

## Monitoring

We implement comprehensive monitoring to ensure compliance with security requirements and provide transparency:

**Current Monitoring**

* Measurement results are made public through our datasets (see [datasets section](/docs/datasets/))
* Abuse [contact](/docs/reference/contact/) monitoring to handle complaints

**Future Enhancements**

* Automated daily IP reputation checks to detect if peering prefixes appear on public blocklists
* Per user probe statistics to track usage patterns and detect anomalies
* Target blocklist filtering to prevent probes from reaching blocked destinations, regardless of TTL value
