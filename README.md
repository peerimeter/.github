<p align="center">
  <img src="https://raw.githubusercontent.com/peerimeter/.github/HEAD/assets/peerimeter-logo.png" alt="Peerimeter logo" width="600">
</p>

# Peerimeter

Peerimeter is a self-hosted WireGuard mesh agent for direct peer-to-peer connections. Its offline cross-platform agent and inventory-aware self-update are implemented; the control plane is planned for a later milestone.

The project is organized around two independent repositories:

- **peerimeter-agent** — a headless Go agent and background service targeting Linux, macOS, and Windows. The first milestone is offline WireGuard connectivity using local static JSON configuration.
- **peerimeter-service** — a Go control plane with a REST API, PostgreSQL, MQTT coordination, and an embedded React dashboard with GitHub sign-in, planned after the offline agent milestone.

Later milestones cover STUN-assisted endpoint discovery, connection recovery, and gateways for standard WireGuard clients. Connections are direct only: there is no relay fallback, so restrictive NAT or firewalls may prevent connectivity.

**Status:** Phase 1 offline agent and Phase 2 self-update complete, with native Linux, macOS and Windows verification. The standalone STUN probe still needs native verification; Phase 3 service/control-plane work has not started.

**Website:** [peerimeter.app](https://peerimeter.app) · [Agent installer](https://agent.peerimeter.app) (release packages are available, currently at v0.17.0).

See [the implementation plan](https://github.com/peerimeter/.github/blob/main/docs/planning/plan.md), [additional requirements](https://github.com/peerimeter/.github/blob/main/docs/planning/additional-plan.md), and [progress tracker](https://github.com/peerimeter/.github/blob/main/docs/planning/progress.md).

## GitHub pull request settings

Checked through the GitHub REST API on **2026-11-14**, after the branch-protection
settings were updated. Both repositories allow auto-merge and rebase-only merges,
require linear history, require up-to-date branches and resolved review
conversations, and delete branches after merge. Neither requires an approving
review, and admin enforcement is off.

| Repository | Required status checks on `main` | Require up-to-date branch | Require all review conversations resolved |
| --- | --- | --- | --- |
| [`peerimeter/.github`](https://github.com/peerimeter/.github) | `CodeRabbit` | Yes (`strict: true`) | Yes |
| [`peerimeter/peerimeter-agent`](https://github.com/peerimeter/peerimeter-agent) | `check`, `CodeRabbit` | Yes (`strict: true`) | Yes |

The settings now match on branch freshness and conversation resolution. The
agent repository additionally requires the pipeline's aggregate `check`. Keep
inspecting, addressing and resolving review conversations before auto-merge;
GitHub now enforces both conversation resolution and branch freshness. Refresh
this summary when repository rules change.
