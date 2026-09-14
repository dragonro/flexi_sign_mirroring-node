# Changelog

Author: **FlexiSign / DualBytes — Adrian Bega**

## 1.0.935 — stable

- Updated the public node image reference to the production `1.0.935`
  release on GHCR.
- Added protected performance telemetry for CPU, memory, swap, uptime, load,
  and inbound, outbound, and total network rates.
- Uses the Cloudflare Tunnel deployment profile with the node adapter private
  on `127.0.0.1:8080` and no inbound web port requirement.
- Expanded the direct WebRTC media range to UDP `52000-62000`.
- Retained LiveKit Server `v1.13.6`, protocol `1`, and the initial admission
  limits of 10 sessions, 40 receivers, and 50 participants.

## 1.0.933 — stable

- First stable public FlexiSign Mirroring Node release.
- Pinned LiveKit Server `v1.13.6` base image.
- UDP-first WebRTC media on ports `50000-60000`.
- HTTPS/WSS adapter and TCP `7881` fallback transport.
- Health reporting, signed event intake, redacted telemetry, and protected
  participant-revocation control.
- Immutable image deployment through a Cosign-verified GHCR digest.
- Downloadable Linux/amd64 Docker image archive with SHA-256 checksum.
