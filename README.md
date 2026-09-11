# FlexiSign Mirroring Node

Public release documentation and deployment assets for the FlexiSign Mirroring
Node. The node provides secure, UDP-first internet screen mirroring through a
customer-operated Docker deployment.

## Stable release

- Version: `1.0.933`
- Protocol: `1`
- Supported release architecture: `linux/amd64`
- Container image: `ghcr.io/dragonro/flexy_sign/mirroring-node`
- Base media server: LiveKit Server `v1.13.6`

This public repository contains release documentation and deployment assets
only. It does not contain the Mirroring Node application source code.

## Quick installation

1. In FlexiSign, open **Manage Mirroring Server**, create a node, and copy the
   one-time installation values.
2. Download `docker-compose.yml` from this release.
3. Set `FLEXISIGN_MIRRORING_NODE_IMAGE` to the exact Cosign-verified image
   digest supplied by FlexiSign. Do not deploy a mutable tag.
4. Fill the one-time values in the Compose environment section.
5. Configure TLS on public TCP 443 and apply the [firewall reference](docs/FIREWALL.md).
6. Start the node:

   ```sh
   export FLEXISIGN_MIRRORING_NODE_IMAGE='ghcr.io/dragonro/flexy_sign/mirroring-node@sha256:<verified-digest>'
   docker pull "$FLEXISIGN_MIRRORING_NODE_IMAGE"
   docker compose up -d
   ```

7. Return to FlexiSign and verify the node. It becomes selectable for screen
   mirroring only after verification and a healthy heartbeat.

## Deployment contract

- Public HTTPS/WSS: TCP `443` through a TLS reverse proxy to host loopback
  `127.0.0.1:8080`.
- LiveKit fallback transport: TCP `7881` directly to the node.
- Normal WebRTC media: UDP `50000-60000` directly to the node.
- Node-to-console heartbeats and redacted events: outbound TCP `443`.
- Never expose adapter TCP `8080`, metrics, LiveKit credentials, or the Docker
  socket publicly.

The initial admission profile is 10 concurrent sessions, 40 receivers, 50
participants, 4 targets per session, 6 Mbps maximum video bitrate, and
1080p30. Limits can be adjusted later in FlexiSign without replacing the
container image.

## Operations

- Use the live usage view in FlexiSign to inspect sessions, receivers, health,
  capacity, and redacted technical events.
- Rotate node credentials after suspected exposure, then redeploy before
  restoring the node to service.
- Stop active mirroring sessions before retiring a node.
- Keep the existing direct/local route available as the rollback path.

See the [firewall reference](docs/FIREWALL.md) for network rules. The
FlexiSign console contains the current internal deployment documentation and
the verified image digest for each release.
