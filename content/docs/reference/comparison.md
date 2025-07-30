---
title: Platforms Comparison
weight: 3
---

This section compares **nxthdr** with other popular platforms for Internet measurements, such as CAIDA Ark, RIPE Atlas, NLNOG Ring, and M-Lab. The comparison focuses on key features, data access, measurement capabilities, and data retention policies to help you choose the right platform for your research needs.

Please note that information about other platforms is based on publicly available documentation and may not reflect the latest changes. Always refer to the official documentation of each platform for the most accurate and up-to-date information. Feel free to [open an issue](https://github.com/nxthdr/docs/issues) if you find any discrepancies or have questions.

### User-Defined Measurements

| Feature                               | CAIDA Ark                                   | RIPE Atlas                             | NLNOG Ring                                     | nxthdr                                    |
| ------------------------------------- | ------------------------------------------- | -------------------------------------- | ---------------------------------------------- | ----------------------------------------- |
| **Probing restrictions & quotas**     | Limited to vetted researchers                | Daily quotas, \~100k results/day       | No enforced quotas                             | Daily quotas, 10k results/day    |
| **Supported tools & protocols**       | Scamper (ping, traceroute, HTTP, DNS)       | Ping, traceroute, DNS, HTTP, NTP       | Any Linux CLI tool                             | ICMP/UDP probes |
| **Source cities/countries**      | ~220/70                                     | ~1,000+/187                                 | ~300/60                                        | 2/2 |
| **Source ASes**                    | ~200                                       | ~4,000                          | ~547                                           | 1   |
| **IPv4 / IPv6 support**               | Yes                                         | Yes                                    | Yes                                            | Currently only IPv6 support           |
| **Anycast measurement support**       | No                                          | No                                     | No                                             | Yes  |
| **BGP support**                       | No                                          | No                                     | No                                             | Yes  |


### Datasets

| Feature                          | CAIDA Ark                                  | RIPE Atlas                              | M‑Lab                                       | nxthdr                              |
| -------------------------------- | ------------------------------------------ | --------------------------------------- | ------------------------------------------- | ----------------------------------- |
| **Data access & authentication** | Public datasets; raw data requires request; by default with a delay | Public API; private data with auth      | Public via BigQuery and GCP account   | Full access without authentication |
| **Data licensing**               | Academic, non-commercial use; under CAIDA terms              | Open access under RIPE NCC terms        | Public domain (CC0)   | Public domain (PDDL) |
| **Data retention**               | Long-term archives; curated datasets       | Private data \~1 month; archives public | Indefinite retention; 24h publication delay | 7 days |


### Recommendations

Choosing the right platforms depends on multiple factors. **nxthdr** is a great choice if you want to conduct measurements, but:
* you don't have RIPE Atlas credits, access to CAIDA Ark infrastructure, or access to NLNOG Ring
* you need to conduct measurements with high flexibility and control, for instance to design and test new measurement tools
* your measurements methodology is compatible with the platforms limitations: IPv6-only, few probing locations, no TCP probes support and only one source AS
* you need anycast measurements support or you need to conduct measurements combined with BGP experiments

If you only need fresh or long-standing data from common tools such as ping, traceroute, DNS, HTTP, etc., and you agree on their [terms of use](https://www.ripe.net/about-us/legal/ripe-data-repository-terms-and-conditions/), then **RIPE Atlas** is a great choice. It has a large number of probes from around the world and from many source ASes.

**M-LAB** is also a good choice if you need fresh or long-standing TCP and traceroute-like measurements data, notably from a lot of speedtest sources. Moreover, the data is available in the public domain.

Finally, if you are looking for curated datasets, **CAIDA Ark** is a great choice. It provides long-term archives of active measurements and BGP data, but requires you to apply for access and follow their [terms of use](https://www.caida.org/about/legal/).

### Sources

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


## Comparison with PKTLAB

*Content coming soon...*

## Comparison with PEERING

*Content coming soon...*
