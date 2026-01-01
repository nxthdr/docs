---
title: Security & Compliance
weight: 3
prev: /docs/peering/getting-started/
next: /docs/peering/limitations/
---

When using PeerLab, you must follow these security requirements. For complete details, see our [terms of service](/docs/reference/terms/).

**Route Announcements**

You can only announce IPv6 prefixes that have been explicitly leased to you through the [nxthdr.dev dashboard](https://nxthdr.dev/peering). Attempting to announce unauthorized prefixes will result in your announcements being filtered and your access being potentially suspended.

**ASN Assignment**

Each user is assigned a unique private ASN. You cannot use another user's ASN or modify your assigned ASN. This ensures proper isolation between users and prevents routing conflicts.

**Hosted Content**

Any content you host using your leased prefixes, even temporarily, must comply with applicable laws and regulations. This includes but is not limited to:

* No malicious or harmful content
* No copyright infringement
* No spam or abuse
* Compliance with applicable laws and regulations

Violations of these requirements may result in immediate suspension of your access to PeerLab and revocation of your leased prefixes.

## Monitoring

We implement comprehensive monitoring to ensure compliance with security requirements and provide transparency:

**Current Monitoring**

* BGP/BMP announcements and traffic utilization (sFlow) are monitored and made public through our datasets (see [datasets section](/docs/datasets/))
* Abuse [contact](/docs/reference/contact/) monitoring to handle complaints
* All monitoring data is auditable by anyone

This public monitoring enables:

* Verification that users only announce their leased prefixes
* Detection of unauthorized route announcements
* Transparency in network traffic patterns
* Community oversight of PeerLab operations

**Future Enhancements**

* Automated daily IP reputation checks to detect if user prefixes appear on public blocklists
* Traffic anomaly detection to alert on unusual patterns such as sudden spikes or port scanning activity
