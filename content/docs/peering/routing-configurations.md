---
title: Routing Configurations
weight: 2
prev: /docs/peering/getting-started/
next: /docs/peering/bgp-data-feedback/
---

Once your PeerLab environment is running and BGP sessions are established, you can customize your BIRD routing configuration to perform advanced experiments. This guide covers common routing scenarios.

## How Configuration Works

PeerLab generates a default BIRD configuration in `workspace/bird.conf` based on your `.env` settings. You can edit this file directly and reload BIRD without restarting the stack:

```bash
make bird-config
```

The default configuration accepts all routes from IXP peers and advertises only your leased prefixes. The sections below show how to customize this behavior.

## Receive-Only Mode

If you only want to observe the full IPv6 routing table without advertising anything, leave `USER_PREFIXES` empty in your `.env` file:

```bash
USER_PREFIXES=
```

In this mode, BIRD will establish BGP sessions and import the full routing table from all IXP peers, but will not export any routes.

You can inspect the received routes:

```bash
# Show all received routes
make bird-routes

# Count total routes in the routing table
make bird-routes-count
```

## Selective Route Import

By default, PeerLab imports all routes from IXP peers. You can modify the `ImportFromIXP` filter in `workspace/bird.conf` to accept only specific routes.

### Filter by Prefix Length

Accept only prefixes within a specific length range:

```bird
filter ImportFromIXP {
    # Accept only prefixes between /32 and /48
    if net.len < 32 then reject;
    if net.len > 48 then reject;
    accept;
}
```

### Filter by AS Path

Accept only routes originating from specific ASNs:

```bird
filter ImportFromIXP {
    # Accept routes originating from AS13335 (Cloudflare)
    if bgp_path.last = 13335 then accept;

    # Accept routes originating from AS15169 (Google)
    if bgp_path.last = 15169 then accept;

    reject;
}
```

### Default Route Only

If you want to receive only a default route instead of the full table, you can add a static default route pointing to the IXP servers. Replace the import filter to drop all received routes, and add a default route:

```bird
filter ImportFromIXP {
    reject;
}

protocol static default_route {
    ipv6;
    route ::/0 via <IXP_SERVER_IP>;
}
```

Replace `<IXP_SERVER_IP>` with the Tailscale IP of one of your IXP peers (visible in the generated `bird.conf`).

> **Note**: Since PeerLab connects over a Tailscale overlay, receiving the full table does not affect your local routing unless you explicitly install routes into your kernel. The default configuration does install routes via the kernel protocol, so adjust accordingly if you only want to observe.

## Prefix Advertisement

### Announcing Leased Prefixes

Configure your leased prefixes in `.env`:

```bash
USER_PREFIXES=2a06:de00:5b:1000::/48
```

For multiple prefixes:

```bash
USER_PREFIXES=2a06:de00:5b:1000::/48,2a06:de00:5b:2000::/48
```

The bootstrap service creates static blackhole routes for each prefix and the `ExportToIXP` filter advertises them to all IXP peers.

### AS Path Prepending

To influence inbound traffic by making your route appear longer to specific peers, modify the `ExportToIXP` filter:

```bird
filter ExportToIXP {
    if proto = "user_prefixes" then {
        # Prepend your ASN twice to make this path less preferred
        bgp_path.prepend(YOUR_ASN);
        bgp_path.prepend(YOUR_ASN);
        accept;
    }
    reject;
}
```

Replace `YOUR_ASN` with your assigned private ASN (e.g., `65000`).

You can also apply prepending selectively per IXP peer by using different export filters for different BGP protocol blocks.

### Selective Export per IXP

To announce different prefixes to different IXP peers, create separate export filters and assign them to individual BGP protocol blocks in `workspace/bird.conf`:

```bird
filter ExportToFRA {
    if proto = "user_prefixes" then {
        # Only announce the first prefix to Frankfurt
        if net = 2a06:de00:5b:1000::/48 then accept;
    }
    reject;
}

filter ExportToAMS {
    if proto = "user_prefixes" then {
        # Announce all prefixes to Amsterdam
        accept;
    }
    reject;
}
```

Then update the corresponding BGP protocol blocks to use these filters.

## Working with BGP Communities

BGP communities are used by networks to tag routes with metadata. You can use communities in your import and export filters.

### Filtering on Received Communities

Filter incoming routes based on communities set by upstream peers:

```bird
filter ImportFromIXP {
    # Accept routes tagged with a specific community
    if (65000, 100) ~ bgp_community then accept;

    # Accept routes with a specific large community
    if (215011, 100, 200) ~ bgp_large_community then accept;

    reject;
}
```

### Tagging Exported Routes

Add communities to your advertised routes:

```bird
filter ExportToIXP {
    if proto = "user_prefixes" then {
        # Add a standard community
        bgp_community.add((YOUR_ASN, 100));

        # Add a large community
        bgp_large_community.add((YOUR_ASN, 1, 0));

        accept;
    }
    reject;
}
```

You can then verify that your communities are propagated by querying the [peering dataset](/docs/datasets/):

```sh
curl -X POST "https://nxthdr.dev/api/query/" \
  -u "read:read" \
  -H "Content-Type: text/plain" \
  -d "SELECT
    time_received_ns,
    prefix_addr,
    prefix_len,
    communities,
    large_communities
FROM bmp.updates
WHERE peer_asn = YOUR_ASN
ORDER BY time_received_ns DESC
LIMIT 10 FORMAT CSVWithNames"
```

## Useful BIRD Commands

PeerLab provides Makefile shortcuts for common BIRD operations:

| Command | Description |
|---------|-------------|
| `make bird-status` | Show BIRD daemon status |
| `make bird-protocols` | Show BGP session status |
| `make bird-routes` | Show all received routes |
| `make bird-routes-count` | Count total routes |
| `make bird-exports` | Show exported routes per peer |
| `make bird-prefixes` | Show configured user prefixes |
| `make bird-config` | Reload BIRD configuration |

For more advanced queries, you can exec into the BIRD container:

```bash
docker exec -it peerlab-bird birdc
```

Then use BIRD's interactive CLI:

```bird
birdc> show route where bgp_path.last = 13335
birdc> show route filter { if net.len > 48 then reject; accept; }
birdc> show protocols all ixpfra01
```

## Important Notes

- **Only announce leased prefixes.** The IXP servers enforce policies that reject announcements not matching your active leases. See [Security & Compliance](/docs/peering/security-compliance/).
- **Use private ASNs.** Your assigned ASN from the pool (65000-65999) is a private ASN. It will be visible in the AS path within the nxthdr network but stripped or replaced at network boundaries.
- **Configuration resets on restart.** If you run `make up` again, the bootstrap service regenerates `workspace/bird.conf` from the template, overwriting your changes. Back up your customizations before restarting.
