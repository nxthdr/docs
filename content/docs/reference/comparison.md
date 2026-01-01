---
title: Platforms Comparison
weight: 3
---

This section compares **nxthdr** with other popular platforms for Internet research. nxthdr provides two integrated platforms: Saimiris for active measurements and PeerLab for BGP and routing experiments. The comparison focuses on key features, data access, and capabilities to help you choose the right platform for your research needs.

Please note that information about other platforms is based on publicly available documentation and may not reflect the latest changes. Always refer to the official documentation of each platform for the most accurate and up-to-date information. Feel free to [open an issue](https://github.com/nxthdr/docs/issues) if you find any discrepancies or have questions.

## What Makes nxthdr Unique

nxthdr is the only platform that integrates both active measurements and BGP experimentation in a single, cohesive environment:

### Integrated Platform Advantage

Unlike other platforms that focus on either measurements OR BGP experiments, nxthdr provides both Saimiris (probing) and PeerLab (peering) under one roof. This integration enables:

* **Joint control and data plane studies**: Announce your own prefixes via PeerLab and immediately measure how routing changes affect reachability and latency using Saimiris
* **End-to-end routing research**: Study the complete picture from BGP announcements to actual packet delivery
* **Unified data access**: All BGP data, traffic data, and measurement results available in one place under [PDDL](https://opendatacommons.org/licenses/pddl/1-0/)
* **Seamless workflows**: No need to coordinate between separate platforms or merge datasets from different sources

### Platform-Specific Advantages

Saimiris (Probing Platform)
* Anycast measurement support (unique capability)
* Full control over probe parameters and source addressing
* Framework for measurement tool development

PeerLab (Peering Platform)
* Lease dedicated IPv6 prefixes
* Automated Docker-based setup
* Real BGP experiments on production Internet

Both Platforms
* Truly open data with no authentication, delays, or licensing restrictions ([PDDL](https://opendatacommons.org/licenses/pddl/1-0/))
* No application or approval process required
* Immediate access to both platforms

## Peering Platform Comparison

This section compares PeerLab with the PEERING testbed, the leading platform for BGP experimentation.

### BGP Experimentation Capabilities

| Feature                               | PEERING Testbed                                   | nxthdr (PeerLab)                                     |
| ------------------------------------- | ------------------------------------------- | ----------------------------------------- |
| **Access requirements**     | Application required; academic use only                | Open signup, no approval needed    |
| **Prefix allocation**       |  Prefix pool or bring your own       | Prefix pool as a service or bring your own                             |
| **Source ASNs**       | 7 ASNs       | 1 ASN (AS215011) |
| **IXP locations**       | 5 IXPs worldwide (NA, EU, SA) + Universities + Vultr Cloud       | 4 IXPs in EU + Optional Vultr Cloud |
| **Connection method**      | Remote peering via tunnels                                     | Remote peering via Headscale overlay                                         |
| **Setup complexity**                    | Manual BGP configuration required                                       | Automated via Docker (PeerLab)   |
| **BGP data access**               | Limited; primarily for your own announcements                                         | Full BGP feeds and traffic data (sFlow) from AS215011           |
| **Integrated measurements**       | No (separate platform needed)                                          | Yes (combined with Saimiris)  |
| **Data licensing**       | Varies by use; research-focused                                          | Public domain ([PDDL](https://opendatacommons.org/licenses/pddl/1-0/))  |
| **IPv4/IPv6 support**       | Both IPv4 and IPv6                                          | IPv6 only (currently)  |

### Advantages Summary

**PEERING Testbed** is better if you need:
* IXP presence across multiple continents
* Multiple source ASNs (7 ASNs)
* Both IPv4 and IPv6 support
* Established platform with long operational history
* Large academic collaboration network

**nxthdr (PeerLab)** is better if you need:
* No application required: immediate access for anyone
* Prefix leasing as a service to lease IPv6 prefixes directly from the platform
* Automated setup via Docker, Headscale and BIRD for quick experimentation
* Full BGP and traffic data from AS215011 made publicly available
* Integrated with Saimiris for joint peering and measurement studies
* Truly open data with no licensing restrictions ([PDDL](https://opendatacommons.org/licenses/pddl/1-0/))

## Probing Platform Comparison

### User-Defined Measurements

| Feature                               | CAIDA Ark                                   | RIPE Atlas                             | NLNOG Ring                                     | nxthdr (Saimiris)                                   |
| ------------------------------------- | ------------------------------------------- | -------------------------------------- | ---------------------------------------------- | ----------------------------------------- |
| **Access requirements**     | Limited to vetted researchers                | Open with daily quotas       | Requires participation (host a node)                             | Open signup, no approval needed    |
| **Daily quotas**       | N/A (vetted access)       | ~100k results/day       | No enforced quotas                             | 10k results/day (currently) |
| **Supported protocols**       | Scamper (ping, traceroute, HTTP, DNS)       | Ping, traceroute, DNS, HTTP, NTP       | Any Linux CLI tool                             | ICMP/UDP probes |
| **Source locations**      | ~220 cities / 70 countries                                     | ~1,000+ cities / 187 countries                                 | ~300 cities / 60 countries                                        | 2 cities / 2 countries (expanding to ~40) |
| **Source ASes**                    | ~200                                       | ~4,000                          | ~547                                           | 1 (AS215011)   |
| **IPv4 / IPv6 support**               | Both                                         | Both                                    | Both                                            | IPv6 only (currently)           |
| **Anycast measurement support**       | No                                          | No                                     | No                                             | Yes  |
| **BGP joint experiments**       | No                                          | No                                     | No                                             | Yes (via PeerLab and Saimiris)  |

### Measurement Framework Comparison

PKTLAB is a measurement framework (not a platform) that can be compared with Saimiris in terms of probe input capabilities:

| Feature                          | PKTLAB Framework                                  | nxthdr (Saimiris + Prowl)                              |
| -------------------------------- | ------------------------------------------ | ----------------------------------- |
| **Type** | Framework (self-hosted) | Platform as a service |
| **Probe specification**               | Custom probe definitions              | Caracal probe specification        |
| **Infrastructure**               | Bring your own servers       | Managed infrastructure (AS215011) |
| **Setup complexity** | Requires deployment and configuration | Sign up and start measuring |
| **Data storage**               | Self-managed       | Managed ClickHouse database |
| **Data access**               | Local only       | Public API, no authentication |

### Advantages Summary

**RIPE Atlas** is better if you need:
* Extensive geographic coverage (1,000+ locations worldwide)
* IPv4 measurements
* Diverse source ASes (4,000+)
* Established, mature platform with long-term stability
* Higher daily measurement quotas if you have credits

**CAIDA Ark** is better if you need:
* Long-term curated datasets
* Historical data archives
* Established research datasets for reproducibility
* Vetted research environment

**NLNOG Ring** is better if you need:
* Full shell access to measurement nodes
* Ability to run any Linux CLI tool
* No protocol restrictions
* Direct server access

**M-Lab** is better if you need:
* Long-term retention of performance data
* Large-scale speedtest and NDT data from end users
* TCP-based performance measurements

**nxthdr (Saimiris)** is better if you need:
* Anycast measurement support (unique capability)
* No application required: immediate access for anyone
* Integration with BGP experiments (PeerLab) for joint studies
* Full control over probe parameters and source addressing
* Truly open data with no authentication or licensing restrictions ([PDDL](https://opendatacommons.org/licenses/pddl/1-0/))
* All measurement data publicly available

## Datasets

| Feature                          | CAIDA Ark                                  | RIPE Atlas                              | M‑Lab                                       | nxthdr                              |
| -------------------------------- | ------------------------------------------ | --------------------------------------- | ------------------------------------------- | ----------------------------------- |
| **Data access** | Public datasets; raw data requires request; delayed availability | Public API; private data with auth      | Public via BigQuery (requires GCP account)   | Direct ClickHouse access, no authentication |
| **Data licensing**               | Academic, non-commercial use only              | Open access under RIPE NCC terms        | Public domain (CC0)   | Public domain ([PDDL](https://opendatacommons.org/licenses/pddl/1-0/)) |
| **Data retention**               | Long-term archives; curated datasets       | Private data ~1 month; public archives | Indefinite retention; 24h publication delay | 7 days (rolling window) |
| **BGP data**               | Yes (RouteViews, BGPStream)       | Limited | No | Yes (full BGP feeds from AS215011) |
| **Traffic data**               | No       | No | Yes (NDT, speedtest) | Yes (sFlow from AS215011) |

## Sources

* **CAIDA Ark**

  * [https://www.caida.org/projects/ark/](https://www.caida.org/projects/ark/)
  * [https://www.caida.org/data/overview/](https://www.caida.org/data/overview/)
  * [https://www.caida.org/about/legal/](https://www.caida.org/about/legal/)

* **RIPE Atlas**

  * [https://atlas.ripe.net/](https://atlas.ripe.net/)
  * [https://atlas.ripe.net/docs/api/](https://atlas.ripe.net/docs/api/)
  * [https://www.ripe.net/manage-ips-and-asns/community/active-measurements/ripe-atlas/data-access](https://www.ripe.net/manage-ips-and-asns/community/active-measurements/ripe-atlas/data-access)
  * [https://www.ripe.net/legal/terms-conditions](https://www.ripe.net/legal/terms-conditions)

* **NLNOG Ring**

  * [https://ring.nlnog.net](https://ring.nlnog.net)
  * [https://ring.nlnog.net/documentation/](https://ring.nlnog.net/documentation/)
  * [https://ring.nlnog.net/users/faq/](https://ring.nlnog.net/users/faq/)

* **M-Lab**

  * [https://www.measurementlab.net/data/](https://www.measurementlab.net/data/)
  * [https://www.measurementlab.net/mlab-scope/](https://www.measurementlab.net/mlab-scope/)
  * [https://www.measurementlab.net/privacy/](https://www.measurementlab.net/privacy/)
  * [https://www.measurementlab.net/tools/](https://www.measurementlab.net/tools/)
  * [https://console.cloud.google.com/marketplace/details/measurement-lab/ndt](https://console.cloud.google.com/marketplace/details/measurement-lab/ndt)
  * [https://github.com/m-lab](https://github.com/m-lab)

* **PEERING Testbed**

  * [https://peering.ee.columbia.edu/](https://peering.ee.columbia.edu/)
  * [https://peering.ee.columbia.edu/tutorial/](https://peering.ee.columbia.edu/tutorial/)
  * [https://peering.ee.columbia.edu/contact/](https://peering.ee.columbia.edu/contact/)

* **PKTLAB**

  * [https://packetlab.github.io/](https://packetlab.github.io/)
  * [https://github.com/packetlab](https://github.com/packetlab)
