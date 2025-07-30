---
title: Collected Data
weight: 1
prev: /docs/as215011/
---

We are performing passive measurements from our network devices, from IXP Servers as well as from our Probing Servers.

Please take a look at our Grafana [dashboards](https://grafana.nxthdr.dev) to see the collected data in real-time. Also take a look at our [datasets](/docs/datasets/) to see how to access the collected data.

### Peering Data

Each of our routers are using the BGP Monitoring Protocol (BMP) ([rfc7854](https://tools.ietf.org/html/rfc7854)) to collect routing data from our peers. We developped our own BMP collector called [risotto](https://github.com/nxthdr/risotto) which stores the data in a ClickHouse table.
This data includes BGP updates (prefix announcements, withdrawals) received and sent by our routers.

### Traffic Data

Each of our routers is also collecting traffic data using the sFlow protocol ([rfc3176](https://tools.ietf.org/html/rfc3176)). We run [goflow2](https://github.com/netsampler/goflow2) which stores the data in a ClickHouse table.
This data includes flow samples, which are mainly used to troubleshoot and monitor the traffic on our network. However, we think this data can potentially be used for research purposes, so we are making it public.
