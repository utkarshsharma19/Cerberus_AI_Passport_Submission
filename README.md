
# Cerberus

**Today AI Passport controls what an AI may *know* about you. Cerberus controls what an AI may *do* for you.**

Human-owned delegation for autonomous agents acting on production systems. Built for the **AI Passport Ideathon** (Egoist Machines), Agents track, Build lane.

## The idea in one screen

Maya is on call for a GPU cluster. She wants an agent to take routine toil off her plate, but she is still the person paged if it gets production wrong. So she authorizes her agent through her **AI Passport**: one scoped, revocable delegation, capped by what her employer already lets her do. When the agent proposes draining a rack, a deterministic gate (the **Portcullis**) derives what the agent may actually do, verifies it against live state, and only then touches a switch. Every decision returns a receipt. She can revoke at any time.

The enterprise keeps security sovereignty. The person keeps delegation sovereignty. The gate derives effective authority as an intersection:

```
effective authority = request ∩ delegation ∩ current entitlement ∩ agent-delegation policy
```

A privilege Maya may exercise by hand is not automatically one she may delegate to a machine, so the enterprise separately marks what is delegatable to agents at all.

## Try it (live demo)

**▶ [Run the demo](https://utkarshsharma19.github.io/Cerberus_AI_Passport_Submission/cerberus_demo.html)**

Walks the full arc in ~90 seconds: set envelope → agent proposes → Passport signs delegation Pass → Portcullis derives effective authority → **DENY** while unsafe → conditions change → fresh Pass → **PASS** → receipt → **REVOKE** → **DENY**. Mock IAM, scheduler, and orchestrator; no production systems are touched.

## What's here

| File | What it is |
|------|------------|
| `cerberus_demo.html` | Working logical-path prototype (the live demo above) |
| `Cerberus_AI_Passport_Submission_Final_document.docx` | Full submission: problem, architecture, threat model, V1, appendices |
| `cerberus_hero.svg` | One-glance flow: Person → Passport → Agent → Portcullis → Production |
| `cerberus_architecture.svg` | Five-plane architecture with the trust bridge and ceiling |
| `cerberus_moneyshot.pdf` | One-page summary mapped to the judging criteria |

## Why it holds up

- **Complements enterprise IAM** (OAuth, SPIFFE, OPA); no new cryptography. The novelty is user-owned delegation, with consent and receipts, above existing authority.
- **Fail-closed, current-state, single-spend**: entitlement, policy, envelope, and revocation are re-checked at execution; the nonce is atomically consumed; unverifiable inputs deny.
- **Clean data boundary**: the Pass carries a minimal claim with opaque references resolved gateway-side, so corporate identity, resource ids, and telemetry stay enterprise-side.

> The interesting primitive here is governance, not cryptography.
