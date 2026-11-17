<p align="center">
  <img src="https://raw.githubusercontent.com/peerimeter/.github/HEAD/assets/peerimeter-logo.png" alt="Peerimeter logo" width="600">
</p>

# Peerimeter

<p align="center">
  <a href="https://github.com/peerimeter/peerimeter-agent">peerimeter-agent</a> · <a href="https://github.com/peerimeter/peerimeter-service">peerimeter-service</a> · <a href="https://github.com/peerimeter/peerimeter-dashboard">peerimeter-dashboard</a>
</p>

## Project setup

Start AI coding agents from the workspace root (the directory that contains
`.github`) so they can discover the shared instructions. When this repository
is checked out there as `.github` alongside the Peerimeter repositories, run:

```sh
if [ -f .github/AGENTS.md ] && [ ! -e AGENTS.md ] && [ ! -L AGENTS.md ]; then
  ln -s .github/AGENTS.md AGENTS.md
fi
```

This creates the workspace-level `AGENTS.md` symlink only when the shared
instructions exist and no workspace-level `AGENTS.md` file or symlink is
already present.

Peerimeter is a self-hosted WireGuard mesh agent for direct peer-to-peer connections. The agent can use local configuration independently, while an optional control-plane service provides a shared API for network and node configuration.

The project is organized around three independent code repositories:

