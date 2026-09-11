# FlexiSign Mirroring Node firewall reference

Author: **FlexiSign / DualBytes — Adrian Bega**

Apply these rules on the cloud security group, router, and host firewall for a
public FlexiSign Mirroring Node.

## Required traffic

| Direction | Protocol / port | Purpose | Exposure |
| --- | --- | --- | --- |
| Inbound | TCP `443` | HTTPS and WSS through the TLS reverse proxy | Public |
| Inbound | TCP `7881` | LiveKit fallback connectivity | Public |
| Inbound | UDP `50000-60000` | Normal WebRTC media | Public |
| Outbound | TCP `443` | FlexiSign heartbeats, verification, and redacted events | FlexiSign console |

## Rules

- Use a stable public IP or DNS name.
- Allow bidirectional UDP for `50000-60000`; these ports must not pass through
  an HTTP reverse proxy.
- Terminate TLS on TCP `443` and proxy HTTPS/WSS to host loopback
  `127.0.0.1:8080`.
- Publish TCP `7881` directly when fallback connectivity is needed.
- Keep host TCP `8080`, `/metrics`, `/v1/control/revoke`, and
  `/v1/webhooks/livekit` private as described by the deployment contract.
- Do not expose LiveKit API credentials or the Docker socket.

## Example UFW rules

```sh
sudo ufw allow 443/tcp
sudo ufw allow 7881/tcp
sudo ufw allow 50000:60000/udp
sudo ufw reload
```

Restrict outbound TCP `443` to the FlexiSign console when the organisation’s
egress policy supports an allowlist. Apply equivalent restrictions in the
cloud provider security group.

## Verification checklist

- The public hostname has a valid certificate and resolves to the node.
- Public TCP `443` reaches the TLS proxy, not the adapter directly.
- Public UDP `50000-60000` reaches the node without proxying or port rewriting.
- TCP `7881` is reachable when fallback transport is required.
- TCP `8080` is reachable only from the local reverse proxy.
- The node reports **online** in FlexiSign after the firewall is applied.
