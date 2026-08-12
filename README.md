# Cerberus

**Today AI Passport controls what an AI may *know* about you. Cerberus shows how that same user-owned context can help control what an AI may *do* for you.**

Human-owned context and delegation for autonomous agents acting on production systems. Built for the **AI Passport Ideathon** by Egoist Machines, Agents track, Build lane.

## The idea in one screen

Maya is on call for a GPU cluster. She wants an agent to remove routine operational toil, but she is still the person paged if the automation gets production wrong.

Her **AI Passport** holds user-controlled operational preferences and delegation boundaries, such as which agent she trusts, which classes of maintenance actions she is comfortable delegating, her maintenance window, and her preferred maximum lease duration.

Cerberus receives scoped access to only the AI Passport context Maya allows.

When the agent proposes an action such as `rack.drain`, a deterministic authorization gate called the **Portcullis** combines Maya's user-controlled delegation context with current enterprise IAM, agent policy, and live operational state before anything reaches production.

The enterprise determines what Maya is authorized to do. Maya determines what she is comfortable allowing an agent to do on her behalf.

The enterprise keeps **security sovereignty**. The person keeps **delegation sovereignty**.

Portcullis derives effective authority as:

```text
effective authority =
request
∩ user-approved delegation context
∩ current enterprise entitlement
∩ enterprise agent-delegation policy
```

A privilege Maya may exercise manually is not automatically one she may delegate to an autonomous agent. Enterprise policy separately defines which privileges are agent-delegatable.

## Try it

