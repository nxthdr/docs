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

### Multi-Agent Measurements and Correlated Source Addresses

When sending the same probe set from multiple agents simultaneously — useful for catchment or latency comparison experiments — the CLI generates a single random 48-bit value shared across all agents in that measurement. Each agent's source IP is then derived by embedding this shared value into its own `/80` prefix:

```
nxthdr probing send --agent vltcdg01 --agent vltewr01 --agent vltsgp01 probes.csv
```

The result is a set of source addresses that all share the same lower 48 bits but differ in their upper prefix bits (one per agent). For example:

```
2a0e:97c0:8a4:USER:SHARED::  ← vltcdg01 (Paris)
2a0e:97c0:8a3:USER:SHARED::  ← vltewr01 (New Jersey)
2a0e:97c0:8a5:USER:SHARED::  ← vltsgp01 (Singapore)
```

This design has two useful properties:

1. **Group queries**: you can retrieve replies for the entire measurement from ClickHouse by filtering on any one of the source IPs, since they share the same host bits. All agents' replies appear in the same result set.

2. **Per-agent attribution**: even in anycast mode where replies from multiple sources arrive at a single catchment node, you can tell which agent sent each probe by looking at the `probe_src_addr` prefix — no additional tagging is needed.

The CLI prints the derived source IP for each agent after submitting, along with a ready-to-run results query:

```
hint: nxthdr probing results --src-ip 2a0e:97c0:8a4:... --src-ip 2a0e:97c0:8a3:... --src-ip 2a0e:97c0:8a5:...
```

### Tracking Agents in Anycast Measurements

In anycast mode, a probe may be sent from one agent, but the corresponding reply may be received by a different agent. To determine which agent sent the probe, you can inspect the **source IP address** of the probe.

For example:

If agent `vltatl01` sends a probe using the source IP `2a0e:97c0:8a0:17d9:8e72::10`, and you receive a reply where this IP appears as the `probe_src_addr`, you know that the probe originated from `vltatl01`.

If you want to explicitly encode the agent identity in your probe source address for anycast measurements, you have a couple of options:

* **Embed agent ID bits** into your `/80` address allocation
* **Use another field**, such as the source port, to carry this metadata

The choice is yours, nxthdr gives you full control over probe construction and addressing.
