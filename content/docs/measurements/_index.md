---
title: Measurements
weight: 2
prev: /docs/as215011/experiments/
next: /docs/measurements/probes/
---

This section covers the design and execution of network measurement campaigns using the **nxthdr** probing infrastructure. You'll learn how to configure probes and understand the trade-offs and implications of different measurement strategies.

To get started quickly, check out our [interactive tutorial](https://nxthdr.dev/docs/measurements) and send your first probes directly from our network.

### What Are Active Measurements?

**Active measurements** involve sending controlled traffic across the Internet to collect data about network performance, topology, and behavior. They are widely used by researchers, network operators, and engineers to:

* Understand routing dynamics
* Diagnose connectivity or latency issues
* Evaluate the impact of routing or configuration changes
* Map Internet paths and detect anomalies

They are called *active* because they intentionally generate traffic, unlike *passive* measurements, which analyze existing traffic flows without injecting new packets. Examples of passive measurements include monitoring BGP updates, RPKI state, or observing flow data on routers.

## Credits

To ensure fair and responsible use of the platform, we limit the number of probes each user can send by using a credit system.

Each user is allocated `10,000` measurement credits per day, which are reset at midnight UTC.

* 1 credit = 1 probe
* This means you can send up to `10,000` probes per day across all your experiments.

If you need more credits, please contact us via [Discord](https://discord.gg/KRsVs7jafg) or [email](mailto:admin@nxthdr.dev).