**▶ [Run the live demo](https://utkarshsharma19.github.io/Cerberus_AI_Passport_Submission/cerberus_demo.html)**

The prototype walks through the complete flow in about 90 seconds:

```text
Maya owns Operational Delegation context in AI Passport
        ↓
Cerberus receives scoped AI Passport access
        ↓
Agent proposes rack.drain
        ↓
Portcullis evaluates:
    user delegation context
    ∩ current IAM
    ∩ enterprise agent policy
    ∩ live safety state
        ↓
DENY while unsafe
        ↓
conditions change
        ↓
agent retries
        ↓
PASS
        ↓
AI Passport access receipt
+
Portcullis execution receipt
        ↓
Maya revokes Cerberus context access
        ↓
next attempt
        ↓
DENY
```

The demo uses mock IAM, a mock scheduler, and a mock orchestrator. No production systems are touched.

## What AI Passport does

AI Passport remains the **user-owned context layer**.

For Maya, that context can include an `Operational Delegation` category containing information such as:

```text
Trusted agent: Cerberus
Allowed task class: rack maintenance
Maintenance window: 22:00 to 02:00
Maximum lease preference: 30 minutes
Escalate outside policy: yes
```

Cerberus does not automatically receive this information.

It requests a scoped **AI Passport Pass** to the context categories it needs. Maya controls that access and can revoke it at any time.

AI Passport receipts show what context Cerberus accessed and when.

AI Passport does **not** replace enterprise IAM or directly grant production authority.

## What Portcullis does

Portcullis is the deterministic execution boundary between an autonomous agent and production infrastructure.

For every proposed action it evaluates:

```text
User delegation context      ✓
Current enterprise IAM       ✓
Enterprise agent policy      ✓
Authoritative safety state   ✓ / ✕
Live telemetry evidence      ✓ / ✕
```

Only when all required checks pass can the action reach the orchestrator.

If any required authorization or safety input cannot be verified, Portcullis fails closed.

## Example

Maya's AI Passport says she is comfortable allowing Cerberus to perform rack maintenance during her approved maintenance window.

The agent proposes:

```text
drain rack 4
```

Portcullis checks:

```text
AI Passport context          ✓ rack maintenance allowed
Current IAM                  ✓ Maya may drain rack 4
Agent-delegation policy      ✓ rack.drain may be automated
Scheduler state              ✕ rack not safe to drain
sFlow evidence               ✕ 3 active flows
```

Result:

```text
DENY
Rack 4 is not currently safe to drain.
```

Later the scheduler reports the rack safe and active flows reach zero.

The agent retries.

```text
AI Passport context          ✓
Current IAM                  ✓
Agent-delegation policy      ✓
Scheduler state              ✓
sFlow evidence               ✓
```

Result:

```text
PASS
Rack 4 drain authorized.
```

Maya can then revoke Cerberus's AI Passport context access.

The next request fails immediately:

```text
DENY
Cerberus no longer has access to Maya's Operational Delegation context.
```

## Two layers of receipts

Cerberus keeps AI Passport context access separate from enterprise execution authorization.

### AI Passport receipt

Shows what Cerberus learned about Maya.

```text
App: Cerberus
Accessed: Operational Delegation
Purpose: rack maintenance
Time: 22:14:07
```

### Portcullis execution receipt

Shows what the agent attempted to do.

```text
Action: rack.drain
Resource: resource_83af
Decision: PASS
Policy: v6
Time: 22:14:08
```

This gives Maya visibility into both:

1. **what the AI accessed about her**
2. **what the agent attempted to do using that context**

## Architecture

Cerberus separates probabilistic planning from deterministic execution.

```text
          MAYA
            │
            ▼
      AI PASSPORT
  user-owned context
            │
     scoped Context Pass
            │
            ▼
      CERBERUS AGENT
            │
      proposes action
            │
            ▼
       PORTCULLIS
            │
     ┌──────┼────────┐
     ▼      ▼        ▼
 Passport   IAM    Agent Policy
 Context            +
                Live State
     \       |       /
      \      |      /
       └─────┼─────┘
             ▼
         PASS / DENY
             │
             ▼
         PRODUCTION
```

The underlying infrastructure implementation is grounded in SONiC-based orchestration and an sFlow telemetry server exposed to agents through MCP.

Telemetry is treated as **evidence**, not as the authoritative safety oracle. Portcullis combines telemetry with authoritative controller or scheduler state before making an execution decision.

## Why it holds up

* **Uses AI Passport for what it is built for.** Maya owns her context, Cerberus receives scoped access through an AI Passport Pass, access is revocable, and Passport receipts show what context was read and when.

* **Complements enterprise IAM.** OAuth, SPIFFE, OPA, mTLS, and existing IAM remain authoritative for production permissions. Cerberus does not attempt to replace them.

* **Separates authority from delegation.** The enterprise determines what Maya can do. Maya controls what she is comfortable delegating to an agent.

* **Separates human authority from machine-delegatable authority.** A privilege that a person can exercise manually does not automatically become available to an autonomous agent.

* **Evaluates current state.** Portcullis checks current IAM, current enterprise policy, current context-access status, and authoritative safety state when execution is attempted.

* **Fails closed.** If a required input cannot be verified, the action is denied.

* **Protects against duplicate execution.** Execution requests can use atomic request identifiers or nonces to prevent replay and double execution without redefining the AI Passport Pass as a single-use production credential.

* **Maintains a clean data boundary.** Corporate identity, infrastructure identifiers, IAM entitlements, topology, policy state, and raw telemetry remain enterprise-side. AI Passport exposes only the user-controlled context Cerberus has permission to access.

* **Audits both layers.** AI Passport records what Cerberus accessed about Maya. Portcullis records what the agent attempted and whether execution was allowed.

## Why infrastructure

Infrastructure is a useful proving ground because agent actions are consequential, authorization boundaries are concrete, and failures are measurable.

A production action naturally maps to:

```text
action × resource × time × policy × live state
```

That makes user-controlled delegation testable rather than abstract.

The same pattern can later extend to deployment agents, database agents, financial agents, and other systems where AI takes consequential actions on a person's behalf.

## What's included

| File                                                  | What it is                                                                                                  |
| ----------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| `cerberus_demo.html`                                  | Working logical-path browser prototype                                                                      |
| `Cerberus_AI_Passport_Submission_Final_document.docx` | Full submission with problem, architecture, risks, V1, and appendices                                       |
| `cerberus_hero.svg`                                   | One-glance flow from Person to Passport to Agent to Portcullis to Production                                |
| `cerberus_architecture.svg`                           | Detailed architecture showing AI Passport context, enterprise authority, policy, and execution              |
| `cerberus_business_impact.pdf`                        | Business impact view covering safer autonomy, bounded blast radius, reduced approval toil, and auditability |

## What we learned

> **The interesting primitive is governance, not cryptography.**

Existing identity infrastructure already answers:

> What is this person authorized to do?

AI Passport answers:

> What context does this person want an AI application to access about them?

Cerberus connects those ideas to autonomous action:

> Of everything I am authorized to do, what am I comfortable allowing this agent to do for me?

That answer should not be buried independently inside every AI agent.

It should come from context the person can inspect, control, share selectively, and revoke.

## What's next

The next step is to replace the prototype adapters with real enterprise integrations:

* real IAM attestation
* real enterprise agent-delegation policy
* authoritative scheduler state
* production telemetry
* an orchestrator connection protected by mTLS
* an enforced Portcullis-only execution path

From there, the same model can extend beyond infrastructure to any environment where autonomous agents take consequential actions.

> **Today AI Passport controls what an AI may know about you. Cerberus shows how that same user-owned context can help control what an AI may do for you.**
