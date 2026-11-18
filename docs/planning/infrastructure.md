# Peerimeter infrastructure plan

**Status:** evaluation and proposed first deployment shape  
**Checked:** 2026-11-18  
**Scope:** free-tier suitability and Terraform coverage for Render, Railway,
Supabase, Better Stack, and a managed MQTT broker. Prices and quotas change;
recheck provider and plan details before provisioning.

## Current architecture constraints

The service currently supports a file backend and PostgreSQL. PostgreSQL
migrations and a single Compose deployment are implemented; MQTT notifications,
and scoped device credentials are not implemented. PostgreSQL provides shared
persistent state for service replicas; the file backend remains single-process
only, and the proposed two-instance hosted topology still needs an integration
verification run.
The existing deployment default is self-hosted Mosquitto, and PostgreSQL remains
the authoritative store when MQTT is added.

The agent's direct WireGuard data plane does not pass through the service.
Hosted infrastructure carries control-plane HTTPS and (when implemented) MQTT
notifications only. MQTT loss must be repaired by API reconciliation; it cannot
be the only way to deliver desired state.

## Provider and free-plan comparison

| Service | Terraform provider | Free plan and IaC assessment |
| --- | --- | --- |
| Render | Official `render-oss/render` provider. It manages Render services; provider configuration needs an API key and owner ID. | Free web services are available, but sleep after 15 minutes without inbound traffic, cold-start in about a minute, have ephemeral filesystems, and are explicitly not recommended for production. Suitable for a demo API or static UI, not an always-on production control plane. Render recommends Blueprints if all managed resources are on Render; Terraform is useful when combining providers. |
| Railway | `terraform-community-providers/railway` exists, but it is community maintained, not an official Railway provider. | Free provides $1/month of usage credit. Terraform coverage should be treated as a community integration risk and checked against supported resource schemas before adoption. Could host a low-traffic demo, but not a predictable always-on free production service. |
| Supabase | Official `supabase/supabase` provider, currently documented as Public Alpha. It can manage project settings and database/API resources. | Free permits two active projects across owner/admin organizations. Free projects pause after one week of inactivity; included database is 500 MB, and automatic backups/PITR are absent. Reasonable for development/staging validation; weak production fit for an always-on control plane. The service can use its PostgreSQL endpoint, but must verify driver/TLS/network settings and migrations against a dedicated project. |
| Better Stack | Official Uptime and Telemetry Terraform providers. | Free personal-project tier currently lists 10 monitors/heartbeats and one status page; Telemetry includes limited log/trace retention. Good fit for uptime and basic observability IaC. It is not an application host, database, or MQTT broker. |
| MQTT candidate: EMQX Cloud Serverless | EMQ publishes an `emqxcloud` Terraform provider. Confirm in a small spike that it provisions the required Serverless deployment and per-device authentication/ACL resources; do not assume every console feature is exposed. | Free quota currently includes 1 million connection/session minutes and 1 GB monthly traffic with spend limit set to zero. Promising for a prototype, subject to verifying provider coverage, ACL behavior, connection quotas and outage/reconnect behavior. It is a managed MQTT service rather than self-hosted Mosquitto. |

The Railway provider is not first-party even though a Registry entry exists.
Provider availability means the provider itself is free to use; it does not
mean the hosted resources or Terraform state storage are free. Keep Terraform
state encrypted and access-controlled, and never commit provider tokens or
database credentials.

## Proposed phased deployment

### Target topology requested for the first hosted deployment

```text
Clients / Better Stack monitor
             |
             v
Render Web Service (Docker image: zerosuxx/simple-load-balancer:0.1.2)
  BACKEND_URLS=<Railway instance 1 URL>,<Railway instance 2 URL>
             |
       +-----+-----+
       |           |
       v           v
Railway service 1  Railway service 2
       |           |
       +-----+-----+
             v
Supabase PostgreSQL
```

This is the requested target shape, not a claim that it is production-ready or
already provisioned. Configure both Railway instances from the same
`peerimeter-service` image/release and pass each the same
`PEERIMETER_STORAGE_BACKEND=postgres` and Supabase-backed
`PEERIMETER_DATABASE_URL` secret. The public API URL and Better Stack monitor
target should be the Render load balancer's HTTPS address; monitor
`/healthz` (the service currently returns HTTP 204 when healthy).

Before relying on this topology, validate that the load balancer image accepts
the supplied comma-separated `BACKEND_URLS`, binds the port Render supplies via
`PORT` (or has a documented compatible port setting), routes health checks and
API paths correctly, and removes an unhealthy backend from rotation. Confirm
the exact Render/Terraform resource schema can select this prebuilt Docker image
and configure its environment variables. Pin the image by digest after
verification while retaining `0.1.2` as the requested version reference.

PostgreSQL is the shared source of truth for both service replicas. The current
PostgreSQL backend stores network, node and registration records in the shared
database; network creation and node address allocation use database
transactions/locks to protect concurrent writes. With both replicas using the
same database and compatible service configuration (including the same API key
and public URL), the architecture is intended to support requests being served
by either instance.

The remaining caveat is verification and feature scope, rather than a lack of
shared persistence: the service README says multi-instance API authorization is
not included. Today this means there is no distributed per-device/user auth
system; it does not by itself mean the current shared API key cannot be
configured identically on both replicas. Before production traffic, verify
concurrent startup/migrations, identical auth and CORS configuration, request
routing across both instances, and concurrent registration/network writes.
MQTT coordination is also not implemented, so it is not part of this two-node
HTTP deployment yet.

