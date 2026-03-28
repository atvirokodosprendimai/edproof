# Session — EdProof Eidos Bootstrap & EdGrant Protocol

**Date:** 2026-03-27 → 2026-03-28
**Branch:** task/eidos-init (merged to main)

## What Happened

### EdProof Eidos Bootstrap
- `/eidos:init` — created eidos/, memory/, .eidos-config.yaml
- `/eidos:pull` — extracted 3 specs from existing codebase:
  - spec - edproof protocol.md (core 5-layer architecture)
  - spec - coroot backend profile.md (observability tenant provisioning)
  - spec - narrative identity.md (STORY.md as design validation)

### EdGrant Permission Grant Protocol
- `/eidos:brainstorm` — explored permission grant mechanisms for bots/services
  - User provided detailed Lithuanian spec for the use case
  - Key clarifications: Gyros = Jira, sidecar is adapter not architecture, bot owner approval is policy not protocol
- Prior art research — 7 systems analyzed: UCAN, Biscuit, GNAP (RFC 9635), UMA 2.0, OAuth RAR (RFC 9396), Macaroons, ZCAP-LD
- Use cases — 13 concrete scenarios across Jira, Antark, component registries, Grafana, generic resources
- Discovery-first flow — bot learns requirements by trying (ACME/DPoP pattern)
- Tamarin model — `formal/edgrant.spthy`, 7 lemmas, all verified in 0.66s
- RFC-EDGRANT.md v0.1 — full protocol specification drafted
- **Separated into own repo** — https://github.com/atvirokodosprendimai/edgrant

## Key Design Decisions

1. **EdProof = identity only.** EdGrant is a separate protocol, separate repo
2. **Two protocol entities:** requestor and resource owner. Bot owner is policy, not protocol
3. **Discovery-first:** bot approaches resource with identity only, gets rejected with requirements response
4. **Resource owner owns the decision:** consistent with EdProof Layer 4 sovereignty
5. **Sidecar is resource owner's concern:** for legacy resources (Jira, Grafana), not part of the protocol
6. **Short TTLs for grants** (anti-pattern to EdProof's long-lived credentials)
7. **Token format agnostic:** JSON, Biscuit, SSH certificates — resource owner chooses

## Artifacts Created

### edproof repo (merged to main)
- `.eidos-config.yaml`
- `eidos/seed.md`
- `eidos/spec - edproof protocol.md`
- `eidos/spec - coroot backend profile.md`
- `eidos/spec - narrative identity.md`
- `memory/human.md`
- `memory/pull - 2603271158 - extracted spec from edproof identity protocol.md`

### edgrant repo (new, pushed to GitHub)
- `README.md`
- `RFC-EDGRANT.md`
- `formal/edgrant.spthy` (7/7 lemmas verified)
- `spec - permission grant use cases.md`
- `brainstorm - 2603271651 - edproof permission grants for bots and services.md`
- `research - 2603271704 - prior art for permission grant protocol.md`

## Open Items

- 4 untracked files in edproof/eidos/ (pre-existing, not from this session)
- EdGrant RFC needs review: discovery format, token format mandate, revocation detail
- Resource-specific profiles (Jira, Grafana appendices) not yet written
- EdProof README not yet updated with EdGrant reference
