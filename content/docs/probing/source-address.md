---
title: Source Addresses
weight: 3
---

Each *agent* in **nxthdr** is assigned two IPv6 `/48` prefixes:

* One for *unicast* measurements
* One for *anycast* measurements

The *unicast* prefix is advertised by a single agent, while the *anycast* prefix is advertised by multiple agents simultaneously. This allows both targeted and distributed measurement scenarios.

Each user of the platform is assigned a unique 32-bit *user ID*. This ID is appended to each agent’s `/48` prefixes to derive a unique `/80` prefix for that user. This ensures:

* Users cannot interfere with each other’s measurement space
* Source addresses are user-scoped and globally unique across the platform

With this design, a user can vary the source address of their probes within their allocated `/80`, while still being able to attribute results to their own measurements.

### Tracking Agents in Anycast Measurements

In anycast mode, a probe may be sent from one agent, but the corresponding reply may be received by a different agent. To determine which agent sent the probe, you can inspect the **source IP address** of the probe.

For example:

If agent `vltatl01` sends a probe using the source IP `2a0e:97c0:8a0:17d9:8e72::10`, and you receive a reply where this IP appears as the `probe_src_addr`, you know that the probe originated from `vltatl01`.

If you want to explicitly encode the agent identity in your probe source address for anycast measurements, you have a couple of options:

* **Embed agent ID bits** into your `/80` address allocation
* **Use another field**, such as the source port, to carry this metadata

The choice is yours, nxthdr gives you full control over probe construction and addressing.
