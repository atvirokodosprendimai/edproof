# EdGrant: A Permission Grant Protocol for EdProof Identities

```
Status:       Draft
Version:      0.1
Authors:      oldroot
Date:         2026-03-28
Requires:     RFC-EDPROOF.md (v0.1)
```

## Abstract

EdGrant is a permission grant protocol built on EdProof identity. It
defines how any entity — bot, agent, service, or human — requests
scoped, time-limited access to a resource, and how the resource owner
evaluates and grants that access. The protocol is request-initiated:
the entity discovers what a resource accepts, asks for what it needs,
and the resource owner decides.

The protocol has two entities: the **requestor** (the entity seeking
access) and the **resource owner** (the entity controlling the
resource). What evidence the resource owner requires before granting —
bot owner co-approval, risk assessment, identity checks, or nothing at
all — is the resource owner's policy decision. The protocol does not
prescribe approval chains.

EdGrant produces a **capability token**: a signed, time-limited,
scope-bound artifact that the requestor carries and presents to the
resource. The resource verifies the token locally. The resource owner
is not contacted at access time.

A formally verified Tamarin model covers the grant exchange.

---

## Table of Contents

- [1. Introduction](#1-introduction)
  - [1.1. The Visa Model](#11-the-visa-model)
  - [1.2. Design Principles](#12-design-principles)
  - [1.3. Relationship to EdProof](#13-relationship-to-edproof)
  - [1.4. Relationship to Existing Work](#14-relationship-to-existing-work)
- [2. Terminology](#2-terminology)
- [3. Protocol Overview](#3-protocol-overview)
- [4. Phase 0 — Discovery](#4-phase-0--discovery)
  - [4.1. Discovery Request](#41-discovery-request)
  - [4.2. Requirements Response](#42-requirements-response)
- [5. Phase 1 — Request](#5-phase-1--request)
  - [5.1. Request Format](#51-request-format)
  - [5.2. Request Signature](#52-request-signature)
- [6. Phase 2 — Grant](#6-phase-2--grant)
  - [6.1. Policy Evaluation](#61-policy-evaluation)
  - [6.2. Grant Issuance](#62-grant-issuance)
  - [6.3. Capability Token Format](#63-capability-token-format)
- [7. Phase 3 — Access](#7-phase-3--access)
  - [7.1. Token Presentation](#71-token-presentation)
  - [7.2. Verification](#72-verification)
- [8. Scope Model](#8-scope-model)
  - [8.1. Scope Structure](#81-scope-structure)
  - [8.2. Domain-Specific Scopes](#82-domain-specific-scopes)
- [9. Revocation](#9-revocation)
  - [9.1. Revocation as Registry Concern](#91-revocation-as-registry-concern)
  - [9.2. TTL Expiry](#92-ttl-expiry)
- [10. Security Considerations](#10-security-considerations)
  - [10.1. Threat Model](#101-threat-model)
  - [10.2. Scope Escalation](#102-scope-escalation)
  - [10.3. Token Lifetime](#103-token-lifetime)
- [11. Formal Verification](#11-formal-verification)
- [12. Legacy Resource Adaptation](#12-legacy-resource-adaptation)
- [13. Relationship to Other Standards](#13-relationship-to-other-standards)
- [14. References](#14-references)

---

## 1. Introduction

### 1.1. The Visa Model

A visa is a scoped, time-limited permission granted by a sovereign
authority. It is issued after an application process. It is carried by
the holder alongside their passport. It is verified locally by the
border agent against the document's own properties — the issuing
authority is not contacted. The visa grants specific rights (work,
transit, study) for a specific duration. It can be revoked
independently of the passport.

EdGrant is this model, applied to digital permissions for bots,
agents, and services.

The passport is the EdProof credential — long-lived, stable,
identity-only. The visa is the EdGrant capability token — scoped,
temporal, permission-bearing. The border agent is the resource — it
verifies both, applies its own policy, and owns the decision.

### 1.2. Design Principles

**1. Discovery before request.**
The requestor does not pre-know a resource's permission model. It
discovers what the resource accepts by trying. The resource rejects
the first attempt and describes what it needs. The requestor then
constructs an informed request.

**2. The resource owner owns the decision.**
What evidence the resource owner requires — bot owner approval,
risk assessment, scope limits, time-of-day restrictions, or nothing
— is the resource owner's policy. The protocol does not prescribe
approval chains. Consistent with EdProof Layer 4 sovereignty.

**3. Least privilege by design.**
The requestor asks for specific scopes and a specific TTL. The
resource owner may narrow these further. No permanent permissions.
No wildcard grants.

**4. The token is a document, not a session.**
The capability token is a self-contained, signed artifact. It is
verified locally. The resource owner is not contacted at access
time. Consistent with EdProof's offline verification model.

**5. Scopes are domain-specific.**
Each resource defines its own scope vocabulary. The protocol carries
scopes as opaque structured data. There is no central scope registry
and no universal scope language.

**6. Revocation is a registry concern.**
The capability token has a TTL. Revocation before TTL expiry is
handled through registries — the same mechanism EdProof uses for
identity revocation.

### 1.3. Relationship to EdProof

EdGrant requires EdProof identity. Every requestor MUST have an
EdProof credential (Layer 2). The EdProof credential proves identity.
The EdGrant capability token proves permission.

EdProof does not change. EdGrant does not extend EdProof. They are
separate protocols that compose: identity (EdProof) + permission
(EdGrant) = authenticated, authorized access.

```
┌─────────────────────────────────────────────────┐
│  EdProof                                         │
│  "Who are you?"                                  │
│  Long-lived credential. Identity only.           │
│  Ceremony once, verify forever.                  │
└──────────────────────────┬──────────────────────┘
                            │ identity established
┌──────────────────────────▼──────────────────────┐
│  EdGrant                                         │
│  "What may you do right now?"                    │
│  Time-limited capability token. Scoped.          │
│  Request, grant, use, expire.                    │
└──────────────────────────┬──────────────────────┘
                            │ both presented to
┌──────────────────────────▼──────────────────────┐
│  Resource                                        │
│  Verifies identity + permission.                 │
│  Applies own policy. Owns the decision.          │
└─────────────────────────────────────────────────┘
```

### 1.4. Relationship to Existing Work

**UCAN (User Controlled Authorization Networks)**: UCAN uses
public-key delegation chains with attenuation. EdGrant borrows the
attenuation principle (permissions can only narrow, never widen) but
differs in being request-initiated rather than delegator-initiated.
UCAN has no discovery phase and no concept of the requestor
discovering what a resource accepts.

**Biscuit Tokens**: Biscuit provides a public-key-verifiable token
format with embedded Datalog policy and attenuation. EdGrant MAY use
Biscuit as a capability token format. Biscuit is a token format, not
a protocol — it does not define request, discovery, or grant flows.

**GNAP (RFC 9635)**: GNAP defines multi-party grant negotiation with
key-bound tokens. EdGrant borrows the "no pre-registration" principle
(use key proofs, not client IDs). GNAP assumes a central authorization
server; EdGrant is decentralized.

**UMA 2.0**: UMA defines resource-owner-controlled access with
requesting-party initiation. EdGrant borrows the "requesting party
initiates" pattern. UMA is OAuth-based and centralized; EdGrant is
not.

**OAuth 2.0 Rich Authorization Requests (RFC 9396)**: RAR defines
structured JSON authorization details replacing flat scopes. EdGrant's
scope model is influenced by RAR's `type`, `actions`, `locations`
structure.

**Google Macaroons**: Macaroons introduced caveats and attenuation for
bearer tokens. EdGrant's attenuation model follows the same principle.
Macaroons use HMAC (shared secrets); EdGrant uses Ed25519 (public key).

---

## 2. Terminology

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in
BCP 14 [RFC 2119] [RFC 8174].

**Requestor**: An entity requesting access to a resource. Identified
by its EdProof credential. May be a bot, agent, service, or human.

**Resource Owner**: The entity controlling a resource and deciding
whether to grant access. Applies its own policy. Sovereign.

**Resource**: The system or data the requestor wants to access. May
verify EdGrant tokens natively or via a sidecar.

**Capability Token**: A signed, time-limited, scope-bound document
granting the requestor specific permissions to a specific resource.
Carried by the requestor. Verified locally by the resource.

**Scope**: A domain-specific description of what the requestor may
do. Opaque to the protocol. Interpreted by the resource.

**Discovery**: The process by which the requestor learns what a
resource accepts before constructing a request.

**Grant**: The resource owner's act of issuing a capability token
after evaluating the request against its policy.

---

## 3. Protocol Overview

```
Requestor                                Resource Owner / Resource
  │                                          │
  │── Phase 0: Discovery ──────────────────►│
  │   (EdProof credential, no grant)         │
  │◄──────────── 401 + Requirements ─────────│
  │   (available scopes, max TTL,            │
  │    approval endpoint, grant format)      │
  │                                          │
  │── Phase 1: Request ────────────────────►│
  │   (resource, scopes, TTL, reason,        │
  │    signed with requestor's key)          │
  │                                          │
  │             [policy evaluation]           │
  │                                          │
  │◄──────────── Phase 2: Grant ─────────────│
  │   (capability token, encrypted)          │
  │                                          │
  │── Phase 3: Access ─────────────────────►│
  │   (EdProof credential +                  │
  │    capability token)                     │
  │◄────────────── scoped response ──────────│
```

---

## 4. Phase 0 — Discovery

### 4.1. Discovery Request

The requestor approaches a resource with only its EdProof credential
and no capability token. This is an unauthenticated access attempt
from the permission perspective (though the identity is authenticated
via EdProof).

The resource MUST:

1. Verify the EdProof credential (valid signature, known fingerprint)
2. Reject access (no capability token present)
3. Return a requirements response describing what the requestor can
   request

### 4.2. Requirements Response

The resource returns:

```json
{
  "resource": "gyros://board/project-alpha",
  "available_scopes": ["read", "write:filter", "write:task", "admin"],
  "max_ttl": "4h",
  "approval_endpoint": "https://grants.example.com/request",
  "grant_format": "edgrant-v1",
  "description": "Jira Project Alpha board"
}
```

**`resource`** — the canonical resource identifier.

**`available_scopes`** — the scopes the resource offers. The
requestor MUST choose from this set. Scopes are domain-specific
strings defined by the resource.

**`max_ttl`** — the maximum time-to-live the resource will accept for
a grant. The requestor SHOULD request less than or equal to this value.

**`approval_endpoint`** — where the requestor submits its grant
request. This MAY be the resource itself or a separate grant service.

**`grant_format`** — the capability token format the resource expects.

**`description`** — human-readable description of the resource.
OPTIONAL.

Discovery is OPTIONAL. A requestor that already knows a resource's
requirements MAY skip Phase 0 and proceed directly to Phase 1.

---

## 5. Phase 1 — Request

### 5.1. Request Format

The requestor constructs a grant request:

```json
{
  "requestor_fingerprint": "SHA256:abc1...",
  "resource": "gyros://board/project-alpha",
  "scopes": ["read"],
  "ttl": "1h",
  "reason": "Fetch task statuses for deploy summary report",
  "requestor_public_key": "ssh-ed25519 AAAAC3...",
  "signature": "AAAAB3NzaC1lZDI1NTE5..."
}
```

**`requestor_fingerprint`** — SHA-256 fingerprint of the requestor's
Ed25519 public key. Matches the EdProof credential.

**`resource`** — the resource identifier from discovery (or known).

**`scopes`** — requested scopes. MUST be a subset of the resource's
available scopes (from discovery). The requestor SHOULD request the
minimum scopes needed.

**`ttl`** — requested time-to-live. MUST NOT exceed the resource's
`max_ttl`. The requestor SHOULD request the minimum TTL needed.

**`reason`** — human-readable reason for the request. RECOMMENDED for
audit purposes.

### 5.2. Request Signature

The requestor signs the request using the SSH `sshsig` wire format
with namespace `edgrant-request`:

```bash
echo -n "${RESOURCE}${SCOPES}${TTL}" > /tmp/req
ssh-keygen -Y sign -f identity -n edgrant-request /tmp/req
```

The namespace `edgrant-request` binds the signature to this protocol
phase. It cannot be replayed as a grant signature or an EdProof
attestation signature.

---

## 6. Phase 2 — Grant

### 6.1. Policy Evaluation

The resource owner evaluates the request against its policy. The
protocol does not prescribe what this policy contains.

A resource owner's policy MAY:

- Accept any request from a requestor with a valid EdProof credential
- Require bot owner co-approval (as additional signed evidence)
- Require human review for certain scopes (e.g. `admin`, `write`)
- Auto-approve low-risk scopes (e.g. `read`) below a TTL threshold
- Reject requests from unknown fingerprints
- Require the requestor to be listed in a registry
- Apply time-of-day, rate, or quota restrictions
- Require any other evidence or condition

The resource owner owns the decision. No external party — not the
requestor, not the protocol — has authority over this decision.

### 6.2. Grant Issuance

If policy is satisfied, the resource owner issues a capability token.
The token is delivered to the requestor encrypted with a TLS session
key (the requestor and resource owner establish a TLS connection for
grant delivery).

### 6.3. Capability Token Format

The capability token is a signed document:

```json
{
  "grant_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "requestor_fingerprint": "SHA256:abc1...",
  "resource": "gyros://board/project-alpha",
  "scopes": ["read"],
  "not_before": "2026-03-28T09:00:00Z",
  "not_after": "2026-03-28T10:00:00Z",
  "issued_at": "2026-03-28T08:59:30Z",
  "resource_owner_fingerprint": "SHA256:ghi3...",
  "resource_owner_signature": "AAAAB3NzaC1lZDI1NTE5..."
}
```

The resource owner signs the token using `sshsig` with namespace
`edgrant-grant`:

```bash
echo -n "${GRANT_PAYLOAD}" > /tmp/grant
ssh-keygen -Y sign -f ro_identity -n edgrant-grant /tmp/grant
```

The token MUST contain:

- **`grant_id`** — unique identifier for this grant (for revocation
  and audit)
- **`requestor_fingerprint`** — binds the token to one requestor
- **`resource`** — the resource this token grants access to
- **`scopes`** — the granted scopes (may be narrower than requested)
- **`not_before`** and **`not_after`** — validity window
- **`resource_owner_fingerprint`** — identifies the granting party
- **`resource_owner_signature`** — the resource owner's Ed25519
  signature over the token payload

The token MAY contain additional fields (e.g. `reason`, `policy_ref`,
`bot_owner_fingerprint`, `bot_owner_signature`) as required by the
resource owner's policy.

EdGrant is token-format-agnostic. Implementations MAY use:

- The JSON format above (simple, widely tooled)
- **Biscuit tokens** (Datalog policy language, attenuation, public-key
  verification)
- **SSH certificates** with custom extensions
- Any signed format that carries the required fields

---

## 7. Phase 3 — Access

### 7.1. Token Presentation

The requestor presents both its EdProof credential and the capability
token to the resource:

```
Authorization: EdProof fingerprint="SHA256:abc1...",
  nonce="...",
  signature="..."
EdGrant: grant_id="a1b2c3d4...",
  token="..."
```

### 7.2. Verification

The resource MUST verify:

1. **EdProof credential** — valid identity (signature, fingerprint)
2. **Capability token signature** — signed by a trusted resource owner
3. **Requestor binding** — the token's `requestor_fingerprint` matches
   the EdProof credential's fingerprint
4. **Time validity** — current time is within `not_before` and
   `not_after`
5. **Scope match** — the requested operation is within the token's
   `scopes`
6. **Revocation check** — the `grant_id` is not in any revocation
   registry the resource consults (OPTIONAL)

If all checks pass, the resource serves the request within the granted
scopes. If any check fails, the resource rejects the request.

The resource owner is not contacted during verification. The token is
verified locally against the resource owner's known public key.

---

## 8. Scope Model

### 8.1. Scope Structure

Scopes are structured as `resource + action + constraints`:

```json
{
  "resource": "grafana://folder/production-alerts",
  "scopes": ["read:dashboard", "write:alert-rule"],
  "constraints": {
    "max_queries_per_minute": 60
  }
}
```

The `constraints` field is OPTIONAL and resource-defined.

### 8.2. Domain-Specific Scopes

Each resource type defines its own scope vocabulary. The protocol
carries scopes as opaque strings. There is no central scope registry.

Examples:

| Resource Type | Scopes |
|---------------|--------|
| Jira board | `read`, `write:filter`, `write:task`, `admin` |
| Grafana folder | `read:dashboard`, `write:alert-rule`, `admin` |
| Component registry | `read`, `publish` |
| CI/CD pipeline | `trigger`, `read:logs`, `admin` |
| Data store | `read`, `write:prefix/<path>` |
| API endpoint | `read:transactions`, `write:orders` |

The resource advertises its available scopes in the discovery response
(Phase 0). The requestor MUST choose from those scopes.

---

## 9. Revocation

### 9.1. Revocation as Registry Concern

Revocation follows the EdProof model: it is a registry concern, not a
token concern.

To revoke a grant before its TTL expires, add the `grant_id` to a
revocation registry. The resource consults this registry at
verification time (Phase 3).

Revocation registry formats follow EdProof registry conventions:

- **Git repository with signed commits** — append-only revocation log
- **JSON document** — list of revoked `grant_id` values with metadata
- **OpenSSH `revoked_keys` format** — if grants are SSH certificates

A resource MAY consult zero revocation registries (rely on TTL alone),
one, or many. Registry unavailability is a policy input, not a
verification failure.

### 9.2. TTL Expiry

Every capability token has a TTL. When the TTL expires, the token is
invalid. No action is required. The requestor must submit a new
request if it needs continued access.

Short TTLs (minutes to hours) are the expected pattern. This reduces
the revocation window and limits exposure from compromised tokens.

**Long TTLs are an anti-pattern in EdGrant** (unlike EdProof
credentials, which SHOULD be long-lived). Permission grants are
inherently temporal — they represent a specific need for a specific
duration.

---

## 10. Security Considerations

### 10.1. Threat Model

The grant exchange (Phases 1-2) operates under the Dolev-Yao
adversary model. The attacker controls the network.

The request (Phase 1) is signed with the requestor's Ed25519 key
(namespace `edgrant-request`). An attacker cannot forge a request
without the private key.

The grant (Phase 2) is delivered encrypted over TLS. An attacker
cannot learn the capability token.

The capability token (Phase 3) is signed by the resource owner
(namespace `edgrant-grant`). An attacker cannot forge a token without
the resource owner's private key.

Namespace separation (`edgrant-request` vs `edgrant-grant` vs
EdProof's `edproof`) prevents cross-protocol and cross-phase signature
replay.

### 10.2. Scope Escalation

Scope escalation is structurally prevented. The request signature
covers the (resource, scopes, TTL) tuple. The grant signature covers
the same fields. If the resource owner narrows the scopes, the token
reflects the narrowed scopes. The requestor cannot present a token
with wider scopes than what was granted.

### 10.3. Token Lifetime

Capability tokens SHOULD have short lifetimes. Recommended ranges:

| Use Case | Recommended TTL |
|----------|----------------|
| One-shot operation (publish, deploy) | 15-30 minutes |
| Batch job (pipeline, ETL) | 1-6 hours |
| Continuous monitoring (dashboards) | 12-24 hours |
| Standing access | Not recommended — use repeated short grants |

---

## 11. Formal Verification

The grant exchange has a machine-checked formal model in the Tamarin
prover at `formal/edgrant.spthy`.

The model operates under the Dolev-Yao adversary and verifies:

| Lemma | Property |
|-------|----------|
| `request_authenticity` | Grant traces back to a genuine requestor request |
| `no_scope_escalation` | Granted scopes match requested scopes |
| `bot_binding` | Grant is bound to exactly one requestor key |
| `no_request_forgery` | Attacker cannot forge a request |
| `grant_secrecy` | Attacker cannot learn the grant token |
| `grant_authorization` | Only resource owner can issue grants |
| `executability` | Protocol can complete successfully |

To verify:

```bash
tamarin-prover --prove formal/edgrant.spthy
```

---

## 12. Legacy Resource Adaptation

Resources that do not support EdGrant natively (e.g. Jira, Grafana)
may be fronted by a **sidecar** — a proxy that implements the EdGrant
protocol and translates grant scopes to the resource's native
authorization model.

The sidecar is the resource owner's implementation concern. EdGrant
does not specify sidecar behavior. The sidecar is the verifier in
EdProof/EdGrant terms — it owns the decision.

Example: a Jira sidecar holds a Jira API token with broad permissions.
When a requestor presents an EdGrant capability token with
`scopes: ["read"]`, the sidecar proxies only read operations to Jira
and rejects writes.

---

## 13. Relationship to Other Standards

**[RFC-EDPROOF]** — EdProof identity protocol. EdGrant requires
EdProof credentials. The two protocols compose but do not depend on
each other's internals.

**[RFC 8032]** — EdDSA (Ed25519). The signature algorithm.

**[SSHSIG]** — OpenSSH signature format. Request and grant signatures
use `sshsig` with protocol-specific namespaces.

**[RFC 9396]** — OAuth 2.0 Rich Authorization Requests. Influenced
the scope model structure.

**[RFC 9635]** — GNAP. Influenced the key-proof and no-pre-registration
design.

**[UCAN]** — User Controlled Authorization Networks. Influenced the
attenuation and delegation model.

**[BISCUIT]** — Biscuit tokens. A RECOMMENDED capability token format
for EdGrant implementations.

---

## 14. References

**[RFC-EDPROOF]** EdProof: A Layered Identity Protocol for Humans and
Machines, v0.1, 2026.

**[RFC 2119]** Bradner, S., "Key words for use in RFCs to Indicate
Requirement Levels", BCP 14, RFC 2119, March 1997.

**[RFC 8032]** Josefsson, S. and I. Liusvaara, "Edwards-Curve Digital
Signature Algorithm (EdDSA)", RFC 8032, January 2017.

**[RFC 8174]** Leiba, B., "Ambiguity of Uppercase vs Lowercase in RFC
2119 Key Words", BCP 14, RFC 8174, May 2017.

**[RFC 9396]** Lodderstedt, T. et al., "OAuth 2.0 Rich Authorization
Requests", RFC 9396, May 2023.

**[RFC 9635]** Richer, J. and F. Imbault, "Grant Negotiation and
Authorization Protocol (GNAP)", RFC 9635, October 2024.

**[SSHSIG]** OpenSSH, "SSHSIG — SSH Signature Format",
https://github.com/openssh/openssh-portable/blob/master/PROTOCOL.sshsig

**[UCAN]** UCAN Working Group, "User Controlled Authorization Network
Specification", https://ucan.xyz/specification/

**[BISCUIT]** Music, G. et al., "Biscuit: Authorization Token Format",
https://www.clever.cloud/blog/engineering/2021/04/12/introduction-to-biscuit/

**[MACAROONS]** Birgisson, A. et al., "Macaroons: Cookies with
Contextual Caveats for Decentralized Authorization in the Cloud",
NDSS 2014.

---

*End of document.*
