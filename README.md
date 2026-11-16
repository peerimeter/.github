<p align="center">
  <img src="https://raw.githubusercontent.com/peerimeter/.github/HEAD/assets/peerimeter-logo.png" alt="Peerimeter logo" width="600">
</p>

# Peerimeter

Peerimeter is a self-hosted WireGuard mesh agent for direct peer-to-peer connections. The agent can use local configuration independently, while an optional control-plane service provides a shared API for network and node configuration.

The project is organized around two independent repositories:

- **peerimeter-agent** — a headless Go agent and background service targeting Linux, macOS, and Windows. It runs independently using local static YAML configuration, with optional one-shot configuration pull and push commands for the service API.
- **peerimeter-service** — a Go HTTP API that stores network and node configuration in a local JSON file. Writes are queued in memory and committed atomically by one service process. See the [service setup guide](https://service.peerimeter.app/) for a simple setup and Docker deployment.

Later milestones cover STUN-assisted endpoint discovery, connection recovery, and gateways for standard WireGuard clients. Connections are direct only: there is no relay fallback, so restrictive NAT or firewalls may prevent connectivity.

**Status:** The agent, self-update, and initial service API are implemented and verified on Linux, macOS, and Windows where applicable. The service currently uses one local JSON file and a shared static API key; it does not include PostgreSQL, MQTT, user accounts, or multi-instance coordination. Agent service commands pull or push configuration on request; automatic runtime reconciliation is not included. The standalone STUN probe still needs native verification.

**Website:** [peerimeter.app](https://peerimeter.app) · [Agent installer](https://agent.peerimeter.app) · [Service setup](https://service.peerimeter.app).

See [the implementation plan](https://github.com/peerimeter/.github/blob/main/docs/planning/plan.md), [additional requirements](https://github.com/peerimeter/.github/blob/main/docs/planning/additional-plan.md), and [progress tracker](https://github.com/peerimeter/.github/blob/main/docs/planning/progress.md).

## GitHub pull request settings

Checked through the GitHub REST API on **2026-11-16**, after the branch-protection
settings were updated. The two code repositories allow auto-merge and rebase-only merges,
require linear history, require up-to-date branches and resolved review
conversations, and delete branches after merge. Neither requires an approving
review, and admin enforcement is off.

| Repository | Required status checks on `main` | Require up-to-date branch | Require all review conversations resolved |
| --- | --- | --- | --- |
| [`peerimeter/.github`](https://github.com/peerimeter/.github) | `CodeRabbit` | Yes (`strict: true`) | Yes |
| [`peerimeter/peerimeter-agent`](https://github.com/peerimeter/peerimeter-agent) | `check`, `CodeRabbit` | Yes (`strict: true`) | Yes |
| [`peerimeter/peerimeter-service`](https://github.com/peerimeter/peerimeter-service) | `check`, `CodeRabbit` | Yes (`strict: true`) | Yes |

The settings now match on branch freshness and conversation resolution. Both
code repositories require the pipeline's aggregate `check`. Keep
inspecting, addressing and resolving review conversations before auto-merge;
GitHub now enforces both conversation resolution and branch freshness. Refresh
this summary when repository rules change.
