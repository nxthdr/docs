---
title: Probes Definition
weight: 1
prev: /docs/measurements/
---

Measurements are performed by servers called *agents* (also known as *vantage points*) deployed at various locations around the world. These *agents* send network packets called *probes* to the Internet and listen for *replies*. Probes can use different transport protocols, such as ICMP, UDP, or TCP. Unlike UDP and TCP, which include source and destination port fields, ICMP does not use ports.

A *probe* is sent to a *target*—a specific IP address. Like any packet on the Internet, a probe is forwarded across multiple *hops* before reaching its destination. Each hop is a router that forwards the packet to the next until it arrives. Every IPv4 or IPv6 packet includes a TTL (Time To Live) or Hop Limit field that is decremented by one at each hop. When the TTL reaches zero, the router discards the packet and returns an ICMP "Time Exceeded" message to the sender.

Our agents follow the [Caracal probe specification](https://dioptra-io.github.io/caracal/usage/), which defines a standard format for measurement probes. Each probe is defined by five parameters that specify how the packet should be constructed and sent:

* **Destination address**: An IPv6 or IPv4-mapped IPv6 address (e.g., `2001:4860:4860::8888` or `::ffff:8.8.8.8`).
* **Source port**: A port number between 0–65535. For UDP probes, it is encoded in the UDP header. For ICMP probes, it's used to vary the flow ID via the checksum.
* **Destination port**: A port number between 0–65535, encoded in the UDP header (UDP probes only).
* **TTL**: The Time-To-Live or Hop Limit value that determines how many hops the packet can traverse.
* **Protocol**: The protocol used by the probe, either `icmpv6` or `udp`.

### Example

```
2001:4860:4860::8888,2000,33434,64,udp
```

This probe targets `2001:4860:4860::8888` (Google’s public DNS), with a source port of `2000`, destination port `33434`, a TTL of `64`, using the UDP protocol.

In this case, the packet will likely reach its destination, but since port `33434` is not open on the target, it will respond with an ICMP "Port Unreachable" message.

If the same probe is sent with a low TTL (e.g., `1`), it won’t reach the destination. Instead, the first router will drop the packet and reply with an ICMP "Time Exceeded" message. The source IP address of that reply is usually the router that dropped the packet.

By sending multiple probes with increasing TTL values, we can trace the path that packets take through the network—this is known as a traceroute-like measurement. Each ICMP "Time Exceeded" reply reveals the IP address of a router along the path.
