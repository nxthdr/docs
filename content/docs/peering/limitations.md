---
title: Limitations
weight: 4
prev: /docs/peering/security-compliance/
next: /docs/probing/
---

While PeerLab provides a powerful platform for peering experiments and IPv6 connectivity, there are some current limitations to be aware of. We're actively working to improve these areas, but we want to be transparent so you can plan accordingly.

For a detailed comparison of PeerLab with other peering platforms like PEERING, see our [platforms comparison](/docs/reference/comparison/) section.

### IPv6-Only Prefixes

PeerLab currently only supports /48 IPv6 prefix leases, since our Autonomous System ([AS215011](https://www.peeringdb.com/net/36080)) only announces IPv6 prefixes for now.

If you'd like to contribute public IPv4 prefixes or infrastructure, please [contact us](/docs/reference/contact).

### Limited Number of Prefix Leases

Due to financial constraints, we can only lease a limited number of prefixes to users at this time. Our allocation is finite, and we need to ensure fair distribution among researchers and experimenters.

If you need additional prefixes or have specific requirements, please [contact us](/docs/reference/contact) to discuss your use case.

### Limited IXP Coverage

Currently, we are only present at a small number of Internet Exchange Points (IXPs) in Europe. See the [infrastructure section](/docs/peering/infrastructure/) for the complete list of IXPs where we peer.

Unlike platforms such as PEERING, we do not yet support lease announcements from cloud providers like Vultr. This limitation is primarily due to financial constraints.

We are working to expand our presence to more IXPs and regions. If you can help us gain access to additional peering locations or cloud infrastructure, we'd love to hear from you.

