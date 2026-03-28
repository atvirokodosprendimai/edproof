# Pull — EdProof Identity Protocol

**Source files:**
- `README.md` — project overview, quick start, verification results
- `RFC-EDPROOF.md` — full protocol specification v0.1
- `formal/edproof.spthy` — Tamarin prover formal model
- `STORY.md` — narrative fiction illustrating protocol principles

---

## Collected Material

### Core Protocol Architecture

Five cleanly separated layers, each independently replaceable:

| Layer | Concern | Independence |
|-------|---------|-------------|
| 5 — Backing | Physical key protection (file, YubiKey, TPM, threshold) | Protocol does not observe it |
| 1 — Proof | Ed25519 challenge-response attestation | Math only, offline-capable |
| 2 — Credential | Signed document, carried by entity, long-lived | Issuer not contacted at verify time |
| 3 — Registry | Dataset of key information, optional | Not authoritative, verifier chooses |
| 4 — Policy | Verifier's sovereign judgment | No external authority |

Layer interactions are unidirectional through well-defined interfaces:
- L5 → L1: signing oracle
- L1 → L2: attestation result authorizing credential issuance
- L2 → L3/L4: credential presented for evaluation
- L3 → L4: information (not a decision)

### Attestation Exchange (Layer 1)

Four-message challenge-response:
1. Prover → Issuer: nonce request (unauthenticated)
2. Issuer → Prover: nonce (challenge, fresh, single-use)
3. Prover → Issuer: fingerprint + nonce + signature + public key
   - signature = sign(<nonce, 'edproof'>, sk) — namespace binding prevents cross-protocol replay
4. Issuer → Prover: encrypted credential (over TLS)

HTTP transport uses `Authorization: EdProof` scheme with `Replay-Nonce` header pattern from ACME/DPoP.

SPIRE integration maps directly onto node attestor plugin interface.

### Credential Model (Layer 2)

- Format-agnostic: SPIFFE X.509 SVID, SSH Certificate, custom JSON
- Long-lived (recommended minimum 1 year, 10 years for stable entities)
- **Short lifetimes are an anti-pattern** — transforms document into session token
- **Dormancy is not invalidation** — 8-year-old unused credential is valid
- Carries key identity only — no name, role, or nature claims
- Revocation is a L3/L4 concern, never credential expiry

### Registry Model (Layer 3)

- Dataset, not authority — returns information, never a boolean
- Formats: authorized_keys, JSON, Git repo
- Consulting is optional — verifier may use zero, one, or many registries
- Registry unavailability ≠ credential failure

### Policy Sovereignty (Layer 4)

- **The verifier owns the decision** — no external authority
- Policy is sovereign to each verifier
- Federation is bilateral, explicit, revocable — no global root of trust
- Same credential, different verifiers → different decisions (by design)

### Formal Verification

Tamarin model under Dolev-Yao adversary. 7 lemmas, all verified in 0.40s:
- authentication_injective, nonce_single_use, nonce_freshness
- credential_secrecy, entity_binding, receiver_agreement, executability

Key modeling decisions:
- fingerprint/1 as injective uninterpreted function (models SHA-256 collision resistance)
- NonceStore as linear fact (structurally enforces single-use)
- TLS modeled as fresh symmetric key in state (never on wire)
- 'edproof' namespace constant in signature for protocol binding

### Coroot Backend Profile (Appendix B)

Specific application: entity presents credential to coroot-bridge service to receive Coroot project API key. Deterministic project name via HMAC. Bootstrap event, not runtime service.

---

## Patterns

- **Passport model**: issue once, carry forever, verify locally
- **Layer separation**: each concern independently replaceable
- **Ceremony-based**: one-time events, not running services
- **Information vs decision**: registries inform, policy decides
- **Namespace binding**: 'edproof' in signature prevents cross-protocol replay
- **Scale neutrality**: same protocol for a country or a single agent

## Dependencies

- Ed25519 (RFC 8032)
- SSHSIG format (OpenSSH)
- ACME nonce pattern (RFC 8555)
- DPoP error-then-retry flow (RFC 9449)
- SPIFFE/SPIRE (optional integration)
- Tamarin prover (verification tooling)

---

## Intent Sketch

### What someone needs to know to re-implement this differently

**The core insight**: Identity proof, credential, policy information, and access decision are four separate things. Most systems conflate them. Separating them eliminates the online dependency that makes systems fragile.

**Design axioms**:
1. Ceremony once, verify forever — the issuer exits the picture after attestation
2. A credential is information about key identity, not a permission slip
3. Registries are datasets a verifier may consult — they return data, not verdicts
4. The verifier owns the decision — no external party has authority over it
5. Key backing is orthogonal — the protocol is indifferent to how the key is protected
6. Any entity can be both prover and verifier — no mandatory hierarchy

**The anti-pattern being rejected**: Short-lived tokens that force the issuer into every access decision (OAuth refresh, SVID TTLs, subscription identity). This creates a chokepoint: when the issuer is unreachable, the system fails.

**The threat model is honest**: Dolev-Yao (full network control). The protocol is secure even without TLS for the challenge exchange. Credential secrecy requires TLS.

**Federation is explicitly bilateral**: No global root. Unknown trust domain = unknown, not untrusted. The verifier decides.

**Revocation model**: Never via expiry. Always via registry + policy. A valid document rejected by policy is the passport model applied to digital identity.
