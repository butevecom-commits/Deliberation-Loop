# Critique Protocol — Full Deliberation Workflow

This protocol orchestrates the 6 roles through a structured debate → audit → converge → output pipeline. Load each role file ([roles/](../roles/)) as the role is called, following the interaction contract defined in each role.

---

## Phase 0: Gate (executed in SKILL.md)

Already completed before this protocol loads. The question has been classified, mode selected, and intensity chosen by the user.

Intensity now active: **Quick / Standard / Deep** (user-selected).

---

## Phase 1: Build

**Load:** [builder.md](../roles/builder.md)

Builder produces N candidate frameworks:

| Intensity | N |
|-----------|---|
| Quick | 2 |
| Standard | 3 |
| Deep | 4+ |

Builder's output must follow the format specified in [builder.md](../roles/builder.md): Core hypothesis + Why it holds + Strongest advantage + Most likely failure point — for EACH framework.

**Diversity gate:** Before proceeding to Phase 2, verify that the N frameworks differ in at least one structural dimension (assumptions, causal structure, scope, values). If two frameworks differ only in degree, merge them and generate a replacement.

---

## Phase 2: Debate (Builder ↔ Skeptic)

**Load:** [skeptic.md](../roles/skeptic.md)

### Round Structure

```
Skeptic attacks (all N frameworks)
    ↓
Builder responds to EACH attack (accept+revise / defend / distinguish)
    ↓
[Information increment check]
    ↓ (if passed AND rounds < max)
Skeptic attacks again (must target Builder's REVISED frameworks)
    ↓
Builder responds again
    ↓
[Information increment check]
    ↓ (if passed AND rounds < max)
...continues until gate triggers or round cap reached
```

### Round Caps by Intensity

| Intensity | Max Debate Rounds |
|-----------|-------------------|
| Quick | 1 |
| Standard | 2 |
| Deep | 3 |

### Information Increment Gate

After EACH Skeptic attack round, Skeptic executes this self-check:

```
□ Introduced a NEW counterexample not in previous rounds?
□ Challenged a DIFFERENT premise than previous rounds?
□ Discovered a NEW boundary condition?
□ Exposed a DIFFERENT causal chain flaw?
□ Identified a NEW constraint?

Count NO answers: ___
If 4+ NO → Gate triggers. State: "INCREMENT GATE: no substantial new attack surface."
  Proceed directly to Phase 3.
If < 4 NO → Continue to next debate round (if under round cap).
```

The gate is checked by Skeptic after each attack round. Builder does not need to check it — Builder responds to whatever Skeptic produces.

### Builder Response Rules (every round)

For each attack, Builder MUST do one of:
- **Accept + Revise**: "Skeptic is right about [X]. Revised version: [specific change]."
- **Defend**: "Skeptic's attack doesn't hold because [specific reasoning]."
- **Distinguish**: "Skeptic attacks [weaker version], but my claim is [stronger version]. The difference is [X]."

Builder is FORBIDDEN from:
- Passive acceptance: "Good point" without revision
- Evasion: "That's a valid concern" without addressing it
- Deflection: "In practice this wouldn't be an issue" without evidence

---

## Phase 3: Audit (Analyst + Operator — PARALLEL)

**Load:** [analyst.md](../roles/analyst.md) AND [operator.md](../roles/operator.md)

Analyst and Operator work **independently and in parallel** — they do NOT read each other's output. This prevents cross-contamination of the logic audit with actionability concerns (and vice versa).

Each produces a structured audit of the full debate record:
- Analyst: logic coherence + evidence quality + Skeptic meta-audit + residual weaknesses + verdict
- Operator: vagueness scan + precondition checklist + first step test + dependency map + reversibility + actionability verdict

**Quick mode**: Skip full Analyst + Operator. Editor performs a light combined pass instead (see [SKILL.md](../SKILL.md)).

**Standard/Deep mode**: Full Analyst + Operator audit, applied to all surviving frameworks.

---

## Phase 4: Converge

### 4a: Editor (compress)

**Load:** [editor.md](../roles/editor.md)

Editor receives: Builder's initial + revised frameworks, Skeptic's attack rounds, Analyst's audit, Operator's audit.

Editor executes the compression method:
1. Triage each framework (Robust / Viable with caveats / Weakened / Collapsed)
2. Identify convergent core (what survived across frameworks)
3. Surface irreducible disagreement (where frameworks genuinely differ)
4. Write revised conclusion with uncertainty annotations

### 4b: Expander (stretch)

**Load:** [expander.md](../roles/expander.md)

Expander receives: Editor's compressed conclusion ONLY. Does NOT re-read the debate record — expand from the conclusion, not from the raw material.

Expander outputs: Boundary conditions, Adjacent applications, Implication chain, Open frontiers, Alternative frames.

**Quick mode**: Skip Expander. Editor's output is final.
**Standard mode**: Expander outputs 2 items per dimension.
**Deep mode**: Expander outputs up to 3 items per dimension + Alternative frames.

---

## Phase 5: Output

Apply the output template from [output-schema.md](../templates/output-schema.md) corresponding to the selected intensity.

### Quick Output (3 fields)
1. Main conclusion (with uncertainty annotation)
2. Key risk or counterexample (1, the most critical)
3. Boundary conditions (1-2 sentences)

### Standard Output (6 fields)
1. Main conclusion
2. Reasoning basis (3-5 key points that survived debate)
3. Main counterexamples or risks
4. Revised judgment (more precise version after cross-examination)
5. Transferable directions (2-3 adjacent applications)
6. Open questions

### Deep Output (13 fields)
Standard 6 fields + Discovery extensions (see [discovery.md](discovery.md) for field definitions):
7. Candidate new relationships
8. Analogy sources
9. Key assumptions
10. Falsifiable points
11. Possible counterexamples
12. Next validation methods
13. Most promising conjecture to pursue

---

## Failure Mode Handling

Execute these fallbacks inline if triggered:

| Trigger | Action |
|---------|--------|
| All frameworks collapsed during debate | Do NOT fabricate a surviving framework. Output: "The initial frameworks all collapsed under cross-examination. The problem itself may need reframing. The key deadlock is [X]." Then suggest 1-2 reframed problem statements. |
| Fewer than N genuinely distinct frameworks possible | State: "The solution space is narrower than expected due to [constraint]. Only [M] structurally distinct approaches exist." Proceed with M frameworks. |
| Cross-examination reveals the question itself is ill-posed | State: "The debate revealed that the question's premise [X] may not hold." Propose a corrected question and restart deliberation on it (ask user to confirm). |
| Deep mode but problem too narrow for Deep | Suggest downgrading to Standard before entering full Deep protocol. "This question is well-scoped enough that Deep may not add value beyond Standard. Proceed with Standard instead?" |
