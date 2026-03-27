# Prior Art — Permission Grant Protocol (RFC-EDGRANT)

**Date:** 2026-03-27

## Summary

Seven prior art systems analyzed for relevance to an EdProof-based permission grant protocol. Ordered from most to least relevant.

---

## 1. UCAN (User Controlled Authorization Networks) — **Most relevant**

**What it is:** Trustless, decentralized capability delegation using public-key cryptography. JWT-based tokens with delegation chains.

**Key ideas:**
- Capabilities are delegated via signed tokens forming a **proof chain** back to the resource owner
- **Attenuation**: each delegation MUST diminish or restate capabilities — you can never escalate
- **Expiry built in**: every UCAN has `exp` (expiry) and `nbf` (not-before) fields
- **Revocation**: any issuer in the chain can revoke downstream. But revocation requires delivery to relevant parties (partition-tolerant design)
- Public-key verifiable — no shared secrets needed
- Designed for local-first, offline-capable systems

**Relevance to EdProof:**
- UCAN's model is the closest match — it uses public-key identity (like EdProof) as the foundation for capability delegation
- The attenuation model (you can only narrow, never widen) maps to least privilege
- UCAN's revocation challenge (delivery problem) is the same challenge EdProof would face
- UCAN doesn't have dual approval — it's single-chain delegation. EdProof's bot owner → resource owner flow is novel

**Gap UCAN doesn't fill:**
- No concept of "request → approval" — UCAN assumes the delegator initiates. EdProof needs the requestor to initiate
- No dual-party approval chain
- No built-in audit trail beyond the proof chain itself

