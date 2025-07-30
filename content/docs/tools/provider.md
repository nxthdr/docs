---
title: Provider Tools
weight: 4
next: /docs/internals/
---

Opportunistically, we develop "provider" tools and systems for a wider audience. These tools are designed to be used even by those who are not directly involved in Internet measurements, and we are developing them as part of our journey maintaining an autonomous system connected with the Internet.

## DynDNS

This is a simple dynamic DNS (dynDNS) server that allows to update A/AAAA/TXT records for a given domain.

Similar to services like DuckDNS, but you can host it yourself easilly. Currently, it only supports Porkbun's backend for DNS updates. Future updates may include support for additional DNS providers.

Code is available on [GitHub](https://github.com/nxthdr/dyndns) (MIT license).

## ROVCheck

Simple tool to check if your AS (ISP, own AS, etc.) is correctly implementing RPKI ROV. Basically isbgpsafeyet but as a CLI tool.

Code is available on [GitHub](https://github.com/nxthdr/rovcheck) (MIT license).
