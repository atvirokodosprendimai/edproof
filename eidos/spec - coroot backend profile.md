---
tldr: Coroot observability tenant provisioning as an EdProof application profile
category: profile
---

# Coroot Backend Profile

## Target

A specific application of the EdProof protocol: provisioning Coroot observability tenants via credential-based bootstrap.

## Behaviour

- After successful attestation and credential issuance, an entity presents its credential to a **coroot-bridge** service
- The coroot-bridge validates the credential, extracts the fingerprint, and derives a deterministic project name
- Project name = `hex(HMAC-SHA256(server_secret, fingerprint || service_name)[0:16])` — deterministic, enumeration-resistant, collision-resistant
- The bridge creates or retrieves the Coroot project and returns the API key + OTLP endpoint URLs
- After provisioning, the agent uses its API key directly with Coroot — EdProof infrastructure is not involved in telemetry

## Design

- The coroot-bridge is a ceremony tool, not a runtime service
- Provisioning happens once per agent lifetime (bootstrap event)
- Implementation options: CLI tool, one-shot container, GitHub Action, or persistent service for continuous agent creation
- Supersedes RFC-PROVISIONING.md v0.3 — the provisioning protocol is now Layer 1 of EdProof
- For SPIRE deployments, the two-phase HTTP flow is replaced by SPIRE attestation; coroot-bridge validates SVIDs instead of EdProof authorization headers

## Interactions

- Depends on: [[spec - edproof protocol]]
- Consumes: Layer 1 attestation + Layer 2 credential
- Produces: Coroot project API key + OTLP endpoints

## Mapping

> [[RFC-EDPROOF.md]] (Appendix B)
