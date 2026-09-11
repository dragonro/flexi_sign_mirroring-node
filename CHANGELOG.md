# Changelog

Author: **FlexiSign / DualBytes — Adrian Bega**

## 1.0.933 — stable

- First stable public FlexiSign Mirroring Node release.
- Pinned LiveKit Server `v1.13.6` base image.
- UDP-first WebRTC media on ports `50000-60000`.
- HTTPS/WSS adapter and TCP `7881` fallback transport.
- Health reporting, signed event intake, redacted telemetry, and protected
  participant-revocation control.
- Immutable image deployment through a Cosign-verified GHCR digest.
- Downloadable Linux/amd64 Docker image archive with SHA-256 checksum.
