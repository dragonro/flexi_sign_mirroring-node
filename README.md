# FlexiSign Mirroring Node

Public release documentation and deployment assets for the FlexiSign
Mirroring Node. The node provides secure, UDP-first internet screen mirroring
through a customer-operated Docker deployment.

Author: **FlexiSign / DualBytes — Adrian Bega**

## Current stable release

- Version: `1.0.935` (2026-09-14)
- Protocol: `1`
- Supported architecture: `linux/amd64`
- Base media server: LiveKit Server `v1.13.6`
- GHCR image: [`ghcr.io/dragonro/flexi_sign_mirroring-node`](https://github.com/dragonro/flexi_sign_mirroring-node/pkgs/container/flexi_sign_mirroring-node)
- Immutable image reference:
  `ghcr.io/dragonro/flexi_sign_mirroring-node@sha256:25c95c1105882372b68d19b2f29a9d35b49455e224e0211d95fb6576337307c8`
- Release history and downloadable assets: [GitHub Releases](https://github.com/dragonro/flexi_sign_mirroring-node/releases)

Version `1.0.935` adds protected node performance telemetry to the console
heartbeat and metrics endpoint: CPU, memory, swap, uptime, load, and inbound,
outbound, and total network rates. The console exposes these values in the
node's **Performance** tab.

This public repository contains release documentation and deployment assets
only. It does not contain the Mirroring Node application source code.

## Quick installation

1. In FlexiSign, open **Manage Mirroring Server**, create a node, and copy the
   one-time installation values.
2. Download the [Docker Compose template](docker-compose.yml) and use the
   current version of this repository's deployment assets.
3. Set `FLEXISIGN_MIRRORING_NODE_IMAGE` to the exact Cosign-verified immutable
   image reference shown above. Do not deploy a mutable tag in production.
4. Fill in the one-time node ID, backend URL, public hostname, node API token,
   LiveKit API key and secret. Provide the Cloudflare Tunnel token as the
   `cloudflared` Docker secret.
5. In Cloudflare, publish the registered hostname through the tunnel to
   `http://127.0.0.1:8080`. The hostname must exactly match
   `FLEXISIGN_NODE_PUBLIC_HOST`.
6. Apply the [firewall reference](docs/FIREWALL.md). Directly expose only TCP
   `7881` and UDP `52000-62000`; no inbound web port `80`, `443`, or `8443` is
   required for the tunnel-based deployment.
7. Start the node and tunnel together:

   ```sh
   export FLEXISIGN_MIRRORING_NODE_IMAGE='ghcr.io/dragonro/flexi_sign_mirroring-node@sha256:25c95c1105882372b68d19b2f29a9d35b49455e224e0211d95fb6576337307c8'
   docker pull "$FLEXISIGN_MIRRORING_NODE_IMAGE"
   docker compose up -d
   ```

8. Verify `https://<registered-hostname>/health` returns status `ok`, then
   return to FlexiSign and verify the node. It becomes selectable for screen
   mirroring after verification and a healthy heartbeat.

For a downloadable archive or rollback image, use a release asset matching
the required version from [GitHub Releases](https://github.com/dragonro/flexi_sign_mirroring-node/releases).
Do not combine an older archive with the current `1.0.935` Compose settings.
The immutable GHCR image above is the preferred production artifact.

## Deployment contract

- Public HTTPS/WSS control traffic: provided by Cloudflare Tunnel to the
  private adapter at `127.0.0.1:8080`.
- LiveKit fallback transport: TCP `7881` directly to the node.
- Normal WebRTC media: UDP `52000-62000` directly to the node.
- Node-to-console heartbeats and redacted events: outbound TCP `443`.
- Cloudflare Tunnel transport: outbound TCP/UDP `7844` as allowed by the
  selected `cloudflared` protocol, plus outbound TCP `443` for Cloudflare
  management and HTTPS.
- Never expose adapter TCP `8080`, LiveKit HTTP `7880`, metrics without the
  node API token, LiveKit credentials, webhook intake, or the Docker socket.

The initial admission profile is 10 concurrent sessions, 40 receivers, 50
participants, 4 targets per session, 6 Mbps maximum video bitrate, and
1080p30. Limits can be adjusted later in FlexiSign without replacing the
container image.

## Performance telemetry

The protected `/metrics` endpoint and node heartbeat report the latest
performance snapshot under `usage.performance`:

- CPU utilization and one-minute load;
- memory used, total, percentage, and swap;
- node uptime and sampling interval;
- inbound, outbound, and total network bits per second.

The first sample after startup does not contain rate-based CPU or network
values because a previous counter sample is required. Network rates cover
non-loopback host interfaces and may include traffic from other host
processes. The console marks telemetry stale when the node has not reported a
recent snapshot.

## Operations

- Use **Live Stats** and **Performance** in FlexiSign to inspect sessions,
  receivers, health, capacity, traffic, and resource consumption.
- Keep `/health` available through the registered tunnel hostname for bounded
  operational checks.
- Rotate node credentials after suspected exposure, then redeploy before
  restoring the node to service.
- Stop active mirroring sessions before retiring a node.
- Keep the existing direct/local route available as the rollback path.

See the [firewall reference](docs/FIREWALL.md) for network rules. The
FlexiSign console contains the current internal deployment documentation and
the verified image digest for each release.
