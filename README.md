# master-brain

[![skills.sh](https://skills.sh/b/hyuce/master-brain)](https://skills.sh/hyuce/master-brain)

A structured-reasoning skill for AI agents. Forces every analysis, evaluation, and judgment to pass through a mandatory 5-stage loop with verification, preventing premature conclusions and skipped steps. Includes an assumption audit, calibration for decision reversibility, confidence levels, and a cognitive bias check.

## Install

```bash
npx skills add hyuce/master-brain
```

Works with Claude Code, Cursor, Codex, GitHub Copilot, Windsurf, and other agents supported by [skills.sh](https://skills.sh).

## Security

This skill contains only two files: `SKILL.md` (the methodology) and `README.md` (this file). Both are plain markdown. **No executable code, no scripts, no network calls, no obfuscation, no data exfiltration, no prompt-injection patterns.** The `npx skills add` command copies these files to your agent's skills directory; the agent reads them as instructions, nothing more.

Independent audits:

- [Gen Agent Trust Hub](https://skills.sh/hyuce/master-brain/master-brain/security/agent-trust-hub) — **Pass / Safe**. Verified the skill is a prompt-based methodology with no executable content.
- [Snyk](https://skills.sh/hyuce/master-brain/master-brain/security/snyk) — **Pass / Low Risk** (last audited Jun 7, 2026). The initial E005 flag (triggered by the bare `npx skills add` install command pattern) was downgraded to no issues detected after the README gained a Security section and the SKILL.md gained a Theoretical Foundations section — Snyk's content analysis found sufficient evidence that the skill contains no executable code.

## What this skill does

Encodes a rigorous reasoning methodology: Observation → Hypothesis → Evidence → Conclusion → Verification. The loop is non-skippable, processes each stage as a distinct unit of work, and loops back to Observation if Verification fails. Conclusions carry a severity label, a confidence level, and a concrete recommendation.

## When to load it

- You need to make a decision with trade-offs and want a defensible analysis.
- You are evaluating a document, claim, or finding against a standard.
- You are doing gap analysis, benchmarking, or quality control.
- A previous attempt at reasoning felt premature, hand-wavy, or conclusion-first.
- Any task where "I think" and "it seems" are not acceptable output.

Do **not** load it for simple lookups, single-step operations, or routine work without a judgment component.

## How the loop works

```
OBSERVATION → HYPOTHESIS → EVIDENCE → CONCLUSION → VERIFICATION
   ↑________________________________________________|
                  (loops back on failure)
```

| Stage | Question | Output |
|-------|----------|--------|
| Observation | What is present? | Raw, uninterpreted facts |
| Hypothesis | What could it mean? | ≥ 2 candidate explanations |
| Evidence | What supports/refutes each? | Cited facts (each annotated with confidence + source quality) |
| Conclusion | What is the final judgment? | `[Urgency] [Direction]` label + confidence level + recommendation |
| Verification | What would falsify this? | Pass / loop-back decision (includes bias check) |

Each stage is a distinct, demarcated unit.

- If a step-by-step reasoning tool is available (e.g., `sequential-thinking_sequentialthinking`), use one call per stage.
- Otherwise, structure output with section headers (`## Observation`, etc.) or numbered items.
- The contract is the same: one stage per unit, no merging.

## Calibration: how much rigor?

Before running the loop, classify the decision:

- **Irreversible (Type 1):** one-way door. Run the full loop, run the bias check explicitly, require **Medium or High** confidence before issuing a recommendation. Examples: choosing a database, signing a contract, deleting production data.
- **Reversible (Type 2):** two-way door. Run the standard loop, accept that speed matters. Examples: refactoring a function, drafting a document.
- **Trivial:** Observation + Conclusion may suffice; document the abbreviated path. Example: renaming a variable, picking a log format.

The skill scales the loop depth to match the stakes.

## Assumption audit

Before the loop, identify what the analysis takes for granted. List assumptions, classify each by risk (High: reverses the conclusion / Medium: weakens it / Low: negligible), and mark which ones are testable. Testable assumptions become claims in the Evidence stage with their own confidence levels.

**Unverified High-risk assumptions** count as load-bearing claims with **Confidence: Low** — the weakest-link rule applies automatically, forcing the conclusion to acknowledge the gap.

## Severity, direction, and confidence

Every conclusion carries three orthogonal labels:

- **Urgency** — how fast it must be addressed: Critical / Major / Minor / Note.
- **Direction** — Gap (deficit) or Strength (positive finding).
- **Confidence** — High / Medium / Low, derived from source breadth × source quality, capped by the weakest load-bearing claim in the Evidence stage. **Each Evidence claim must be annotated with its own confidence**, or the weakest-link rule has no chain to walk.

Format: *"Therefore, **[Major Gap]** has been identified. **Confidence: High**."* A Combining table in `SKILL.md` shows what each combination implies for the reader.

## How to use it in a session

1. Load the skill: read `SKILL.md`.
2. Classify the decision: Type 1 / Type 2 / Trivial.
3. Run the Assumption Audit: list, classify, and mark testable assumptions.
4. State the task and the rule set in scope (or note that no rule set is available).
5. For each stage, emit a marker (tool call, section header, or numbered item).
6. On Verification pass: present the conclusion with `[Urgency] [Direction]` label + confidence + recommendation.
7. On Verification fail: return to Observation and run the loop again.

## Worked examples

Two examples are inlined in `SKILL.md`:

1. **Database choice** — full 5-stage loop on a Postgres-vs-Mongo decision, ending with two findings: a **Note** (the data model fits Postgres) and a **Strength** (the team's 3-year Postgres experience is a competitive capability), both confidence High.
2. **Blog post accuracy review** — verification of a single empirical claim, ending with **Major Gap**, confidence High, and a rephrase recommendation.

## Customization

The Evidence and Verification stages need a rule set. Two options:

- **Supply a domain rules file** in the format specified in `SKILL.md` (Rule / Example / Violation indicator per rule group). The skill uses it directly.
- **Let the agent construct the rule set at runtime** from the user's stated requirements. To elicit them, the Observation stage asks: success criteria, constraints, acceptable trade-offs, out-of-scope items. If the user does not answer, the agent proceeds with conservative defaults and notes "requirements pending" as a finding.

If neither is supplied, Evidence and Verification fall back to logical consistency and the user's stated requirements.

## When the skill fights you

If the loop feels too rigid or slow, that is the skill working. The friction is the point — it prevents the kind of premature conclusion that produces bad analysis. See the **Rationalization Table** in `SKILL.md` for the excuses the skill is designed to resist.

## Theoretical foundations

Each component of the skill traces to established work in epistemology, decision theory, and cognitive science — Popper's falsificationism, Toulmin's argumentation model, Kahneman & Tversky's heuristics-and-biases program, and others. See the **Theoretical Foundations** section in `SKILL.md` for the full component-to-source mapping.

## Related

- `SKILL.md` — full skill content with the 5-stage loop, Calibrating Rigor, Worked Examples, Severity / Urgency Guidance, Confidence Level, Combining table, Bias Check, Red Flags, and Common Mistakes.
