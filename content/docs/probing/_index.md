---
title: Probing Platform
weight: 2
prev: /docs/peering/getting-started/
next: /docs/probing/probes/
---

**Saimiris** is the nxthdr probing platform that enables researchers, students, and network operators to conduct active Internet measurements from multiple global vantage points using production infrastructure.

To get started quickly, check out our [interactive tutorial](https://nxthdr.dev/docs/measurements) and send your first probes directly from our network.

## Platform Status

Saimiris is currently in **alpha**. Core functionality is operational and available for use, but we are actively working to expand our infrastructure.

**Alpha Exit Criteria**

Deploy additional probing servers in more geographic locations (objective: >= 10 servers worldwide).

If you need specific vantage points or have capacity requirements, please [contact us](/docs/reference/contact/).

## Applications

**Internet Topology & Performance Research**

Perform active measurements from multiple global locations and analyze BGP dynamics, CDN behavior, routing patterns, and global connectivity using comprehensive real-world data.

**Measurement Tooling & Algorithm Development**

Build, test, and benchmark innovative measurement techniques and algorithms leveraging our global high-speed infrastructure with full control over source addressing.

**Joint Peering & Probing Studies**

Combine the probing platform with [PeerLab](/docs/peering/) to conduct unique experiments. Study how routing changes affect reachability, latency, and path selection in real-time.

**Education & Learning**

Teach networking courses with real Internet measurements. Students can perform traceroutes and ping measurements from actual network infrastructure across multiple geographic locations.

**Reproducible Research**

Validate and reproduce prior studies with access to consistent infrastructure and open datasets. All measurement results are made freely available through our public [datasets](/docs/datasets/).

## Features

**Global Vantage Points**

Send probes from multiple servers distributed worldwide. Measure Internet topology, path diversity, and latency from different geographic locations and network perspectives.

**Unicast and Anycast Addressing**

Use both unicast and anycast source addresses from AS215011 IPv6 prefixes. Study how anycast routing affects path selection and measure differences in network behavior.

**High-Speed Measurements**

Perform traceroute-like and ping-like measurements at scale. Our infrastructure is designed to handle large measurement campaigns efficiently.

**Open Data**

All measurement results are stored in a ClickHouse database and made freely available. Query your own measurements and analyze historical data using our public [datasets](/docs/datasets/).

## What Are Active Measurements?

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

If you need more credits, please [contact us](/docs/reference/contact).
