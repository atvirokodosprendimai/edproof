# Next — Post RFC-EDGRANT Draft

**Date:** 2026-03-28
**Branch:** task/eidos-init (15 commits, not yet merged)

## Completed This Session

- Eidos bootstrapped (specs, memory, config)
- 4 specs pulled from existing codebase
- Permission grant brainstorm + prior art research (7 systems)
- Tamarin formal model (edgrant.spthy, 7 lemmas, unverified — needs Tamarin install)
- RFC-EDGRANT.md v0.1 drafted
- Use cases updated for simplified approval model

## Actionable Items

### 1 — Verify Tamarin model
`formal/edgrant.spthy` needs `tamarin-prover --prove`. Brew install failed (stack dependency). Needs manual Tamarin setup or a machine that has it.

### 2 — Merge branch to main
15 commits on `task/eidos-init`. All clean. Ready to merge.

### 3 — Review RFC-EDGRANT for gaps
First draft. Known areas to review:
- Discovery response format — is JSON the right choice? Should it support content negotiation?
- Capability token format — JSON example given, Biscuit recommended. Should the RFC mandate one?
- Revocation section is thin — could use more detail on registry format and timing guarantees
- No appendix yet for specific resource profiles (like EdProof's Coroot appendix)

### 4 — Write resource-specific profiles
Like EdProof's Appendix B (Coroot), EdGrant could have appendices for:
- Jira profile (sidecar, scope mapping to Jira API)
- Grafana profile (sidecar, folder/team mapping)
- Generic API gateway profile

### 5 — Update README.md
Add EdGrant section: link to RFC, link to Tamarin model, quick start for requesting a grant.

### 6 — Write a story for EdGrant
Like STORY.md stress-tests EdProof, a narrative could stress-test the permission grant model.

## Priority Order

```
1 - Verify Tamarin model (blocked: needs tamarin-prover)
2 - Merge branch to main
3 - Review RFC-EDGRANT for gaps
4 - Write resource-specific profiles
5 - Update README.md
6 - Write an EdGrant story
```
