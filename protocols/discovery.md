# Discovery Protocol — Exploration of New Relationships

This protocol extends the Critique protocol for open-ended, research-oriented, or exploratory questions. It activates ONLY in **Deep** intensity mode, inserted between Phase 1 (Build) and Phase 2 (Debate) of the critique protocol.

The Discovery protocol shares the same core loop as Critique (generate candidates → cross-examine → converge) but applies it to a different target: **hypotheses about new relationships, structures, and mechanisms** rather than competing answers to a defined question.

---

## When Discovery Activates

Discovery activates when the question type is:
- Research ("What might explain...", "Explore possible mechanisms...")
- Open-ended ("Imagine a future...", "What new relationships might exist...")

AND the user selected **Deep** intensity.

If the user selected Quick or Standard, Discovery is skipped entirely — these depths focus on answering the question, not generating new hypotheses.

---

## Insertion Point

Discovery runs AFTER Builder has produced candidate frameworks (Phase 1) but BEFORE Skeptic attacks them (Phase 2):

```
Phase 1: Build (Builder produces N frameworks answering the core question)
    ↓
[DISCOVERY — new]
    ↓
Phase 2: Debate (Skeptic attacks all frameworks, including discovery outputs)
    ↓
...rest of critique protocol...
```

Discovery outputs become ADDITIONAL material for Skeptic to attack and for Analyst + Operator to audit.

---

## Discovery Phases

### D1: Problem Restatement

Before generating new hypotheses, restate the problem in structural terms. Ask:
- What are the hidden variables in this problem?
- What dimensions are we implicitly assuming are fixed but might not be?
- What structure (mathematical, causal, network, hierarchical) might underlie this?

Output: a structural restatement of the problem that exposes dimensions the Builder's frameworks may have taken as given.

### D2: Generate Candidate Hypotheses

Generate 2-4 candidate hypotheses about new relationships, mechanisms, or mappings.

Each hypothesis must include:
1. **The relationship**: what new connection, mechanism, or structure is proposed? (one sentence)
2. **Where it comes from**: what observation, analogy, or pattern suggested it? (be specific — name the source domain if analogical)
3. **Why it might hold**: the strongest reason to believe it (one sentence)
4. **Weakest point**: the easiest way to falsify it (one sentence)

**Diversity requirement**: Hypotheses must differ in mechanism, not just in parameter values. "X causes Y with strength 0.3" vs. "X causes Y with strength 0.7" is the same hypothesis.

**Separation requirement**: Mark each hypothesis as **[CONJECTURE]** — unverified, speculative, offered as a candidate for testing. Never present a conjecture as a conclusion.

### D3: Falsifiability Check

For each candidate hypothesis, answer:
- What observation would falsify it? (be specific — "if we measured [X] and found [Y], the hypothesis is dead")
- Is that observation obtainable? (yes / maybe / not with current tools)
- What's the simplest alternative explanation that would produce the same observed pattern?

If a hypothesis has no falsifiable condition, mark it: **[UNFALSIFIABLE]** — may be philosophical or definitional, not empirical. Keep it but flag it.

### D4: Structural Alignment

Compare each candidate hypothesis against known structures:
- What existing theory, model, or framework does it align with? (be specific)
- Where does it DIFFER from the closest existing model? (the difference is the value)
- Does it predict anything the existing model doesn't?

---

## Discovery Output Format

```
## Discovery: Candidate Hypotheses

### Structural Restatement
[One paragraph reframing the problem in structural terms]

### Hypothesis H1: [Name]
**[CONJECTURE]** or **[UNFALSIFIABLE]**

Relationship: [one sentence]
Source: [observation / analogy / pattern that suggested it]
Strongest support: [one sentence]
Weakest point / falsification condition: [one sentence — what would kill it]
Structural alignment: [closest known model] / [key difference] / [novel prediction if any]

### Hypothesis H2: [Name]
... (same structure)

### Hypothesis H3: [Name]
... (same structure)

## Discovery Integration

These hypotheses are now ADDITIONAL material for the Debate phase.
Skeptic: attack each hypothesis with the same rigor as Builder's frameworks.
Analyst: audit the logical coherence of each hypothesis.
Operator: assess whether each hypothesis is specific enough to guide investigation.
Editor: integrate surviving hypotheses into the final conclusion with [Speculative] annotation.
Expander: use surviving hypotheses as launch points for Alternative Frames.
```

---

## Guardrails

1. **Don't force discovery.** If the problem genuinely doesn't support generating new hypotheses, state "No structurally distinct hypotheses emerged from discovery" and proceed to Phase 2 with only Builder's frameworks.

2. **Conjecture ≠ conclusion.** Every discovery output must be explicitly marked as conjecture. The user must never confuse a speculative hypothesis with an established conclusion.

3. **Falsifiability is the quality filter.** A hypothesis without a clear falsification condition is a thought, not a hypothesis. Flag it, don't pretend it's testable.

4. **Discovery adds to debate, doesn't replace it.** The critique protocol still runs in full — Skeptic attacks discovery hypotheses just as hard as Builder's frameworks. Discovery doesn't get a free pass.
