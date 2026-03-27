---
tldr: Concrete use cases for EdProof-based dynamic permission grants — who requests, who approves, what scope, what TTL
category: use-cases
---

# Permission Grant Use Cases

## Target

Concrete scenarios showing how bots and services use EdProof identity to request, receive, and exercise time-limited, scoped permissions to specific resources. Each use case describes the actors, the request, the approval chain, the access pattern, and the audit trail.

## Prerequisites

Every use case assumes:
- The bot has an EdProof identity (Ed25519 key pair, attested credential) — stable, long-lived
- The bot owner has an EdProof identity
- The resource owner has an EdProof identity
- Permission grants are separate artifacts from identity credentials — a "visa" alongside the "passport"
- Approval is sequential: bot owner approves first, then resource owner
- Either approval can be policy-automated — EdProof does not prescribe how

## Model

```
Bot                    Bot Owner              Resource Owner         Resource
 │                        │                        │                    │
 │── permission request ─►│                        │                    │
 │   (resource, scopes,   │                        │                    │
 │    TTL, identity proof) │                        │                    │
 │                        │── approved request ───►│                    │
 │                        │   (+ bot owner sig)     │                    │
 │                        │                        │── grant ──────────►│
 │◄── capability token ───┤◄───────────────────────┤  (registered)      │
 │   (dual-signed, scoped, │                        │                    │
 │    time-limited)        │                        │                    │
 │                        │                        │                    │
 │── access request ─────────────────────────────────────────────────►│
 │   (identity credential │                        │                    │
 │    + capability token)  │                        │                    │
 │                        │                        │     verify identity │
 │                        │                        │     verify grant    │
 │                        │                        │     check TTL      │
 │                        │                        │     check revocation│
 │                        │                        │     apply policy    │
 │◄─────────────────────────────────────────────────── scoped response │
```

For legacy resources that don't support EdProof natively, a sidecar/proxy sits in front of the resource, verifies the bot's identity and grant, and proxies authorized calls using the resource's native credentials.

---

## Use Case A: Gyros Resources

### A1. Bot reads a Gyros board for status reporting

**Actors**
- Bot: `deploy-status-bot` (fingerprint `SHA256:abc1...`), owned by Platform Team
- Bot owner: Platform Team lead (fingerprint `SHA256:def2...`)
- Resource owner: Project Alpha board owner (fingerprint `SHA256:ghi3...`)

**Request**
```
resource:    gyros://board/project-alpha
scopes:      [read]
ttl:         1h
reason:      "Fetch task statuses for deploy summary report"
```

**Approval chain**
1. Bot owner (Platform Team lead): auto-approved by policy — "deploy-status-bot may request `read` on any gyros board for up to 4h"
2. Resource owner (Project Alpha board owner): auto-approved by policy — "`read` scopes auto-approved for bots with verified EdProof identity"

**Access pattern**
- Bot presents identity credential + capability token to Gyros
- Gyros verifies: identity valid, grant signatures valid, TTL not expired, no revocation
- Bot reads board state (tasks, statuses, filters) — write operations rejected
- After 1h, the token expires. Bot must re-request if it needs more time

**Audit trail**
- Request: `deploy-status-bot` requested `read` on `gyros://board/project-alpha` for 1h
- Bot owner approval: auto-approved by policy `platform-team/read-auto-approve`
- Resource owner approval: auto-approved by policy `project-alpha/read-open`
- Access log: 14 read operations over 47 minutes, then TTL expired

---

### A2. Bot writes filters on a Gyros board for automated triage

**Actors**
- Bot: `triage-bot` (fingerprint `SHA256:jkl4...`), owned by QA Team
- Bot owner: QA Team lead
- Resource owner: Sprint Board owner

**Request**
```
resource:    gyros://board/sprint-42
scopes:      [read, write:filter]
ttl:         30m
reason:      "Create priority filters based on defect severity"
```

**Approval chain**
1. Bot owner (QA Team lead): manual approval — `write` scopes require human sign-off per QA team policy
2. Resource owner (Sprint Board owner): manual approval — board owner reviews that `write:filter` is reasonable for triage automation

**Access pattern**
- Bot reads current board state, creates/modifies filters only
- Cannot modify tasks, numbers, board settings, or other entities — only `filter` scope granted
- After 30m, grant expires automatically

**Audit trail**
- Request: `triage-bot` requested `[read, write:filter]` on `gyros://board/sprint-42` for 30m
- Bot owner approval: manual, signed by QA Team lead at 14:02
- Resource owner approval: manual, signed by Sprint Board owner at 14:05
- Access log: 3 reads, 2 filter creations, grant expired at 14:35

---

### A3. Bot needs admin access to restructure a Gyros project

**Actors**
- Bot: `reorg-bot`, owned by Engineering Manager
- Resource owner: Gyros project owner

**Request**
```
resource:    gyros://project/backend-services
scopes:      [admin]
ttl:         2h
reason:      "Restructure boards and reassign ownership per Q2 reorg plan"
```

