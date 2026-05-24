# Output Schemas by Intensity

Apply the schema matching the user-selected intensity. Do NOT mix fields across intensities — a Quick output should not include "Open questions" just because the model generated one.

---

## Quick Output

```
## Conclusion
[One sentence. The most precise, honest statement the deliberation produced.
Use uncertainty annotation: [Established] / [Probable] / [Speculative].]

## Key Risk
[One specific counterexample, vulnerability, or boundary violation.
Must reference something Skeptic raised that Builder did NOT fully resolve.]

## Boundary
[1-2 sentences: "This conclusion holds only if [condition]. If [condition] is
violated, [what happens instead]."]
```

---

## Standard Output

```
## 1. Main Conclusion
[One sentence — the most robust judgment after debate + audit.
Must include the dominant uncertainty annotation.]

## 2. Reasoning Basis
[3-5 bullet points. Each must be a claim that SURVIVED cross-examination.
Do not include claims that Skeptic successfully weakened.
Each point tagged: [Established] / [Probable] / [Speculative].]

## 3. Main Counterexamples or Risks
[2-4 items. The strongest attacks that Builder could not fully resolve.
For each: state the risk, why it matters, and what would mitigate it.]

## 4. Revised Judgment
[More precise version of the conclusion after incorporating debate results.
"While the initial assessment was [X], cross-examination showed [Y],
leading to this refined judgment: [Z]."
Explicitly state what changed from the initial conclusion and why.]

## 5. Transferable Directions
[2-3 adjacent domains or problems where this reasoning applies.
For each: "The [specific insight] from this analysis applies to [domain]
because [shared structure]. However, [key difference] limits the transfer."]

## 6. Open Questions
[2-4 questions that remain genuinely unresolved.
Each must be specific enough that someone could design an experiment or
investigation to answer it. "What is the best approach?" is not an open
question — it's the original question restated.]
```

---

## Deep Output

Deep output = Standard output (all 6 fields above) + Discovery extensions below:

```
## 7. Candidate New Relationships
[From Discovery D2. Each is a proposed mechanism/connection/structure.
Format: "[Name]: [relationship description] — [CONJECTURE] or [UNFALSIFIABLE]"
2-4 candidates max.]

## 8. Analogy Sources
[For each candidate relationship: where did it come from?
"Hypothesis H1 draws from [source domain], specifically [specific structure].
The mapping is [X in source] → [Y in target]."]

## 9. Key Assumptions
[For each candidate relationship: what must be true for it to hold?
List assumptions explicitly. "H1 assumes: (a) [assumption], (b) [assumption]."]

## 10. Falsifiable Points
[For each candidate relationship: what observation would kill it?
"H1 is falsified if [specific observation]. This is [obtainable / not currently
obtainable]."]

## 11. Possible Counterexamples
[For each candidate relationship: the strongest reason to doubt it.
Can be an empirical counterexample, a logical contradiction, or a simpler
alternative explanation that produces the same pattern.]

## 12. Next Validation Methods
[For each candidate relationship: what's the smallest, cheapest experiment or
investigation that would increase confidence or falsify it?
Rank by: (cost to run) / (information gained).]

## 13. Most Promising Conjecture
[Which of the candidate relationships is most worth pursuing, and why?
"Weight the combination of: novelty × falsifiability × potential impact."
One paragraph max.]
```

---

## Uncertainty Annotation Reference

Use these markers consistently across ALL output schemas:

| Marker | Definition | When to use |
|--------|-----------|-------------|
| **[Established]** | Survived full debate + audit. No unresolved attacks. | Claim that both Skeptic and Analyst could not break. |
| **[Probable]** | Survived but with residual concerns partially addressed. | Builder resolved some but not all of Skeptic's concerns. |
| **[Speculative]** | Logical but no evidence; plausible but unverified. | Discovery hypotheses, untestable claims, extrapolations. |
| **[Contested]** | Skeptic and Builder genuinely disagree; both have merit. | Irreducible disagreement that the user must judge. |
| **[Unknown]** | Important to the conclusion but no party has basis to claim. | Gaps in knowledge that materially affect the recommendation. |

Rule: If a claim would change the user's decision, it MUST have an annotation. Unannotated claims are implicitly [Established] — do not leave important claims unmarked.
