---
title: Limitations
weight: 5
previous: /docs/probing/security-compliance/
next: /docs/datasets
---

While nxthdr provides a powerful and flexible platform for active Internet measurements, there are some current limitations to be aware of when designing your experiments. We’re actively working to improve these areas, but we want to be transparent so you can plan accordingly.

### Single Source AS

All probes are currently sent from our own Autonomous System: AS215011. **This is by design**. It helps maintain control over routing behavior during experiments.

We do *not* aim to compete with large-scale platforms like RIPE Atlas, which support probes from thousands of ASes. Instead, nxthdr focuses on offering controlled, customizable measurement capabilities. See the [platforms comparison](/docs/reference/comparison/) for more information.

### IPv6-Only Measurements

At this time, nxthdr only supports IPv6-based measurements. This is because our Autonomous System ([AS215011](https://www.peeringdb.com/net/36080)) only announces IPv6 prefixes for now.

We plan to support IPv4 in the future by leveraging cloud providers’ IPv4 space. If you'd like to contribute public IPv4 prefixes or infrastructure, please [contact us](/docs/reference/contact).

### Limited Probing Locations

Currently, we operate a small number of probing locations, due to financial constraints. Expanding coverage is straightforward: by deploying additional cloud instances and partnering with more providers.

If you can offer access to cloud instances, IXP-connected servers, or are willing to help us diversify infrastructure, we’d love to hear from you.

### Short Data Retention

Data collected by nxthdr is currently retained for 7 days. After that, it is permanently deleted.

This limitation is due to infrastructure constraints, but we are exploring ways to extend retention. If you have cloud credits, storage capacity, or resources to help us increase retention time, please get in touch!

### No TCP Probe Support

nxthdr currently only supports ICMP and UDP probes. We use our own probing agent software, [saimiris](https://github.com/nxthdr/saimiris), built on top of the [caracat](https://github.com/maxmouchet/caracat) probing engine.

TCP probes are not yet implemented in caracat. If this functionality is important for your work, we welcome contributions. Feel free to help implement [this feature](https://github.com/maxmouchet/caracat/issues/13)!