**Approval chain**
1. Bot owner (Engineering Manager): manual approval — `admin` always requires explicit sign-off
2. Resource owner (Gyros project owner): manual approval — `admin` on a project is high-risk; owner verifies the reorg plan before approving

**Access pattern**
- Bot has full project access for 2h
- All operations logged individually
- Resource owner can revoke at any time if something goes wrong

**What revocation looks like**
- At 15:20, resource owner sees unexpected board deletions in the audit stream
- Resource owner signs a revocation entry in the permission registry
- Next access attempt by reorg-bot is rejected: grant revoked
- Audit trail shows: grant issued at 14:00, revoked at 15:20, reason: "unexpected deletions"

---

## Use Case B: Antark (Internal Platform)

### B1. Bot queries a dataset for analytics processing

**Actors**
- Bot: `analytics-pipeline` (CI/CD job), owned by Data Team
- Resource owner: Antark dataset owner (`customers-eu`)

**Request**
```
resource:    antark://dataset/customers-eu
scopes:      [read]
ttl:         4h
reason:      "Monthly cohort analysis pipeline run"
```

**Approval chain**
1. Bot owner (Data Team): auto-approved by policy — pipeline bots may request `read` on approved datasets
2. Resource owner: auto-approved by policy — `customers-eu` allows `read` from verified Data Team bots

**Access pattern**
- Bot reads dataset records matching its query parameters
- No write, delete, or schema modification possible
- TTL covers the expected pipeline duration plus buffer
- No sidecar needed if Antark supports EdProof natively; sidecar if Antark uses its own auth system

**Audit trail**
- Request, dual auto-approval, 1.2M records read over 2h 17m, grant expired unused at 4h mark

---

### B2. Bot writes results to a specific Antark namespace

**Actors**
- Bot: `analytics-pipeline`, owned by Data Team
- Resource owner: Antark namespace owner (`reports/monthly`)

**Request**
```
resource:    antark://namespace/reports/monthly
scopes:      [write]
ttl:         1h
reason:      "Write monthly cohort analysis results"
```

**Approval chain**
1. Bot owner (Data Team): auto-approved — same pipeline, write to designated output namespace
2. Resource owner: auto-approved — namespace `reports/monthly` is a designated pipeline output

**Access pattern**
- Bot writes only to `reports/monthly` — cannot read other namespaces, cannot modify schema, cannot access other outputs
- Two separate grants for one pipeline: B1 (read input) + B2 (write output) — least privilege, separately scoped

---

## Use Case C: Component Registries / Internal Services

### C1. Bot publishes a package during CI/CD

**Actors**
- Bot: `ci-runner-main` (GitHub Actions), owned by Platform Team
- Resource owner: Component registry admin (`@company/ui-kit` namespace)

**Request**
```
resource:    registry://namespace/@company/ui-kit
scopes:      [publish]
ttl:         15m
reason:      "Publish ui-kit v3.2.1 from release pipeline"
```

**Approval chain**
1. Bot owner (Platform Team): auto-approved by policy — release pipeline bots may `publish` to namespaces they own
2. Resource owner (registry admin): auto-approved by policy — `@company/ui-kit` allows `publish` from `ci-runner-main`

**Access pattern**
- Bot publishes exactly one package version
- Cannot read other packages, modify registry settings, or publish to other namespaces
- TTL is short (15m) — just long enough for the publish operation
- Grant auto-expires; no need for explicit revocation

**Audit trail**
- Request at 09:12, dual auto-approval, publish of `@company/ui-kit@3.2.1` at 09:13, grant expired at 09:27

---

### C2. Bot deploys a service to a specific environment

**Actors**
- Bot: `deploy-agent-prod`, owned by SRE Team
- Resource owner: Production environment owner

**Request**
```
resource:    deploy://environment/production/service/api-gateway
scopes:      [deploy]
ttl:         30m
reason:      "Rolling deploy of api-gateway v2.8.0 per approved change request CR-1234"
```

**Approval chain**
1. Bot owner (SRE Team): manual approval — production deploys always require human sign-off
2. Resource owner (Production owner): manual approval — verifies change request CR-1234 is approved and deploy window is open

**Access pattern**
- Bot can deploy only `api-gateway` to `production` — no other services, no other environments
- The change request reference is part of the grant audit trail
- If deploy takes longer than 30m, bot must re-request (this is intentional — forces re-evaluation)

---

## Use Case D: Grafana (Observability)

### D1. Bot reads specific dashboards for alerting logic

**Actors**
- Bot: `alert-evaluator`, owned by Observability Team
- Resource owner: Grafana folder owner (`Production/API Metrics`)

**Request**
```
resource:    grafana://folder/production-api-metrics
scopes:      [read:dashboard]
ttl:         24h
reason:      "Continuous alert evaluation against API latency dashboards"
```

**Sidecar required** — Grafana doesn't support EdProof natively.

