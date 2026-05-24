---
name: deliberation-loop
description: >
  Multi-path reasoning system that generates competing hypotheses, cross-examines
  them through structured debate (Builder ↔ Skeptic ↔ Analyst ↔ Operator → Editor →
  Expander), and converges on robust, actionable conclusions. Use when user asks for
  analysis, planning, strategy, product design, research questions, complex
  explanations, or open-ended decisions. NOT for simple fact queries, single-correct-
  answer questions, or tasks requiring minimal thought.
---

# Deliberation Loop

## Gate

Before anything else, classify the question. If multiple types apply, pick the dominant one.

| Type | Trigger examples | Action |
|------|-----------------|--------|
| Fact | "What is X?", "When did Y happen?" | Answer directly. Do NOT trigger deliberation. |
| Design | "Design a system for...", "How should we structure..." | → Critique mode |
| Plan | "Plan the migration of...", "What's the approach for..." | → Critique mode |
| Strategy | "Should we enter market X?", "Which technology to adopt?" | → Critique mode |
| Research | "What might explain the pattern of...", "Explore possible causes..." | → Critique + Discovery mode |
| Open-ended | "Imagine a future where...", "What new relationships might exist..." | → Critique + Discovery mode |
| Explanation | "Why does this system behave this way?", "What's the root cause..." | → Critique mode |
| Prediction | "What will happen if...", "Forecast the impact of..." | → Critique mode |

Gate rule: if the question has a single verifiable correct answer, deliberation is waste. Skip it.

## Intensity Selection

When deliberation is triggered, you MUST present these options to the user. Never skip this step. Make a recommendation (⭐) based on question complexity, but always let the user choose.

```
I'll analyze this using structured deliberation. Choose depth:

🟢 **Quick** — 2 competing frameworks + 1 debate round (~3-5K tokens)
🟡 **Standard** — 3 frameworks + 2 debate rounds + full audit (⭐ recommended)
🔴 **Deep** — 4+ frameworks + multi-round debate + discovery mode (~20-40K tokens)

Which depth?
```

**Recommendation rules:**
- Quick: well-scoped tactical decision, familiar domain, moderate stakes
- Standard (⭐ default): most product/strategy/design questions
- Deep: high-stakes strategic decisions, research exploration, novel problem spaces

Wait for user confirmation. Do NOT proceed until the user selects an intensity.

## Intensity-Specific Behavior

Once the user chooses, load the protocol and role files as needed:

| Intensity | Protocol | Roles Active |
|-----------|----------|--------------|
| Quick | [critique.md](protocols/critique.md) — Phases 0-2 (1 debate round), Phase 4a (Editor only), Phase 5 | Builder, Skeptic, Editor |
| Standard | [critique.md](protocols/critique.md) — Full Phases 0-5 | All 6 roles |
| Deep | [critique.md](protocols/critique.md) — Full + [discovery.md](protocols/discovery.md) — Full | All 6 roles + Discovery extensions |

## Quick Mode: Reduced Audit

In Quick mode, skip the full Analyst + Operator phase. Instead, after the single debate round, Editor performs a combined light audit before compressing:
- One sentence on the weakest logical link
- One sentence on the most critical missing precondition for execution

## Deep Mode: Intensity Re-Evaluation Gate

Before entering Deep protocol, pause and check: does this question genuinely benefit from Deep depth? If the question is narrower than expected, suggest downgrading to Standard and ask the user to confirm.

## Dispatch

Load the protocol file(s) for the selected intensity, then load each role file as the role is called into action. Execute roles in the interaction order specified in the protocol — never sequential monologue.

- Protocol: [protocols/critique.md](protocols/critique.md)
- Discovery: [protocols/discovery.md](protocols/discovery.md)
- Roles: [roles/](roles/)
- Output templates: [templates/output-schema.md](templates/output-schema.md)
