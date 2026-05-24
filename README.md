# Deliberation Loop

A Claude Code skill that upgrades the model from a **single-path answer generator** into a **multi-path reasoning and structured debate system**.

## What It Does

Instead of producing the first plausible answer, Deliberation Loop forces the model through:

1. **Multi-candidate generation** — 2-4 competing frameworks with explicit assumptions and failure points
2. **Adversarial debate** — Builder ↔ Skeptic structured cross-examination with mandatory citation and response
3. **Independent audit** — Analyst (logic chain) + Operator (actionability) auditing in parallel, no cross-contamination
4. **Convergence** — Editor compresses surviving claims; Expander radiates to adjacent applications
5. **Discovery mode** (Deep only) — generates novel hypotheses with built-in falsifiability checks

The result: conclusions that are more specific, better bounded, and more actionable than single-pass answers.

## Architecture

```
SKILL.md          → Gate (question classification) + Intensity selection
protocols/        → critique.md (5-phase debate workflow) + discovery.md (hypothesis generation)
roles/            → 6 role prompts: Builder, Skeptic, Analyst, Operator, Editor, Expander
templates/        → Output schemas for Quick / Standard / Deep intensities
examples/         → Worked examples at each intensity level
```

### 6 Roles

| Role | Function | Stance |
|------|----------|--------|
| Builder | Proposes candidate frameworks, responds to attacks, revises | Constructive |
| Skeptic | Attacks vulnerabilities, challenges assumptions, finds counterexamples | Adversarial |
| Analyst | Audits logical coherence and evidence quality of the full debate record | Neutral |
| Operator | Audits actionability: vagueness, preconditions, dependencies, reversibility | Pragmatic |
| Editor | Compresses the deliberation record into the most precise, honest conclusion | Convergent |
| Expander | Radiates from the conclusion: boundaries, adjacent applications, open frontiers | Divergent |

## Intensity Levels

| Level | Frameworks | Debate Rounds | Roles | Output Fields | Est. Tokens |
|-------|-----------|---------------|-------|---------------|-------------|
| Quick | 2 | 1 | Builder, Skeptic, Editor | 3 | ~3-5K |
| Standard | 3 | 2 | All 6 | 6 | ~8-15K |
| Deep | 4+ | Multi + gate | All 6 + Discovery | 13 | ~20-40K |

## When It Triggers

**Triggers on**: Analysis, planning, strategy, product design, research questions, complex explanations, open-ended decisions.

**Does NOT trigger on**: Simple fact queries, single-correct-answer questions, tasks requiring minimal thought.

## Installation

```bash
# Copy to Claude Code skills directory
cp -r deliberation-loop ~/.claude/skills/
```

Restart Claude Code or start a new conversation. The skill auto-loads when a matching question is detected.

## Environment

- **Platform**: Claude Code (CLI, IDE extension, or web)
- **Dependencies**: None — the skill is prompt-only, no scripts or external tools
- **Token cost**: Varies by intensity. Quick (~3-5K), Standard (~8-15K), Deep (~20-40K)

## Key Design Decisions

- **Debate, not monologue**: Builder MUST respond to Skeptic's attacks, not passively accept. This creates genuine adversarial pressure.
- **Information increment gate**: After each debate round, Skeptic self-checks whether new information was produced. If not, the debate converges immediately — no infinite loops.
- **Parallel audit**: Analyst and Operator work independently to prevent cross-contamination of logic and actionability assessments.
- **Uncertainty annotation**: Every claim is tagged [Established] / [Probable] / [Speculative] / [Contested] / [Unknown].

## File Overview

```
deliberation-loop/
├── SKILL.md                    # Entry point: Gate + Intensity + Dispatch
├── README.md                   # This file
├── .gitignore
├── protocols/
│   ├── critique.md             # Main debate protocol (Phase 0-5)
│   └── discovery.md            # Hypothesis generation protocol (D1-D4)
├── roles/
│   ├── builder.md              # Builder role prompt + examples
│   ├── skeptic.md              # Skeptic role prompt + attack patterns
│   ├── analyst.md              # Analyst role prompt + logic audit
│   ├── operator.md             # Operator role prompt + actionability audit
│   ├── editor.md               # Editor role prompt + compression method
│   └── expander.md             # Expander role prompt + expansion dimensions
├── templates/
│   └── output-schema.md        # Output templates for all 3 intensities
└── examples/
    ├── quick-example.md        # PostgreSQL vs MongoDB (Quick)
    ├── standard-example.md     # Auth system redesign (Standard)
    └── deep-example.md         # Remote team velocity gap (Deep + Discovery)
```

## License

MIT
