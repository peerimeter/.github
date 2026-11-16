<p align="center">
  <img src="https://raw.githubusercontent.com/peerimeter/.github/HEAD/assets/peerimeter-logo.png" alt="Peerimeter logo" width="600">
</p>

# Peerimeter

Peerimeter is a self-hosted WireGuard mesh agent for direct peer-to-peer connections. The agent can use local configuration independently, while an optional control-plane service provides a shared API for network and node configuration.

The project is organized around two independent repositories:

- **peerimeter-agent** — a headless Go agent targeting Linux, macOS, and Windows. It can run independently from local YAML or join a service-managed network and reconcile peers while running.
- **peerimeter-service** — a Go HTTP API for creating WireGuard networks, assigning tunnel addresses, registering agents, and exchanging peer records. Network metadata is in the main JSON store; node configurations and registrations are separate YAML files. See the [service setup guide](https://service.peerimeter.app/) for deployment.

Connections are direct only: UDP hole punching is best effort, with no TURN or relay fallback, so restrictive NAT or firewalls may prevent connectivity.

**Status:** The service-managed `join` and peer-reconciliation increment is implemented in [agent PR #51](https://github.com/peerimeter/peerimeter-agent/pull/51) and [service PR #7](https://github.com/peerimeter/peerimeter-service/pull/7); hosted checks and review are in progress. It uses a shared static API key, one service instance, and direct-only peer connections. Per-device credentials, revocation/leave, PostgreSQL, MQTT, user accounts, and multi-instance coordination are not included.

**Website:** [peerimeter.app](https://peerimeter.app) · [Agent installer](https://agent.peerimeter.app) · [Service setup](https://service.peerimeter.app).

See [the implementation plan](https://github.com/peerimeter/.github/blob/main/docs/planning/plan.md), [additional requirements](https://github.com/peerimeter/.github/blob/main/docs/planning/additional-plan.md), and [progress tracker](https://github.com/peerimeter/.github/blob/main/docs/planning/progress.md).

## Quick setup: register agents through the service

Deploy `peerimeter-service` using the [service setup guide](https://service.peerimeter.app/).
Configure a strong `PEERIMETER_API_KEY` on the service and set
`PEERIMETER_PUBLIC_API_URL` to its public HTTPS URL. The join bundle contains
that API URL and the same shared key; anyone with the bundle can administer all
networks, so distribute it only to the machines you trust.

**Preview build required for now:** the network API and `join` command are in
[service PR #7](https://github.com/peerimeter/peerimeter-service/pull/7) and
[agent PR #51](https://github.com/peerimeter/peerimeter-agent/pull/51). Until
those changes are merged and released, the current stable service and agent
installer do not support the commands below. To try them from source, check out
each PR head and build/run it on the target host:

```sh
git clone https://github.com/peerimeter/peerimeter-service.git
cd peerimeter-service
git fetch origin pull/7/head:discovery-preview
git switch discovery-preview
read -rsp "Set service API key: " PEERIMETER_API_KEY; printf '\n'; export PEERIMETER_API_KEY
export PEERIMETER_PUBLIC_API_URL='https://service.example.com'
export PEERIMETER_DATA_FILE="$PWD/peerimeter-config.json"
export PEERIMETER_LISTEN_ADDR=127.0.0.1
export PORT=8080
go run ./cmd/peerimeter-service
```

Keep the service running behind a trusted HTTPS reverse proxy, then use a
separate terminal for the remaining commands. Build the agent on each machine
where it will run:

```sh
git clone https://github.com/peerimeter/peerimeter-agent.git
cd peerimeter-agent
git fetch origin pull/51/head:discovery-preview
git switch discovery-preview
mkdir -p bin
go build -o bin/peerimeter-agent ./cmd/peerimeter-agent
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

Fetch the join bundle and save its `token` field directly into a root-owned
file. `umask 077` protects it before writing; `set -C` refuses to overwrite an
existing token file:

```sh
curl -fsS "$SERVICE_URL/api/v1/join-bundle" \
  -H "Authorization: Bearer $SERVICE_API_KEY" \
  | jq -er '.token' \
  | sudo sh -c 'umask 077; set -C; cat > /tmp/peerimeter-join-token'
```

The example uses `jq` to extract the JSON field. Repeat the authenticated
request on each trusted Linux/macOS node, or transfer the token over a trusted
channel and create the destination file as root with mode `0600`. Join once per
machine, using the same network ID and a unique node ID:

```sh
sudo ./bin/peerimeter-agent join \
  --token-file /tmp/peerimeter-join-token \
  --network-id home \
  --node-id workstation-a
```

On Windows, build the agent from the PR head above. Run elevated PowerShell.
Create the token file, restrict its ACL before writing the credential, then
join:

```powershell
git clone https://github.com/peerimeter/peerimeter-agent.git
Set-Location peerimeter-agent
git fetch origin pull/51/head:discovery-preview
git switch discovery-preview
New-Item -ItemType Directory -Force bin | Out-Null
$ServiceUrl = 'https://service.example.com'
$SecureKey = Read-Host 'Service API key' -AsSecureString
$ServiceApiKey = [System.Net.NetworkCredential]::new('', $SecureKey).Password
Invoke-RestMethod "$ServiceUrl/api/v1/networks" -Method Post `
  -Headers @{ Authorization = "Bearer $ServiceApiKey" } `
  -ContentType 'application/json' `
  -Body '{"version":1,"id":"home","name":"Home","cidr":"10.90.0.0/24"}'
$tokenPath = "$env:ProgramData\Peerimeter\join-token.txt"
New-Item -ItemType Directory -Force (Split-Path $tokenPath) | Out-Null
if (Test-Path $tokenPath) { throw "Token file already exists: $tokenPath" }
New-Item -ItemType File $tokenPath | Out-Null
icacls $tokenPath /inheritance:r /grant:r `
  "$($env:USERDOMAIN)\$($env:USERNAME):(F)" `
  "NT AUTHORITY\SYSTEM:(F)" "BUILTIN\Administrators:(F)"
$bundle = Invoke-RestMethod "$ServiceUrl/api/v1/join-bundle" `
  -Headers @{ Authorization = "Bearer $ServiceApiKey" }
[System.IO.File]::WriteAllText($tokenPath, $bundle.token)
go build -o bin/peerimeter-agent.exe ./cmd/peerimeter-agent
./bin/peerimeter-agent.exe join `
  --token-file $tokenPath `
  --network-id home `
  --node-id workstation-a
```

`join` creates and protects a WireGuard key, registers only its public key and
the selected primary physical interface's addresses, receives its tunnel
address from the service, and fetches peers every 15 seconds. Keep the command
running in the foreground. Repeat it on each node with a different `--node-id`.
The agent chooses an overlapping LAN address when available and otherwise uses
the peer's discovered public UDP endpoint. A subnet match does not guarantee
that the machines can reach each other; NAT and firewall limitations may still
block a direct tunnel. `join` has no server-side leave/revoke operation yet.

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
