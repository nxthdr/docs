---
title: CLI
weight: 1
prev: /docs/tools/
---

The **nxthdr** CLI is a command line tool to interact with the nxthdr platform. It supports authentication, peering operations, and probing credits management.

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
Probing credits (today):
  Used:  1500
  Limit: 10000
  Remaining: 8500
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

{{< callout type="info" >}}
RPKI must be re-enabled before revoking a prefix. The platform enforces this for routing stability.
{{< /callout >}}

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
