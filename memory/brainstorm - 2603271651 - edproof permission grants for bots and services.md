# Brainstorm — EdProof Permission Grants for Bots and Services

**Status:** active
**Date:** 2026-03-27

## Seed

**Problem:** EdProof proves identity (Layer 1-2) and leaves access decisions to the verifier (Layer 4). But there's no standard mechanism for an entity to *request* specific, time-limited permissions from a resource owner — and have those grants be auditable, revocable, and independently verifiable.

**Goal:** Design an EdProof-based permission grant system where:
- Bots (and other services) request specific permissions to specific resources
- Dual approval: bot owner + resource owner must both approve
- Grants are time-limited (TTL), scoped (least privilege), and revocable
- The grant itself is a verifiable artifact (not just a database row)
- Audit trail: who requested, who approved, what scope, what TTL

**Core tension:** EdProof separates identity from permission. This new system must respect that separation — identity proof is Layer 1-2, permission grants are a *new concern* that lives alongside Layer 3-4 (registry + policy), not inside them.

**Context from existing spec:**
- EdProof credential carries key identity only — no permissions
- "The verifier owns the decision" — this must remain true
- Registry is dataset, not authority — permission grants should follow same principle
- Ceremony-once model — but permission grants are inherently temporal

## Ideas

### Architectural Approaches

**A1. Permission Grant as a Signed Document (new Layer 2.5)**
A permission grant is a signed JSON/CBOR document, separate from the identity credential. Signed by both bot owner and resource owner. Carried by the bot alongside its identity credential. Verifier checks identity credential + permission grant + own policy.
- Pro: follows the "document, not a lease" pattern
- Pro: offline-verifiable
- Con: needs a way to revoke before TTL expires

**A2. Permission Registry (extension of Layer 3)**
Permission grants are entries in a registry — a dataset the verifier consults. The registry holds: fingerprint, granted scopes, TTL, approver signatures, grant timestamp.
- Pro: natural extension of existing registry model
- Pro: revocation = registry update (add revocation entry)
- Con: requires registry availability at verify time (breaks offline model?)
- Mitigation: cached registry state is acceptable per existing spec

**A3. Dual-Signed Capability Token**
A capability token is a self-contained, dual-signed artifact:
```
{
  bot_fingerprint: "SHA256:...",
  bot_owner_fingerprint: "SHA256:...",
  resource: "gyros://board/project-x",
  scopes: ["read", "write:filter"],
  not_before: "2026-03-27T17:00:00Z",
  not_after: "2026-03-28T17:00:00Z",
  nonce: "...",
  bot_owner_signature: "...",
  resource_owner_signature: "..."
}
```
Bot presents: identity credential + capability token. Verifier checks both.
- Pro: self-contained, offline-verifiable, auditable
- Pro: dual signatures encode the approval chain
- Con: revocation still needs a registry or CRL mechanism

**A4. Request-Approve-Grant Event Chain (Git-native)**
The entire permission lifecycle is a series of signed Git commits:
1. Bot signs a request commit
2. Bot owner signs an approval commit
3. Resource owner signs a grant commit
4. Revocation = another signed commit

The Git repo *is* the audit trail. Verifiers pull/cache the repo.
- Pro: perfect audit trail, tamper-evident
- Pro: works with existing EdProof registry model (Git as registry)
- Con: latency — Git pull is not instant
- Mitigation: for high-frequency access, cache + webhook notifications

### Approval Flow Variants

**B1. Sequential Dual Approval**
Bot → Bot Owner → Resource Owner → Grant issued
Strict ordering. Bot owner approves first, then resource owner sees a pre-approved request.

**B2. Parallel Dual Approval**
Bot → [Bot Owner, Resource Owner] → Grant issued when both approve
Either can approve first. Grant issued only when both signatures present.

**B3. Policy-Gated Auto-Approval**
Low-risk scopes (e.g., read-only) auto-approved by policy. High-risk scopes require full dual approval. Resource owner defines the risk tiers.
- Pro: reduces friction for routine access
- Con: policy misconfiguration risk

**B4. Delegation with Constraints**
Bot owner pre-authorizes: "this bot may request read access to any gyros resource for up to 24h without my approval." Resource owner still approves. Reduces one approval hop for predictable patterns.

### Scope Modeling

**C1. Resource + Action + Constraints**
```
resource: "grafana://folder/alerts-prod"
action: "read"
constraints: { max_queries: 1000 }
```

**C2. URN-style Scopes**
```
urn:edproof:gyros:board:project-x:read
urn:edproof:grafana:folder:alerts-prod:write:alert-rules
urn:edproof:antark:dataset:customers:read
```

**C3. Domain-Specific Scope Sets**
Each resource type defines its own scope vocabulary. The permission system carries opaque scope strings — the resource interprets them.
- Pro: no central scope registry needed
- Pro: each domain evolves independently
- Con: no cross-domain consistency

### Revocation Approaches

**D1. Registry-Based Revocation**
Add revocation entry to permission registry. Verifier checks on access.

**D2. Short TTL + No Revocation**
Grants expire quickly (1h, 24h). No explicit revocation — just don't renew. Simple but coarse.

**D3. Revocation List (CRL-style)**
Separate signed document listing revoked grant IDs. Verifier fetches periodically.

**D4. Event-Based Revocation**
Webhook/event pushed to verifiers when a grant is revoked. Near-real-time but requires event infrastructure.

### Use Case Specific Ideas

**E1. Gyros: Board-Level Scoping**
Bot requests access to a specific gyros board. Scopes: read, write:filter, write:number, admin. TTL: 1-24h for automated tasks. Bot owner = the team that deployed the bot. Resource owner = board owner.

**E2. Grafana: Folder/Team Mapping**
EdProof permission grant maps to Grafana service account with folder-scoped permissions. The grant is the *authorization* to create the service account. Grafana folder ID in the scope. TTL enforced by service account expiry.

**E3. Component Registry: Namespace-Scoped Publish**
Bot requests `publish` to a specific namespace for a CI/CD pipeline run. TTL = pipeline duration + buffer. Auto-revoked on pipeline completion. Scope: `urn:edproof:registry:namespace:my-lib:publish`.

**E4. Generic API Gateway Pattern**
Gateway validates EdProof identity + permission grant. Translates grant scopes to API-level authorization. Gateway is the verifier — it owns the decision about what the scopes mean in its context.

## Clusters

*To be filled after discussion*

## Standouts

*To be filled after discussion*
