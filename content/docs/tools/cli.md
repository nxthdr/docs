---
title: CLI
weight: 1
prev: /docs/tools/
---

The **nxthdr** CLI is a command line tool to interact with the nxthdr platform. It supports authentication, peering, and probing operations.

Source code: [github.com/nxthdr/cli](https://github.com/nxthdr/cli)

## Installation

Install from [crates.io](https://crates.io/crates/nxthdr) using Cargo:

```bash
cargo install nxthdr
```

## Authentication

All commands that interact with the platform require authentication. The CLI uses the Auth0 device authorization flow.

### Login

```bash
nxthdr login
```

This will display a URL and a code. Open the URL in your browser and enter the code to authenticate. Tokens are stored locally and refreshed automatically when expired.

### Logout

```bash
nxthdr logout
```

Removes stored tokens from your system.

### Status

```bash
nxthdr status
```

Displays your current authentication state and token expiration time.

## Probing

Commands to interact with the probing platform (Saimiris).

### Credits

```bash
nxthdr probing credits
```

Displays your daily probing credits usage. Each probe you send consumes one credit. The daily limit resets at midnight UTC.

Example output:

```
credits
───────
used       0
limit      1000000
remaining  1000000
```

### Agents

```bash
nxthdr probing agents
```

Lists available probing agents and their source prefixes.

Example output:

```
id        status   prefixes
─────────────────────────────────────────────────────────────────────────────
vltewr01  healthy  2a0e:97c0:8a0::/48 (anycast), 2a0e:97c0:8a3::/48 (unicast)
vltsgp01  healthy  2a0e:97c0:8a0::/48 (anycast), 2a0e:97c0:8a5::/48 (unicast)
vltcdg01  healthy  2a0e:97c0:8a0::/48 (anycast), 2a0e:97c0:8a4::/48 (unicast)
```

### Send

```bash
nxthdr probing send --agent <agent-id> [file]
```

Sends probes from the given agent. Each probe is one line in the format `dst_addr,src_port,dst_port,ttl,protocol` (protocol is `icmpv6` or `udp`). Reads from a file or stdin if no file argument is given. Use `--agent` multiple times for multi-agent sends.

```bash
echo '2001:4860:4860::8888,24000,33434,16,udp' | nxthdr probing send --agent vltcdg01
```

Example output:

```
✓ measurement submitted
id        d80d7ecf-d60b-42af-bd22-2f9937e44a7f
probes    1 × 1 agent
vltcdg01  2a0e:97c0:8a0:1865:ceb8:372d:d58:fe47

→ nxthdr probing measurement-status d80d7ecf-d60b-42af-bd22-2f9937e44a7f
→ nxthdr probing results --src-ip 2a0e:97c0:8a0:1865:ceb8:372d:d58:fe47
```

The output includes the measurement ID and the source IP used by each agent — both needed to retrieve results.

### Measurement status

```bash
nxthdr probing measurement-status <id>
```

Shows the progress of a measurement by ID.

Example output:

```
measurement
───────────
id      d80d7ecf-d60b-42af-bd22-2f9937e44a7f
status  complete
agents  1/1 complete
probes  1/1 sent
agent     probes sent/expected  complete
────────────────────────────────────────
vltcdg01  1/1                   yes
```

### Results

```bash
nxthdr probing results --src-ip <ip> [--since <timestamp>] [--until <timestamp>]
```

Queries probe replies from ClickHouse. `--src-ip` is the source IP returned by `send`.

```bash
nxthdr probing results \
  --src-ip 2a0e:97c0:8a0:1865:ceb8:372d:d58:fe47 \
  --since "2026-06-16 00:00:00"
```

Example output:

```
agent     src                                    dst                   ttl  reply                 rtt
────────────────────────────────────────────────────────────────────────────────────────────────────────
vltcdg01  2a0e:97c0:8a0:1865:ceb8:372d:d58:fe47  2001:4860:4860::8888  16   2001:4860:4860::8888  0.00ms
```

Use `--output json` for machine-readable output suitable for piping or scripting:

```bash
nxthdr probing results --src-ip <ip> --since "..." --output json
```

## Peering

Commands to interact with the peering platform (PeerLab).

### Get your ASN

```bash
nxthdr peering asn
```

Displays your assigned private ASN. An ASN is automatically assigned on first use.

### Prefix management

List your active prefix leases:

```bash
nxthdr peering prefix list
```

Request a new /48 IPv6 prefix lease (duration in hours, 1 to 24):

```bash
nxthdr peering prefix request 12
```

Revoke an existing prefix lease:

```bash
nxthdr peering prefix revoke 2a06:de00:5b::/48
```

### RPKI ROA management

When you lease a prefix, an RPKI ROA (Route Origin Authorization) is automatically created. You can toggle it on or off for each leased prefix.

Enable RPKI for a prefix:

```bash
nxthdr peering prefix rpki enable 2a06:de00:5b::/48
```

Disable RPKI for a prefix:

```bash
nxthdr peering prefix rpki disable 2a06:de00:5b::/48
```

The `prefix list` command also shows the current RPKI status for each lease.

### PeerLab environment

Generate a `.env` file for PeerLab with your ASN and active prefixes:

```bash
nxthdr peering peerlab env > .env
```

## Verbosity

All commands support a `--verbose` flag (or `-v`) to increase log output. Use `-vv` for debug level.

```bash
nxthdr -v probing credits
```

## Environment Variables

| Variable | Description | Default |
|---|---|---|
| `NXTHDR_API_URL` | PeerLab API base URL | `https://peerlab.nxthdr.dev` |
| `NXTHDR_SAIMIRIS_API_URL` | Saimiris Gateway base URL | `https://saimiris.nxthdr.dev` |
| `NXTHDR_CLIENT_ID` | Auth0 client ID | Built in default |
