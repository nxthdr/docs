---
title: Getting Started
weight: 2
prev: /docs/peering/infrastructure/
next: /docs/peering/security-compliance/
---

This guide will help you start using the PeerLab peering platform to conduct BGP and routing experiments.

## Prerequisites

Before you begin, you should have:

* Basic understanding of BGP and IPv6 routing
* Docker and Docker Compose installed on your system
* An account on [nxthdr.dev](https://nxthdr.dev)

You do not need your own Autonomous System Number (ASN). We will assign you a private ASN (e.g., 64512) and you will be able to announce prefixes leased from our allocation.

## Step 1: Lease IPv6 Prefixes

1. Navigate to the [peering dashboard](https://nxthdr.dev/peering)
2. Request /48 IPv6 prefix allocation

Your leased prefixes will be displayed on the dashboard and ready to announce.

## Step 2: Setup PeerLab

PeerLab is a Docker-based environment that handles all the complexity of connecting to our IXP servers and establishing BGP sessions.

### Install PeerLab

Clone the PeerLab repository:

```bash
git clone https://github.com/nxthdr/peerlab.git
cd peerlab
```

### Configure Your Environment

Create your configuration file:

```bash
cp .env.example .env
```

Edit `.env` and configure your settings:

```bash
# Your private ASN (use any private ASN like 64512)
USER_ASN=64512

# Your leased prefixes from nxthdr.dev (comma-separated)
USER_PREFIXES=2001:db8:1234::/48,2001:db8:5678::/48
```

Leave `USER_PREFIXES` empty if you only want to receive routes without advertising.

## Step 3: Connect to Headscale Network

PeerLab uses Headscale (a self-hosted Tailscale control server) to create a secure network overlay connecting your local environment to our IXP servers.

Start the Tailscale container:

```bash
make setup
```

In a new terminal, authenticate with Headscale:

```bash
make auth
```

This will output a URL like:
```
To authenticate, visit:
https://headscale.nxthdr.dev/register/<key>
```

Open this URL in your browser and authenticate with your nxthdr.dev account.

## Step 4: Start BGP Sessions

Once authenticated, start the full PeerLab stack:

```bash
make up
```

This will:
1. Configure the BIRD BGP daemon with your ASN and prefixes
2. Automatically establish BGP sessions with our IXP servers
3. Start a webserver for connectivity tests (accessible on port 80)

Check the status of your BGP sessions:

```bash
make status
```

After a few moments, you should see BGP sessions in "Established" state.

## Monitor Your Announcements

You can verify that your BGP announcements are being received by querying our ClickHouse database, which stores all BGP updates collected via BMP (BGP Monitoring Protocol).

### Check All Your Announcements

To see all announcements received from your ASN (using ASN `65000` as an example):

```sh
curl -X POST "https://nxthdr.dev/api/query/" \
  -u "read:read" \
  -H "Content-Type: text/plain" \
  -d "SELECT
    time_received_ns,
    peer_addr,
    peer_asn,
    prefix_addr,
    prefix_len,
    announced,
    as_path,
    next_hop,
    origin,
FROM bmp.updates
WHERE peer_asn = 65000
ORDER BY time_received_ns DESC
LIMIT 100 FORMAT CSVWithNames"
```

Replace `65000` with your assigned ASN.

### Check a Specific Prefix

To verify a specific prefix announcement (using `2a06:de00:5b::` as an example):

```sh
curl -X POST "https://nxthdr.dev/api/query/" \
  -u "read:read" \
  -H "Content-Type: text/plain" \
  -d "SELECT
    time_received_ns,
    peer_addr,
    peer_asn,
    prefix_addr,
    prefix_len,
    announced,
    as_path,
    next_hop,
    origin
FROM bmp.updates
WHERE peer_asn = 65000
  AND prefix_addr = toIPv6('2a06:de00:5b::')
  AND prefix_len = 48
ORDER BY time_received_ns DESC
LIMIT 100 FORMAT CSVWithNames"
```

Replace the prefix address `2a06:de00:5b::` with your leased prefix.

## Get Help

If you encounter issues or have questions, feel free to [contact us](/docs/reference/contact/).
The peering platform is actively evolving. We welcome feedback and suggestions for improving the user experience.