- **peerimeter-agent** — a headless Go agent targeting Linux, macOS, and Windows. It can run independently from local YAML or join a service-managed network and reconcile peers while running.
- **peerimeter-service** — a Go HTTP API for creating WireGuard networks, assigning tunnel addresses, registering agents, and exchanging peer records. Network metadata is in the main JSON store; node configurations and registrations are separate YAML files. See the [service setup guide](https://service.peerimeter.app/) for deployment.
- **peerimeter-dashboard** — a React and TypeScript browser UI for API-key sign-in, network and peer management, and cross-platform agent join commands. See the [dashboard source](https://github.com/peerimeter/peerimeter-dashboard).

Connections are direct only: UDP hole punching is best effort, with no TURN or relay fallback, so restrictive NAT or firewalls may prevent connectivity.

**Status:** The service-managed `join` and peer-reconciliation increment is merged in [agent PR #51](https://github.com/peerimeter/peerimeter-agent/pull/51) and [service PR #7](https://github.com/peerimeter/peerimeter-service/pull/7). The API-key dashboard is merged in [dashboard PR #1](https://github.com/peerimeter/peerimeter-dashboard/pull/1); its CORS allowlist, network-wide peer-list API, local development URLs, and configurable join URL HTTPS requirement are merged in [service PR #10](https://github.com/peerimeter/peerimeter-service/pull/10). It uses a shared static API key, one service instance, and direct-only peer connections. Per-device credentials, revocation/leave, PostgreSQL, MQTT, user accounts, and multi-instance coordination are not included.

**Website:** [peerimeter.app](https://peerimeter.app) · [Agent installer](https://agent.peerimeter.app) · [Service setup](https://service.peerimeter.app).

See [the implementation plan](https://github.com/peerimeter/.github/blob/main/docs/planning/plan.md), [additional requirements](https://github.com/peerimeter/.github/blob/main/docs/planning/additional-plan.md), and [progress tracker](https://github.com/peerimeter/.github/blob/main/docs/planning/progress.md).

## Quick setup: register agents through the service

Deploy `peerimeter-service` using the [service setup guide](https://service.peerimeter.app/).
Configure a strong `PEERIMETER_API_KEY` on the service and set
`PEERIMETER_PUBLIC_API_URL` to its public HTTPS URL. The join bundle contains
that API URL and the same shared key; anyone with the bundle can administer all
networks, so distribute it only to the machines you trust.

Run the latest service image published to GHCR. The `latest` tag tracks
versioned releases from `main`:

```sh
read -rsp "Set service API key: " PEERIMETER_API_KEY; printf '\n'; export PEERIMETER_API_KEY
export PEERIMETER_PUBLIC_API_URL='https://service.example.com'
docker pull ghcr.io/peerimeter/peerimeter-service:latest
docker volume create peerimeter-data
docker run -d --name peerimeter-service --restart unless-stopped \
  --publish 127.0.0.1:8080:8080 \
  --env PEERIMETER_API_KEY \
  --env PEERIMETER_PUBLIC_API_URL \
  --volume peerimeter-data:/var/lib/peerimeter \
  ghcr.io/peerimeter/peerimeter-service:latest
```

Keep the service behind a trusted HTTPS reverse proxy. Its container listens
on the host loopback at port 8080, and the named Docker volume persists the
network and node data. Install the agent on each machine where it will run.

```sh
# Linux/macOS: install the latest released agent.
curl -fsSL https://agent.peerimeter.app/install.sh | sudo sh
```

Create a WireGuard address pool through the API. Pick a private CIDR that does
not overlap the networks your machines already use:

```sh
export SERVICE_URL="https://service.example.com"
read -rsp "Service API key: " SERVICE_API_KEY; printf '\n'

curl -fsS "$SERVICE_URL/api/v1/networks" \
  -H "Authorization: Bearer $SERVICE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"version":1,"id":"home","name":"Home","cidr":"10.90.0.0/24"}'
```

Fetch a join token for the network and pipe it directly to the agent. The token
carries the network ID; its shared API credential still grants access to all
networks and nodes. The agent uses the hostname as node ID unless you pass
`--node-id`:

```sh
curl -fsS "$SERVICE_URL/api/v1/join-bundle?network_id=home" \
  -H "Authorization: Bearer $SERVICE_API_KEY" \
  | jq -er '.token' \
  | sudo peerimeter-agent join
```

The example uses `jq` to extract the JSON field. Repeat the authenticated
request on each trusted Linux/macOS node. Set `--node-id workstation-a` if you
want a stable custom node ID instead of the hostname.

On Windows, run elevated PowerShell to install the latest released agent,
then pipe the token to `join`:

```powershell
irm https://agent.peerimeter.app/install.ps1 | iex
$ServiceUrl = 'https://service.example.com'
$SecureKey = Read-Host 'Service API key' -AsSecureString
$ServiceApiKey = [System.Net.NetworkCredential]::new('', $SecureKey).Password
Invoke-RestMethod "$ServiceUrl/api/v1/networks" -Method Post `
  -Headers @{ Authorization = "Bearer $ServiceApiKey" } `
  -ContentType 'application/json' `
  -Body '{"version":1,"id":"home","name":"Home","cidr":"10.90.0.0/24"}'
$bundle = Invoke-RestMethod "$ServiceUrl/api/v1/join-bundle?network_id=home" `
  -Headers @{ Authorization = "Bearer $ServiceApiKey" }
$bundle.token | & "$env:ProgramFiles\Peerimeter\peerimeter-agent.exe" join
```

`join` creates and protects a WireGuard key, registers only its public key and
the selected primary physical interface's addresses, receives its tunnel
address from the service, and fetches peers every 15 seconds. Keep the command
running in the foreground. Repeat it on each node; hostname is used as node ID
unless you set `--node-id`.
The agent chooses an overlapping LAN address when available and otherwise uses
the peer's discovered public UDP endpoint. A subnet match does not guarantee
that the machines can reach each other; NAT and firewall limitations may still
block a direct tunnel. `join` has no server-side leave/revoke operation yet.

## GitHub pull request settings

Checked through the GitHub REST API on **2026-11-16**. The shared planning/site
repository and all three code repositories use the same general pull request
settings. The `main` branch protections are listed separately below.

| Repository | Auto-merge | Always suggest updating PR branches | Delete head branch after merge | Merge strategy |
| --- | --- | --- | --- | --- |
| [`peerimeter/.github`](https://github.com/peerimeter/.github) | Yes | Yes | Yes | Rebase only |
| [`peerimeter/peerimeter-agent`](https://github.com/peerimeter/peerimeter-agent) | Yes | Yes | Yes | Rebase only |
| [`peerimeter/peerimeter-service`](https://github.com/peerimeter/peerimeter-service) | Yes | Yes | Yes | Rebase only |
| [`peerimeter/peerimeter-dashboard`](https://github.com/peerimeter/peerimeter-dashboard) | Yes | Yes | Yes | Rebase only |

| Repository `main` protection | Required status checks | Require up-to-date branch | Require all review conversations resolved | Linear history | Force-push / branch deletion | Required approvals | Enforce for admins |
| --- | --- | --- | --- | --- | --- | --- | --- |
| [`peerimeter/.github`](https://github.com/peerimeter/.github) | `CodeRabbit` | Yes (`strict: true`) | Yes | Yes | Disabled | 0 | No |
| [`peerimeter/peerimeter-agent`](https://github.com/peerimeter/peerimeter-agent) | `check`, `CodeRabbit` | Yes (`strict: true`) | Yes | Yes | Disabled | 0 | No |
| [`peerimeter/peerimeter-service`](https://github.com/peerimeter/peerimeter-service) | `check`, `CodeRabbit` | Yes (`strict: true`) | Yes | Yes | Disabled | 0 | No |
| [`peerimeter/peerimeter-dashboard`](https://github.com/peerimeter/peerimeter-dashboard) | `check`, `CodeRabbit` | Yes (`strict: true`) | Yes | Yes | Disabled | 0 | No |

The dashboard repository now matches the agent and service code-repository
settings. Keep inspecting, addressing and resolving review conversations before
auto-merge; GitHub enforces branch freshness and conversation resolution.
Refresh this summary when repository rules change.
