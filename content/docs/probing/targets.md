---
title: Targets Lists
weight: 2
---

Now we understand how to design probes, it's time to choose some targets to send them to. Since currently **nxthdr** probing only supports IPv6, we are spoilt for choice. But, many of those IPv6 addresses are not routed, or not reachable.

One very useful source of IPv6 targets is the [IPv6 Hitlist](https://ipv6hitlist.github.io/#), which provide weekly generated responsive IPv6 addresses, aliased prefixes and non-aliased prefixes. Aliased prefixes refers to IPv6 subnets where every address within the prefix responds to a probe, typically due to being handled by the same host or system (but not always).
