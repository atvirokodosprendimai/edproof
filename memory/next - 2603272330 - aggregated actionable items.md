# Next — Aggregated Actionable Items

**Date:** 2026-03-27
**Branch:** task/eidos-init (11 files, +1682 lines, not yet merged)

## Session Deliverables (completed)

- eidos/ bootstrapped with 4 specs + seed
- Permission grant use cases (13 cases, 5 domains, discovery-first flow)
- Prior art research (7 systems analyzed)
- Tamarin formal model for EdGrant (7 lemmas)

## Actionable Items

### 1 — Merge branch `task/eidos-init` to main
This session's work is on a task branch. 11 commits, all clean.

### 2 — Verify Tamarin model
`formal/edgrant.spthy` was written without Tamarin installed locally. Needs verification:
```bash
tamarin-prover --prove formal/edgrant.spthy
```
Expect 7 lemmas to verify. May need minor syntax fixes.

### 3 — Draft RFC-EDGRANT
The use cases doc and prior art are ready. Next step: write the actual RFC document.
- Use RFC-EDPROOF.md as the template/style reference
- Core protocol: discovery → request → grant → access (2 entities)
- Bot owner approval as policy, not protocol
- Scope format (borrow from OAuth RAR RFC 9396)
- Grant token format (evaluate Biscuit vs simpler dual-signed JSON)
- Revocation model (registry-based, consistent with EdProof)
- Reference prior art (UCAN, Biscuit, GNAP, UMA)

### 4 — Update permission grant use cases for simplified model
Use cases still reference "dual approval" and "bot owner signs" in several places. Some should be updated to reflect that bot owner approval is resource owner's policy, not protocol.

### 5 — Extend Tamarin model: revocation
Current model covers grant issuance but not revocation. A revocation model would prove:
- Revoked grants are rejected
- Only authorized parties can revoke
- Revocation is irreversible (or versioned)

### 6 — Update README.md
README doesn't mention EdGrant yet. After RFC is drafted, add a section.

## Priority Order

```
1 - Verify Tamarin model (formal/edgrant.spthy)
2 - Update use cases for simplified approval model
3 - Draft RFC-EDGRANT
4 - Extend Tamarin model: revocation
5 - Merge branch to main
6 - Update README.md
```