**Approval chain**
1. Bot owner (Observability Team): auto-approved — alert evaluators may read dashboards
2. Resource owner (folder owner): auto-approved — `read:dashboard` is low-risk

**Access pattern (via sidecar)**
- Sidecar holds a Grafana service account token with folder-scoped read permissions
- Bot presents identity + capability token to sidecar
- Sidecar verifies, then proxies read requests to Grafana using the service account
- Bot can only read dashboards in `Production/API Metrics` folder — nothing else
- After 24h, bot re-requests (long TTL appropriate for continuous low-risk reads)

**Audit trail**
- Sidecar logs every proxied request: which dashboard, when, what data returned
- Grant lifecycle logged separately: request, approvals, expiry/renewal

---

### D2. Bot creates alert rules in a specific folder

**Actors**
- Bot: `alert-manager-bot`, owned by SRE Team
- Resource owner: Grafana folder owner (`Production/Alerts`)

**Request**
```
resource:    grafana://folder/production-alerts
scopes:      [read:dashboard, write:alert-rule]
ttl:         1h
reason:      "Create P1 latency alert rules for new api-gateway deployment"
```

**Sidecar required.**

**Approval chain**
1. Bot owner (SRE Team): manual approval — `write` scopes to production alerts require human sign-off
2. Resource owner (folder owner): manual approval — alert rule creation in production needs review

**Access pattern (via sidecar)**
- Sidecar holds Grafana service account with `Editor` role scoped to `Production/Alerts` folder
- Bot can read dashboards (to reference panels) and create alert rules — nothing else
- Cannot modify existing alerts, delete alerts, or access other folders
- After 1h, grant expires — alert rules persist, but bot can no longer modify them

---

### D3. Bot reads a datasource for health checks

**Actors**
- Bot: `health-checker`, owned by Platform Team
- Resource owner: Grafana datasource admin (`prometheus-prod`)

**Request**
```
resource:    grafana://datasource/prometheus-prod
scopes:      [read:query]
ttl:         12h
reason:      "Run health check queries against production Prometheus"
```

**Sidecar required.**

**Approval chain**
1. Bot owner: auto-approved — health checkers may query datasources
2. Resource owner: auto-approved — `read:query` on Prometheus is low-risk; rate-limited by sidecar

**Access pattern (via sidecar)**
- Sidecar proxies PromQL queries to Grafana's datasource proxy API
- Sidecar enforces: read-only queries, rate limit (e.g., max 60 queries/minute), no admin operations
- Bot cannot modify datasource configuration, create dashboards, or access other datasources

---

## Use Case E: Generic Resources (Template)

### E1. API Gateway / Internal API

**Request pattern**
```
resource:    api://service/payments/v2
scopes:      [read:transactions]
ttl:         2h
reason:      "Reconciliation batch job"
```
- Gateway is the verifier (native or sidecar)
- Scopes map to API endpoint permissions
- Rate limits and IP restrictions as additional policy constraints

### E2. Data Store

**Request pattern**
```
resource:    datastore://bucket/analytics-raw
scopes:      [read, write:prefix/output/2026-03]
ttl:         6h
reason:      "Monthly ETL pipeline"
```
- Write scoped to a specific prefix/partition — not the entire bucket
- Read on the full bucket, write only to output path

### E3. CI/CD System

**Request pattern**
```
resource:    ci://pipeline/release-backend
scopes:      [trigger, read:logs]
ttl:         45m
reason:      "Trigger release pipeline and monitor completion"
```
- Bot can trigger one specific pipeline and read its logs — nothing else
- Cannot modify pipeline config, access other pipelines, or read secrets

### E4. Documentation / Ticket System

**Request pattern**
```
resource:    tickets://project/PLATFORM
scopes:      [read, write:comment]
ttl:         4h
reason:      "Post deployment status updates on release tickets"
```
- Bot can read tickets and add comments — cannot create, modify, close, or delete tickets
- Sidecar required for Jira (holds Jira API token, proxies authorized operations)

---

## Cross-Cutting Principles

Every use case follows these principles:

1. **Least privilege**: Bot requests only the scopes it needs, for only the time it needs them
2. **TTL always**: Every grant expires. No permanent permissions. Re-request if you need more time
3. **Dual approval always**: Bot owner and resource owner both sign off (manually or via policy)
4. **Identity is stable, permissions are temporal**: EdProof credential lives for years; permission grants live for minutes to hours
5. **Revocation is possible**: Any approver can revoke a grant before TTL expires
6. **Audit is complete**: Every request, approval, access, and revocation is logged with who, what, when, and why
7. **Sidecar only when needed**: Only for resources that don't support EdProof natively — not an architectural requirement
8. **The verifier owns the decision**: Whether native or sidecar, the verifier applies its own policy on top of the grant

## Mapping

> [[RFC-EDPROOF.md]]
> [[eidos/spec - edproof protocol.md]]
> [[memory/brainstorm - 2603271651 - edproof permission grants for bots and services.md]]
