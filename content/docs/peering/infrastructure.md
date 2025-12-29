---
title: Network Infrastructure
weight: 1
prev: /docs/peering/
next: /docs/peering/getting-started/
---

We operate [AS215011](https://www.peeringdb.com/net/36080), an IPv6-only research-focused Autonomous System used to support routing and peering experiments, and to power the active measurement infrastructure behind **nxthdr**.

AS215011 is connected to several Internet Exchange Points (IXPs) in Europe and announces multiple prefixes for different purposes. All routing data collected by our routers is made freely available in our [datasets](/docs/datasets/).

## IXP Servers

Our IXP-connected servers are deployed at strategic exchange points across Europe. These systems advertise the prefix `2a06:de00:50::/44`.

This prefix is used for interconnection and experimentation with other networks. These servers allow us to maintain a stable presence at the edge of the Internet, participate in routing experiments, and support peering via IXPs. We also use this prefix to connect the **nxthdr** services to the global Internet, dogfooding our own network.

**Current IXP locations:**

```
FogIXP – Frankfurt
FranceIX - Paris
LOCIX  – Frankfurt
NL-ix  – Amsterdam
```

## Probing Servers

Our measurement infrastructure relies on a distributed set of probing servers, which are used to generate active measurements such as ping and traceroute. These servers advertise two distinct IPv6 /48 prefixes:

* **Anycast prefix**: shared across all probing servers
* **Unicast prefix**: unique to each server

Both are subprefixes of `2a0e:97c0:8a0::/44`, which is our primary prefix for active measurements.

At the moment, we don't offer the capability to peer with our probing server through PeerLab. Let us know (see [contact](/docs/reference/contact/)) if you're interested.

## Peering with AS215011

If you operate your own Autonomous System and want to peer with AS215011 as another network (not using the platform as a user), we welcome direct peering connections.

### Peering Options

**Direct IXP Peering**

If you have presence at one of our IXP locations, you can establish a direct BGP session over the IXP fabric. [Contact us](/docs/reference/contact/) with your IXP details to coordinate the peering session.

**Remote Peering via Tunnel**

For network operators without physical presence at our IXPs, we support remote peering through tunneled sessions (WireGuard). This allows you to establish BGP sessions with AS215011 from your own infrastructure.

This option is suitable if you plan to do your own peering experiments using your own network and AS215011. However, if so, please [contact us](/docs/reference/contact/) first to discuss your requirements.

### Routing Policy

* **Open peering policy**: AS215011 welcomes peers at our IXPs or over tunneled sessions.
* **RPKI enforcement**: We perform RPKI validation and filter invalid routes, refreshing every 900 seconds.
* **Geofeed**: Our geolocation data is publicly available: [geofeed.nxthdr.dev](https://geofeed.nxthdr.dev)

You can view a list of current peers here: [peers.nxthdr.dev](https://peers.nxthdr.dev).

## Transparency and Security

We embrace radical transparency. The full configuration of our AS215011 routers is openly available in our [infrastructure repository](https://github.com/nxthdr/infrastructure/tree/main/networks).

If you identify a security issue or misconfiguration, you are encouraged to open an issue or submit a pull request, or [contact us](/docs/reference/contact/) directly. We take security seriously and will work with you to resolve any issues promptly.
