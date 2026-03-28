---
tldr: Glass Reef story — fiction that stress-tests the protocol's design claims under crisis
category: narrative
---

# Narrative Identity

## Target

A fictional narrative (STORY.md) that dramatizes the EdProof protocol's design principles through a city infrastructure crisis. Functions as a design validation artifact — each plot point exercises a specific protocol claim.

## Behaviour

The story demonstrates these protocol properties through narrative:

- **Ceremony once, verify forever**: clinics and infrastructure bootstrapped once, continued working through prolonged network outage
- **Offline verification**: local verifiers operated without upstream connectivity; gates, pumps, and rail worked with cached registry state
- **Credential dormancy ≠ invalidation**: an 8-year-dormant survey shell returns with valid credentials; accepted after fresh challenge-response
- **The verifier owns the decision**: "I'm letting it through because I own the decision" — policy sovereignty exercised at the gate
- **Registry as dataset, not authority**: registry returns information; the human (or agent) makes the boolean decision
- **Anti-pattern dramatized**: Vanta Meridian's subscription identity model (short-lived tokens, mandatory refresh) fails catastrophically when the network goes down
- **Federation without hierarchy**: neighborhoods bootstrap independent trust domains; Git registries passed hand-to-hand; no central authority

## Design

- The story is a **design validation tool**, not marketing material
- Every dramatic beat maps to a technical claim in the RFC
- The antagonist (Vanta Meridian) embodies the anti-patterns the protocol rejects: subscription identity, centralized authority, conflating identity with permission
- The crisis scenario (harbor backbone failure) is the protocol's primary design motivation: what happens when the issuer is unreachable?

## Interactions

- Validates claims in: [[spec - edproof protocol]]
- Published at: STORY.md in project root

## Mapping

> [[STORY.md]]
