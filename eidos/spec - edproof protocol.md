---
tldr: Layered identity protocol — ceremony once, verify forever, verifier owns the decision
category: core
---

# EdProof Protocol

## Target

A digital identity protocol that separates proof, credential, registry, policy, and backing into five independent layers. Built on Ed25519. Formally verified. Scale-neutral.

## Behaviour

### The Passport Model

- An entity generates an Ed25519 key pair; the SHA-256 fingerprint of the public key is its stable identifier
- A one-time attestation ceremony proves key possession and results in a long-lived credential
- The credential is carried by the entity and verified locally — the issuer is never contacted after issuance
- A dormant credential (unused for years) is as valid as one used yesterday
- The same protocol works for a country, a football club, or a single autonomous agent

### Layer 1 — Proof (Attestation Exchange)

- Four-message challenge-response: nonce request → challenge → signed response → credential
- The prover signs `<nonce, 'edproof'>` with its Ed25519 private key
- The `edproof` namespace in the signature prevents cross-protocol replay
- Nonces are single-use and issuer-generated
- HTTP transport uses the `EdProof` authorization scheme with `Replay-Nonce` header (ACME/DPoP pattern)
- Maps directly onto the SPIRE node attestor plugin interface

### Layer 2 — Credential

- Format-agnostic: SPIFFE X.509 SVID, SSH Certificate, or custom signed JSON
- Carries key identity only — no name, role, or nature claims
- Long-lived: minimum one year recommended, ten years for stable entities
- Short credential lifetimes are an anti-pattern — they reintroduce the online dependency the protocol eliminates
- Revocation is never implemented via credential expiry

### Layer 3 — Registry

- A dataset of key information, not an authority
- Returns information, never a boolean verdict
- Consulting is optional — a verifier may use zero, one, or many registries
- Formats: OpenSSH authorized_keys, JSON, Git repository with signed commits
- Registry unavailability is a policy input, not a credential failure

### Layer 4 — Policy

- The verifier owns the decision — no external party has authority
- Policy is sovereign to each verifier; no verifier's policy governs another's
- The same credential presented to different verifiers may yield different decisions (by design)
- Federation is bilateral, explicit, and revocable — no global root of trust
- A credential from an unknown trust domain is unknown, not untrusted

### Layer 5 — Backing

- Orthogonal to all layers above — the protocol does not observe it
- Ranges from file on disk to YubiKey to TPM to threshold schemes
- A verifier may care about backing as a policy input, but this is Layer 4 concern

## Design

### Layer Separation

Each layer interacts only through well-defined unidirectional interfaces:
- Backing → Proof: a signing oracle
- Proof → Credential: attestation result authorizing issuance
- Credential → Registry/Policy: credential presented for evaluation
- Registry → Policy: information (never a decision)

Any layer can be replaced independently without changing adjacent layers.

### Anti-Patterns Rejected

- Short-lived tokens that force the issuer into every access decision (OAuth refresh cycles, SVID default TTLs)
- Registries that return allow/deny instead of data
- Central authorities that verifiers must obey
- Revocation via credential expiry instead of registry + policy
- Conflating identity with permission

### Ceremony Architecture

- Attestation is a point-in-time event, not a running service
- The ceremony infrastructure does not need high availability — only availability at entity birth
- Implementations range from permanent SPIRE servers to one-shot CLI tools to CI steps

### Formal Verification

Seven security lemmas machine-checked under Dolev-Yao adversary in Tamarin prover:
- Injective authentication, nonce single-use, nonce freshness
- Credential secrecy (requires TLS model), entity binding, receiver agreement
- Executability (non-vacuity)

Key modeling choices:
- `fingerprint/1` as injective uninterpreted function (SHA-256 collision resistance)
- `NonceStore` as linear fact (structural single-use enforcement)
- TLS as fresh symmetric key in state, never on wire

## Interactions

- RFC 8032 (Ed25519) — the signature algorithm
- RFC 8555 (ACME) — `Replay-Nonce` pattern for nonce delivery
- RFC 9449 (DPoP) — error-then-retry nonce flow
- SSHSIG — OpenSSH signature format with namespace binding
- SPIFFE/SPIRE — optional integration, attestation maps to node attestor plugin
- W3C DID — SPIFFE ID is structurally equivalent to `did:key`

## Mapping

> [[RFC-EDPROOF.md]]
> [[README.md]]
> [[formal/edproof.spthy]]