Render Free web services sleep after 15 minutes without inbound requests and
may restart; the first request can take about a minute to wake the load
balancer. This means Better Stack monitoring through the Render URL may see a
cold-start delay, and the load balancer can itself be unavailable while asleep.
Render Free also grants 750 shared instance hours per workspace per month,
enough for one service running continuously, not multiple continuously active
Render services. Railway's Free plan offers $1 of monthly usage credit; verify
whether two live instances fit that credit. Therefore this exact topology is
best treated as a free-tier evaluation unless the compute plans are upgraded.

The Render load balancer only fronts HTTP(S) control-plane traffic. It is not
the WireGuard data plane and must not be used as a claim of relayed WireGuard
connectivity. The separate MQTT decision remains open: self-hosted Mosquitto is
the baseline and EMQX Cloud Serverless remains an evaluation candidate.

### Phase A — low-cost hosted evaluation

- Use Supabase Free only for a disposable/test PostgreSQL project, with a
  dedicated project and no production data. Run the existing migration and
  integration suite against it; record pause/resume and connection behavior.
- Evaluate the service on Render Free or Railway Free as a demo. Keep a
  persistent API health check and verify wake-up/cold-start behavior. Do not
  claim this is production-grade uptime.
- Evaluate EMQX Cloud Serverless with explicit zero spend limit. Keep the MQTT
  integration behind configuration until the service supports durable event
  delivery and agents have periodic HTTPS reconciliation.
- Configure Better Stack free monitors for the API health endpoint and public
  dashboard, plus a status page if useful. Avoid shipping sensitive or
  high-volume logs to the free telemetry allowance.
- Keep the dashboard on its existing static hosting path unless there is a
  concrete reason to move it; static hosting is independent from the service.

### Phase B — first production-shaped deployment

- The requested production direction is Render for the public load balancer,
  two Railway service instances, Supabase PostgreSQL, and Better Stack uptime
  monitoring of the Render address. Keep it conditional on the replica,
  cost, image behavior, and backup/restore checks above.
- Use PostgreSQL with backups and a tested restore path, and managed MQTT with
  per-device ACLs; alternatively deploy the existing Compose stack on a small
  VPS with persistent storage and managed backups.
- Ensure both service replicas use the same Supabase database and identical
  service configuration; verify concurrent startup/migrations and requests
  before treating horizontal scaling as production-ready. MQTT outbox/retry
  remains a separate requirement for future broker-based notifications.
- Terminate HTTPS at a trusted platform proxy. Restrict CORS to the dashboard
  origin. Keep the service API key and broker admin credentials in the hosting
  platform's secret store; never put broker admin credentials in agents.
- Back up PostgreSQL outside the database host and perform a restore exercise
  before treating the deployment as production.
- Use Better Stack for external uptime checks and alerting within the selected
  plan's quotas. Add log/trace export only after reviewing retention and
  sensitive-field scrubbing.

## Terraform boundaries

Use separate Terraform roots/states for `development` and `production`, with
provider versions pinned and secrets supplied from environment/CI secret
stores. Terraform should own platform-level objects that its provider exposes
(service, project settings, monitors, broker deployment/ACLs). Keep application
database schema migrations in the service's Goose migration workflow, rather
than trying to represent schema as Terraform resources. For Render-only
prototypes, compare the official provider with Render Blueprints. For Railway,
evaluate the community provider's maintenance cadence, import/drift behavior,
and resource deletion semantics before placing production resources under it.

Do not store Terraform state in a free hosted state service until its access
controls, locking and recovery properties are confirmed. Local state is
acceptable only for an individual disposable prototype and must be encrypted
and backed up; it is not appropriate for shared production work.

## Decisions and acceptance checks

No hosting provider is selected for production yet. The existing self-hosted
Compose deployment with Mosquitto remains the implementation baseline; EMQX
Cloud Serverless is the leading managed-broker experiment, not an adopted
replacement.

Before choosing a production target:

1. Run the service's PostgreSQL migration and integration suite against a
   dedicated managed PostgreSQL instance; document provider/version, region,
   TLS behavior and observed results.
2. Confirm the selected compute plan remains awake and supports required
   inbound HTTPS, outbound PostgreSQL TLS and MQTT TLS connections.
3. Confirm MQTT per-device topic ACLs, reconnect behavior, quota alarms and
   zero-spend/usage controls with the production broker candidate.
4. Confirm Terraform plan/apply/import/destroy behavior for each resource and
   inspect state for accidental secret exposure.
5. Verify backup and restore, API health monitoring, and a full restart with
   persistent data before promoting the deployment.

## Primary references

- [Render Terraform provider](https://render.com/docs/terraform-provider)
- [Render free service limits](https://render.com/docs/free)
- [Render web services and prebuilt images](https://render.com/docs/web-services)
- [Railway Terraform community provider](https://registry.terraform.io/providers/terraform-community-providers/railway/latest/docs)
- [Railway pricing plans](https://docs.railway.com/pricing/plans)
- [Supabase Terraform provider](https://supabase.com/docs/guides/deployment/terraform)
- [Supabase Free plan and billing](https://supabase.com/docs/guides/platform/billing-on-supabase)
- [Supabase pricing limits](https://supabase.com/pricing)
- [Better Stack Terraform integration](https://betterstack.com/docs/getting-started/integrations/terraform/)
- [Better Stack pricing](https://betterstack.com/pricing)
- [EMQX Serverless free quota and spend limit](https://docs.emqx.com/en/cloud/latest/deployments/spend_limit.html)
- [EMQX Cloud Terraform provider namespace](https://registry.terraform.io/namespaces/emqx)
