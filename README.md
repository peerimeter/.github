<p align="center">
  <img src="https://raw.githubusercontent.com/peerimeter/.github/HEAD/assets/peerimeter-logo.png" alt="Peerimeter logo" width="600">
</p>

# Peerimeter

Peerimeter is a planned, self-hosted WireGuard mesh VPN with direct peer-to-peer connections and centralized network management.

The project is organized around two independent repositories:

- **peerimeter-agent** — a headless Go agent and background service targeting Linux, macOS, and Windows. The first milestone is offline WireGuard connectivity using local static JSON configuration.
- **peerimeter-service** — a Go control plane with a REST API, PostgreSQL, MQTT coordination, and an embedded React dashboard with GitHub sign-in, planned after the offline agent milestone.

Later milestones cover STUN-assisted endpoint discovery, connection recovery, and gateways for standard WireGuard clients. Connections are direct only: there is no relay fallback, so restrictive NAT or firewalls may prevent connectivity.

**Status:** Planning / pre-implementation. No implementation milestone or platform support has been verified yet.

See [the implementation plan](https://github.com/peerimeter/.github/blob/main/docs/planning/plan.md), [additional requirements](https://github.com/peerimeter/.github/blob/main/docs/planning/additional-plan.md), and [progress tracker](https://github.com/peerimeter/.github/blob/main/docs/planning/progress.md).
