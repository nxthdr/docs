---
title: Peering Platform
weight: 2
prev: /docs/internals/infrastructure/
next: /docs/internals/probing
---

PeerLab is a research and educational platform that allows users to connect to a real Internet Exchange Point (IXP), receive the full IPv6 routing table via BGP, and announce IPv6 prefixes. The platform provides hands-on experience with BGP peering in a production environment.

## Architecture Overview

The PeerLab platform consists of three main components working together to provide secure, authenticated BGP peering:

```
User (Docker) → Headscale (VPN) → IXP Server (BIRD) → Internet
                      ↓
              PeerLab Gateway (ASN/Prefix Management)
                      ↓
              BIRD Config Generator
```

## Components

### PeerLab Client

The [PeerLab](https://github.com/nxthdr/peerlab) client is a Docker Compose stack that users run locally. It provides:

**Tailscale Container**: Connects to the Headscale VPN server, establishing a secure tunnel to the IXP infrastructure. Users authenticate via their nxthdr.dev account.

**Bootstrap Service**: Python service that automatically generates BIRD configuration by discovering IXP servers from the Tailscale network. It parses user-provided ASN and IPv6 prefixes from environment variables and renders Jinja2 templates.

**BIRD Container**: Runs the BIRD routing daemon with auto-generated configuration. Establishes BGP sessions with IXP servers over the Tailscale network using IPv4 addresses (100.64.0.0/10 CGNAT range) while exchanging IPv6 routes.

**Caddy Web Server**: Provides a simple web interface showing BGP session status and advertised prefixes.

### PeerLab Gateway

The [PeerLab Gateway](https://github.com/nxthdr/peerlab-gateway) is a Rust-based API service that manages user resources. It provides two API surfaces:

**Client API** (`/api/*`): JWT-authenticated endpoints for end users via nxthdr.dev
- `GET /api/user/info`: Retrieve user ASN and active prefix leases
- `POST /api/user/asn`: Request ASN assignment (automatic allocation from pool)
- `POST /api/user/prefix`: Request time-limited IPv6 /48 prefix leases

**Service API** (`/service/*`): Agent-authenticated endpoints for infrastructure services
- `GET /service/mappings`: Retrieve all user-to-ASN-to-prefix mappings with email addresses
- `GET /service/mappings/:user_hash`: Query specific user mappings

**Key Features**:
- ASN pool management (default: 65000-65999, providing 1000 ASNs)
- Time-based IPv6 /48 prefix leasing from configurable pool
- PostgreSQL storage for user mappings and lease information
- On-demand email retrieval from Auth0 Management API (no email storage)
- SHA256 user hash for privacy

### BIRD Config Generator

The [PeerLab BIRD Config Generator](https://github.com/nxthdr/peerlab-bird-config) is a Rust service that generates BIRD BGP policy enforcement configuration. It runs as a systemd timer on IXP servers.

**Data Sources**:
- Headscale API: Discovers connected nodes and their Tailscale IPs
- PeerLab Gateway API: Fetches user email-to-ASN-to-prefix mappings

**Generated Output**:
```bird
function enforce_user_policy(ip remote_ip) {
    # User: matthieu@nxthdr.dev
    if (remote_ip = 100.64.0.10) then {
        if (bgp_path.last != 65000) then reject;
        if !(net ~ [ 2a06:de00:5b:1000::/48 ]) then reject;
        accept;
    }

    # User: alice@example.com
    if (remote_ip = 100.64.0.11) then {
        if (bgp_path.last != 65001) then reject;
        if !(net ~ [ 2a06:de00:5b:2000::/48 ]) then reject;
        accept;
    }

    # Unknown user
    reject;
}
```

The function validates both ASN and prefix announcements for each user. The service runs every minute via systemd timer, updating the configuration file (`/etc/bird/peerlab_generated.conf`) only when changes are detected (SHA256 comparison). BIRD automatically reconfigures when the file changes.

## IXP Server Configuration

IXP servers (e.g., ixpfra01) run BIRD with special PeerLab configuration that enables dynamic BGP sessions.

### Dynamic BGP Protocol

BIRD uses a dynamic BGP protocol that accepts connections from the Tailscale CGNAT range (100.64.0.0/10):

```bird
protocol bgp peerlab from peerlab_template {
    dynamic name "peerlab_";
    dynamic name digits 5;
    neighbor range 100.64.0.0/10 external;
}
```

Each connecting user gets a dedicated BGP session with a unique protocol name (e.g., `peerlab_00001`).

### Policy Enforcement

The `enforce_user_asn_and_prefix` filter validates incoming BGP announcements:

- Verifies the peer's ASN matches their assigned ASN from the mapping
- Validates announced prefixes belong to the user's leased prefixes
- Rejects announcements that don't match the user's allocation

### Route Exchange

**Imports**: User announcements are validated and accepted into the routing table. PeerLab prefixes (2a06:de00:5b::/48, 2a06:de00:5c::/48) are exported to upstream peers and IXP route servers.

**Exports**: Users receive the full IPv6 routing table, including:
- nxthdr infrastructure prefix (2a06:de00:50::/48)
- All routes learned from IXP peers and upstream providers
- Routes from other PeerLab participants

### Extended Next Hop

BGP sessions use Extended Next Hop (RFC 5549), allowing IPv6 routes to be exchanged over IPv4 BGP sessions. This simplifies connectivity since Tailscale provides IPv4 addresses.

## User Workflow

1. **Registration**: User creates an account at nxthdr.dev
2. **ASN Request**: User requests an ASN via the web interface (calls Gateway API)
3. **Prefix Lease** (optional): User requests time-limited IPv6 /48 prefix for advertisement
4. **Client Setup**: User configures PeerLab Docker stack with ASN and prefixes
5. **Authentication**: User authenticates Tailscale with Headscale via nxthdr.dev account
6. **Bootstrap**: Bootstrap service discovers IXP servers and generates BIRD config
7. **BGP Session**: BIRD establishes BGP sessions with IXP servers
8. **Route Exchange**: User receives full IPv6 routing table and optionally advertises prefixes

## Prefix Advertisement

Users can advertise IPv6 prefixes leased from the Gateway. The client configuration supports:

**Receive-only mode**: No prefixes configured, user only receives routes
**Single prefix**: One /48 prefix advertised to all IXP peers
**Multiple prefixes**: Comma-separated list of /48 prefixes

Prefixes are configured as static blackhole routes in BIRD and advertised via the `ExportToIXP` filter. The IXP server validates that announced prefixes match the user's active leases.

## Security Model

**Authentication**: Multi-layer authentication ensures only authorized users can peer
- Headscale VPN requires nxthdr.dev account authentication
- Gateway API uses Auth0 JWT tokens for client requests
- Service API uses Bearer token authentication for infrastructure services

**Authorization**: BGP policy enforcement prevents unauthorized announcements
- ASN validation ensures peers use their assigned ASN
- Prefix validation ensures announcements match active leases
- RPKI validation on IXP servers filters invalid routes

**Network Isolation**: Tailscale provides encrypted tunnels between users and IXP servers, isolating PeerLab traffic from production infrastructure.
