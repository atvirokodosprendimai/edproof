# EdProof

Layered identity protocol for humans and machines.

Ed25519 public key cryptography. Five cleanly separated layers:
proof, credential, registry, policy, backing. Formally verified.

## Quick Start — Generating a Key

Every entity (human or machine) needs an Ed25519 key pair. Generate one with:

```bash
ssh-keygen -t ed25519 -f identity -C "entity@context"
```

This produces two files:

- `identity` — private key (never share or leave the host)
- `identity.pub` — public key (safe to publish)

The SHA-256 fingerprint of the public key becomes the entity's stable
identifier within EdProof:

```bash
ssh-keygen -l -E sha256 -f identity.pub
```

Once you have a key pair, present the public key to an EdProof issuer to
obtain a credential (see [RFC-EDPROOF.md §4](RFC-EDPROOF.md#4-layer-1--proof)).

## Documents

- [`RFC-EDPROOF.md`](RFC-EDPROOF.md) — full protocol specification (v0.1)
- [`formal/edproof.spthy`](formal/edproof.spthy) — Tamarin prover model

## Formal Verification

All seven security lemmas machine-checked in 0.40s:

```
authentication_injective  verified   injective agreement on nonce
nonce_single_use          verified   nonce consumed at most once
nonce_freshness           verified   only issuer-generated nonces accepted
credential_secrecy        verified   attacker cannot learn the credential
entity_binding            verified   fingerprint uniquely identifies one key
receiver_agreement        verified   prover's credential matches issuer's
executability             verified   protocol can complete (non-vacuity)
```

```bash
tamarin-prover --prove formal/edproof.spthy
```

Requires [Tamarin prover](https://tamarin-prover.com/) ≥ 1.10 and Maude ≥ 3.5.

## Key Properties

**Ceremony once, verify forever.** Attestation happens once at entity
birth. The credential is long-lived. The issuer is not involved in
subsequent verifications.

**Passport model.** Credential is carried by the holder, verified
locally against the document's own signatures. Issuer not contacted.

**Policy sovereignty.** Each verifier applies its own policy
independently. No central authority. Scale-neutral.

**Key backing orthogonal.** File, YubiKey, TPM, threshold — the
protocol does not observe the backing layer.

## What it this about? 

- [STORY](STORY.md)
