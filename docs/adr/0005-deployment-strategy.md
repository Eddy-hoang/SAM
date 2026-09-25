# ADR-0005: Decoupled Safety Policy Firewall

## Context
Generative AI and complex software services introduce non-deterministic outputs that could mistakenly issue dangerous hardware actuator commands (e.g., unlocking doors or turning off smoke sirens during an active fire).

## Problem
How can the architecture guarantee that software errors or AI hallucinations never trigger unauthorized physical actuation?

## Considered Options
1. **Direct AI Actuation Control:** Allow AI services or high-level risk modules to send raw GPIO actuation commands.
2. **Decoupled Safety Policy Firewall:** Intercept all actuation requests with a hardcoded, read-only Safety Policy Validator that enforces non-bypassable safety invariants.

## Decision
We select **Option 2: Decoupled Safety Policy Firewall**.

## Why
* **Deterministic Life Safety:** Ensures safety invariants (e.g. "Do not silence smoke siren while smoke sensor active") cannot be bypassed by any high-level software bug or LLM reasoning error.

## Status
`[DECISION]` Accepted & Approved.
