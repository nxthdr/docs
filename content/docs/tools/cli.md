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
nxthdr auth login
```

This will display a URL and a code. Open the URL in your browser and enter the code to authenticate. Tokens are stored locally and refreshed automatically when expired.

### Logout

```bash
nxthdr auth logout
```

Removes stored tokens from your system.

### Status

```bash
nxthdr auth status
```

Displays your current authentication state and token expiration time.

## Probing

Commands to interact with the probing platform (Saimiris).

### Agents

```bash
nxthdr probing agent list
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

### Credits

```bash
nxthdr probing credits get
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

### Measurements

Measurement operations are grouped under `measurement`.

#### Send

```bash
nxthdr probing measurement send --agent <agent-id> [file]
```

Sends probes from the given agent. Each probe is one line in the format `dst_addr,src_port,dst_port,ttl,protocol` (protocol is `icmpv6` or `udp`). Reads from a file or stdin if no file argument is given. Use `--agent` multiple times for multi-agent sends.

```bash
echo '2001:4860:4860::8888,24000,33434,16,udp' | nxthdr probing measurement send --agent vltcdg01
```

Example output:

```
✓ measurement submitted
id        d80d7ecf-d60b-42af-bd22-2f9937e44a7f
probes    1 × 1 agent
vltcdg01  2a0e:97c0:8a0:1865:ceb8:372d:d58:fe47

→ nxthdr probing measurement get d80d7ecf-d60b-42af-bd22-2f9937e44a7f
→ nxthdr probing reply list --src-ip 2a0e:97c0:8a0:1865:ceb8:372d:d58:fe47
```

The output includes the measurement ID and the source IP used by each agent — both needed to retrieve replies.

#### List

```bash
nxthdr probing measurement list [--limit <n>] [--status <s>] [--since <t>] [--until <t>] [--agent <id>] [--sort started|updated] [--reverse]
```

Lists your recent measurements, so the ID from `send` isn't the only handle. All filters are applied server-side, before the limit.

Example output:

```
id                                    started                      agents  probes     status
─────────────────────────────────────────────────────────────────────────────────────────────────
d80d7ecf-d60b-42af-bd22-2f9937e44a7f  2026-06-16T17:58:52.571824Z  1/1     1/1        complete
cf78cc3d-b42e-4066-865d-ef08656c6964  2026-03-22T10:59:12.022398Z  0/1     0/3000     cancelled
```

All filters are optional:

| Flag | Description |
|---|---|
| `--limit <n>` | Maximum number to return (1–100, default 20) |
| `--status <s>` | Comma-separated: `complete`, `in-progress`, `cancelled` |
| `--since` / `--until` | Start-time window (e.g. `2026-03-22` or `2026-03-22 10:00:00`) |
| `--agent <id>` | Only measurements involving that agent |
| `--sort started\|updated` | Sort field (default `updated`) |
| `--reverse` | Oldest first |

```bash
# cancelled measurements only, oldest first
nxthdr probing measurement list --status cancelled --sort started --reverse
```

#### Get

```bash
nxthdr probing measurement get <id>
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
agent     probes sent/expected  status
────────────────────────────────────────
vltcdg01  1/1                   yes
```

A cancelled measurement shows `cancelled` as its overall status and for each cancelled agent.

#### Cancel

```bash
nxthdr probing measurement cancel <id>
```

Cancels a stuck or in-progress measurement — useful when an agent dies mid-run, which would otherwise leave the measurement showing "in progress" indefinitely. The unfinished agents are marked cancelled, and the measurement then appears as `cancelled` in `measurement list` and `measurement get`. The command is idempotent.

Example output:

```
✓ Measurement cancelled
id                cf78cc3d-b42e-4066-865d-ef08656c6964
cancelled         true
agents_cancelled  1
message           Measurement cancelled
```

### Replies

```bash
nxthdr probing reply list --src-ip <ip> [--since <timestamp>] [--until <timestamp>]
```

Queries probe replies from ClickHouse. `--src-ip` is the source IP returned by `measurement send`.

```bash
nxthdr probing reply list \
  --src-ip 2a0e:97c0:8a0:1865:ceb8:372d:d58:fe47 \
  --since "2026-06-16 00:00:00"
```

Example output:

```
agent     src                                    dst                   ttl  reply                 rtt
────────────────────────────────────────────────────────────────────────────────────────────────────────
vltcdg01  2a0e:97c0:8a0:1865:ceb8:372d:d58:fe47  2001:4860:4860::8888  16   2001:4860:4860::8888  0.00ms
```

Use `--output json` or `--output csv` for machine-readable output suitable for piping or scripting (see [Output format](#output-format)):

```bash
nxthdr probing reply list --src-ip <ip> --since "..." --output json
```

## Peering

Commands to interact with the peering platform (PeerLab).

### Get your ASN

```bash
nxthdr peering asn get
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

### Routes (BGP visibility)

Check how prefixes are seen by public BGP collectors (RIPE RIS). List the visibility of your own leased prefixes:

```bash
nxthdr peering route list
```

Or look up any prefix, looking-glass style:

```bash
nxthdr peering route lookup 2a06:de00:5b::/48
```

### PeerLab environment

Generate a `.env` file for PeerLab with your ASN and active prefixes:

```bash
nxthdr peering peerlab env > .env
```

## Output format

Every command supports a global `-o` / `--output` flag with three formats:

- `text` (default) — human-readable tables and key/value output
- `json` — a JSON object or array, for piping and scripting
- `csv` — RFC-4180 CSV (header row + rows) for tabular commands

```bash
nxthdr probing measurement list --status cancelled -o csv
nxthdr probing reply list --src-ip <ip> -o json
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
