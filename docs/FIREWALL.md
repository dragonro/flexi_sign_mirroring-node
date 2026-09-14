# FlexiSign Mirroring Node firewall reference

Author: **FlexiSign / DualBytes — Adrian Bega**

Apply these rules on the cloud security group, router, and host firewall for a
customer-operated FlexiSign Mirroring Node using Cloudflare Tunnel.

## Required traffic

| Direction | Protocol / port | Purpose | Exposure |
| --- | --- | --- | --- |
| Inbound | TCP `7881` | LiveKit fallback connectivity | Public |
| Inbound | UDP `52000-62000` | Normal WebRTC media | Public |
| Outbound | TCP `443` | FlexiSign and Cloudflare HTTPS/control traffic | Required |
| Outbound | TCP/UDP `7844` | Cloudflare Tunnel HTTP/2 fallback and QUIC transport | Required |

The Cloudflare Tunnel publishes HTTPS/WSS through an outbound-only connection.
There is no inbound web-port requirement for TCP `80`, `443`, or `8443`.

## Rules

- Use a stable public IP for direct media and fallback traffic.
- Allow bidirectional UDP for `52000-62000`; these ports must not pass through
  an HTTP reverse proxy or be port-translated.
- Publish TCP `7881` directly when fallback connectivity is needed.
- Allow `cloudflared` outbound TCP and UDP `7844`; QUIC uses UDP and HTTP/2
  fallback uses TCP. Allow outbound TCP `443` for HTTPS and Cloudflare
  management traffic.
- Route the registered public hostname in Cloudflare to
  `http://127.0.0.1:8080` inside the node host. The hostname must match
  `FLEXISIGN_NODE_PUBLIC_HOST` exactly.
- Keep adapter TCP `8080`, LiveKit HTTP `7880`, `/v1/webhooks/livekit`, and
  the Docker socket private.
- `/metrics` and `/v1/control/revoke` remain protected by the node API token;
  do not publish unauthenticated monitoring or control endpoints.
- Do not expose LiveKit API credentials, node API tokens, or Cloudflare Tunnel
  tokens.

## Example UFW rules

```sh
sudo ufw allow 7881/tcp
sudo ufw allow 52000:62000/udp
sudo ufw allow out 443/tcp
sudo ufw allow out 7844/tcp
sudo ufw allow out 7844/udp
sudo ufw reload
```

Restrict outbound TCP `443` and TCP/UDP `7844` to the required FlexiSign and
Cloudflare destinations when the organisation's egress policy supports an
allowlist. Apply equivalent restrictions in the cloud provider security
group. Cloudflare's tunnel endpoints can change; use its current firewall
reference when an IP allowlist is required.

## Verification checklist

- The registered public hostname is configured as a Cloudflare Tunnel
  published application to `http://127.0.0.1:8080`.
- The node host can reach Cloudflare on outbound TCP/UDP `7844` and TCP `443`.
- Public UDP `52000-62000` reaches the node without proxying or port rewriting.
- TCP `7881` is reachable when fallback transport is required.
- TCP `8080` and LiveKit HTTP `7880` are not reachable from the Internet.
- `https://<registered-hostname>/health` returns status `ok`.
- The node reports **online** in FlexiSign after the firewall is applied.