**Sources:** [UCAN Specification](https://ucan.xyz/specification/) · [UCAN Working Group](https://github.com/ucan-wg/spec) · [Getting Started](https://ucan.xyz/guides/getting-started/)

---

## 2. Biscuit Tokens — **Highly relevant**

**What it is:** Bearer tokens with embedded policy language, using public-key cryptography (not HMAC like Macaroons). Created by Clever Cloud.

**Key ideas:**
- **Public-key verifiable** — unlike Macaroons, no shared secret needed. Uses aggregated signatures
- **Datalog-based policy language** embedded in the token — rules define what the bearer can do
- **Attenuation**: anyone holding a Biscuit can add restrictions (caveats) before passing it on
- Caveats can restrict: time, scope, operations, resource paths
- Third-party blocks: external parties can add verifiable conditions

**Relevance to EdProof:**
- Public-key verification aligns with EdProof's Ed25519 model
- The embedded policy language could express scopes + constraints directly in the grant token
- Attenuation model is elegant — the capability token itself gets narrower as it's delegated
- Could be the **format** for EdProof capability tokens

**Gap Biscuit doesn't fill:**
- No request/approval workflow — it's a token format, not a protocol
- No multi-party approval concept
- Revocation is not specified (relies on short TTLs or external mechanisms)

**Sources:** [Biscuit Introduction](https://www.clever.cloud/blog/engineering/2021/04/12/introduction-to-biscuit/) · [Macaroons vs Biscuit comparison](https://medium.com/asecuritysite-when-bob-met-alice/cybersecurity-biscuits-04ddbb4a4915)

---

## 3. GNAP (Grant Negotiation and Authorization Protocol, RFC 9635) — **Highly relevant**

**What it is:** OAuth successor (effectively "OAuth 3"). IETF standard (Oct 2024). Defines how software requests, negotiates, and receives delegated authorization.

**Key ideas:**
- **No pre-registration required** — clients don't need to register with the auth server beforehand (unlike OAuth 2.0 client_id). Uses key proofs instead
- **Multi-party grant negotiation** — multiple roles (client, resource owner, end user, auth server) can interact over time to negotiate a grant
- **Interaction modes** — supports redirect, user code, push notification, and other patterns for getting approval
- **Fine-grained access requests** — structured access descriptions (similar to OAuth RAR but native)
- **Key-bound tokens** — access tokens are bound to cryptographic keys, not just bearer tokens

**Relevance to EdProof:**
- GNAP's "no pre-registration, use key proofs" aligns with EdProof's ceremony-based identity
- Multi-party negotiation is close to dual approval (bot owner + resource owner)
- GNAP's interaction modes could map to how approvals are collected
- Key-bound tokens match EdProof's Ed25519 model

**Gap GNAP doesn't fill:**
- GNAP is authorization-server-centric — it assumes a central AS that issues tokens. EdProof's model is decentralized
- GNAP doesn't separate identity from authorization the way EdProof does
- Heavy protocol — many round trips, complex state machine

**Sources:** [RFC 9635](https://www.rfc-editor.org/rfc/rfc9635.html) · [GNAP overview](https://oauth.net/gnap/) · [TwigBush GNAP engine](https://github.com/TwigBush/TwigBush)

---

## 4. UMA 2.0 (User-Managed Access) — **Relevant for approval model**

**What it is:** OAuth-based protocol where the resource owner controls who gets access to their resources, and under what conditions. Kantara Initiative standard.

**Key ideas:**
- **Resource owner sets policy asynchronously** — doesn't need to be present at access time
- **Requesting party asks for access** — the requesting party initiates, not the resource owner (this matches EdProof's "bot requests" model)
- **Claims-based authorization** — the requesting party presents claims; the auth server evaluates them against the resource owner's policy
- **Permission tickets** — the resource server issues a ticket when access is denied; the client takes the ticket to the auth server to negotiate access

**Relevance to EdProof:**
- UMA's "requesting party initiates, resource owner controls policy" is the exact flow EdProof needs
- Asynchronous approval (resource owner sets policy ahead of time) maps to policy-gated auto-approval
- Permission tickets are analogous to permission requests in the EdProof model

**Gap UMA doesn't fill:**
- UMA is OAuth-based — requires authorization server infrastructure
- No concept of dual approval (bot owner + resource owner) — only resource owner
- Centralized auth server model vs EdProof's decentralized approach

**Sources:** [UMA 2.0 Spec](https://docs.kantarainitiative.org/uma/wg/rec-oauth-uma-grant-2.0.html) · [UMA Wikipedia](https://en.wikipedia.org/wiki/User-Managed_Access) · [UMA 2.0 by Justin Richer](https://justinsecurity.medium.com/uma-2-0-437c293c3283)

---

## 5. OAuth 2.0 Rich Authorization Requests (RFC 9396) — **Relevant for scope modeling**

**What it is:** Extension to OAuth 2.0 that replaces flat scope strings with structured JSON authorization details.

**Key ideas:**
- **Structured authorization details**: `{ type, locations, actions, datatypes, identifier, privileges }` instead of flat scope strings
- Supports fine-grained requests like "transfer 45 EUR to Merchant A" or "read directory A, write file X"
- Compatible with existing OAuth 2.0 — incremental migration path
- Multiple authorization detail objects in a single request

**Relevance to EdProof:**
- The `authorization_details` JSON structure is a good model for how EdProof capability tokens could express scopes
- `type` + `actions` + `locations` maps cleanly to EdProof's `resource` + `scopes` + `constraints`
- Well-specified, IETF-standard format

**Gap RAR doesn't fill:**
- It's an OAuth extension, not a standalone protocol
- No multi-party approval
- No public-key verification (it's bearer tokens)

**Sources:** [RFC 9396](https://datatracker.ietf.org/doc/html/rfc9396) · [RAR overview](https://oauth.net/2/rich-authorization-requests/) · [RAR explained](https://www.authlete.com/kb/oauth-and-openid-connect/authorization-requests/rich-authorization-requests/)

---

## 6. Google Macaroons — **Relevant for token design (but superseded by Biscuit)**

**What it is:** Bearer tokens with chained HMAC caveats for decentralized delegation and attenuation. Google Research, 2014.

**Key ideas:**
- **Caveats** constrain the token: time limits, scope restrictions, third-party conditions
- **Attenuation by chaining**: anyone can add caveats (narrowing the token) without contacting the issuer
- **Third-party caveats**: require a third party to vouch before the token is valid (this is interesting for dual approval)
- Based on HMAC — verification requires the shared secret

**Relevance to EdProof:**
- Third-party caveats are conceptually close to "bot owner must also approve"
- Attenuation model is elegant for least privilege
- But HMAC-based: requires shared secrets, which breaks EdProof's public-key model

**Why Biscuit is preferred:**
- Biscuit solves Macaroons' shared-secret problem with public-key cryptography
- Same attenuation model, but verifiable without knowing the signing key

**Sources:** [Google Research paper](https://research.google/pubs/macaroons-cookies-with-contextual-caveats-for-decentralized-authorization-in-the-cloud/) · [Macaroons PDF](https://research.google.com/pubs/archive/41892.pdf) · [libmacaroons](https://github.com/rescrv/libmacaroons)

---

## 7. ZCAP-LD (Authorization Capabilities for Linked Data) — **Relevant for decentralized model**

**What it is:** W3C CCG specification for object capabilities in linked data systems. Uses Linked Data Proofs for cryptographic signing.

**Key ideas:**
- Capabilities are linked data objects signed with Data Integrity Proofs
- **Delegation chains** with caveats to restrict scope
- Designed for decentralized systems (no central authority)
- Integrates with DID (Decentralized Identifiers) ecosystem

**Relevance to EdProof:**
- Decentralized, no central authority — matches EdProof philosophy
- DID-compatible (EdProof SPIFFE IDs are structurally equivalent to `did:key`)
- Capability delegation with attenuation

**Gap:**
- Linked Data / JSON-LD ecosystem adds complexity
- W3C CCG spec, not widely adopted
- No request/approval workflow

**Sources:** [ZCAP-LD Spec v0.3](https://w3c-ccg.github.io/zcap-spec/) · [GitHub repo](https://github.com/w3c-ccg/zcap-spec)

---

## 8. SPIFFE + OPA / Zanzibar — **Relevant for integration patterns**

**SPIFFE + OPA:** SPIRE provides workload identity (AuthN); OPA provides policy evaluation (AuthZ). Envoy sidecar proxies requests, checks identity via SPIFFE, evaluates policy via OPA. Common zero-trust pattern.

**Zanzibar / OpenFGA / SpiceDB:** Google's relation-based access control. Models authorization as a graph of relationships (user:alice is editor of document:readme). Permission checks traverse the graph. SpiceDB and OpenFGA are open-source implementations.

**Relevance to EdProof:**
- SPIFFE + OPA shows how identity (Layer 1-2) and authorization (Layer 4) compose in practice
- Zanzibar's relationship model could inform how permission grants are stored and queried
- Both are infrastructure-heavy — EdProof should be lighter

**Sources:** [SPIRE + OPA integration](https://spiffe.io/docs/latest/microservices/envoy-opa/readme/) · [SpiceDB](https://github.com/authzed/spicedb) · [OpenFGA](https://openfga.dev/) · [Zanzibar at Authzed](https://authzed.com/docs/spicedb/concepts/zanzibar)

---

## Synthesis — What EdProof Can Learn

### What exists and what's missing

| Concern | Best prior art | What's missing for EdProof |
|---------|---------------|---------------------------|
| **Capability token format** | Biscuit (public-key, attenuable, embedded policy) | Dual-signature model (bot owner + resource owner) |
| **Delegation model** | UCAN (proof chains, attenuation, expiry) | Request-initiated flow (bot asks, doesn't just receive) |
| **Approval workflow** | UMA (requesting party initiates, resource owner sets policy) | Dual approval (UMA only has resource owner) |
| **Grant negotiation** | GNAP (multi-party, key-bound tokens) | Decentralized model (GNAP assumes central AS) |
| **Scope structure** | OAuth RAR (structured JSON authorization details) | Nothing — RAR's format is directly usable |
| **Revocation** | UCAN (issuer revokes downstream) + registry | Partition-tolerant revocation is unsolved everywhere |
| **Audit trail** | None built-in anywhere | Git-signed audit is novel |

### What RFC-EDGRANT could uniquely provide

1. **Dual-signed capability tokens** — no existing system requires two approvers (bot owner + resource owner) to both sign the grant. GNAP negotiates, UMA delegates, but neither requires dual cryptographic signatures on the artifact itself.

2. **Request-initiated, decentralized** — UCAN is decentralized but delegator-initiated. UMA is request-initiated but centralized. EdProof's model is both request-initiated AND decentralized.

3. **Identity-separated** — every other system either bundles identity with authorization (OAuth, GNAP) or doesn't specify identity at all (Macaroons, Biscuit). EdProof has a clean, formally verified identity layer that the grant protocol can build on without reinventing.

4. **Git-native audit** — no existing protocol specifies audit trail format. Signed Git commits as the audit log is novel and fits EdProof's existing registry model.

### Recommended borrowing

| Borrow from | What | Why |
|-------------|------|-----|
| **Biscuit** | Token format with embedded Datalog policy | Public-key verifiable, attenuable, expressive |
| **UCAN** | Proof chain structure, attenuation rules | Decentralized delegation with least privilege |
| **UMA** | "Requesting party initiates" pattern | Matches bot-requests-access flow |
| **OAuth RAR** | `authorization_details` JSON structure | Well-specified scope format, IETF standard |
| **GNAP** | Key-bound tokens, interaction modes for approval | No pre-registration, multi-party negotiation |
