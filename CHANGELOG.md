# Changelog

## [1.4.0] — 2026-06-21

### Added
- **Configuration system** — 5 configurable parameters (rigor, evidence, audit, checks, confidenceFloor) with safe defaults. All-default config behaves identically to v1.3.0.
- **Self-Audit Output** — when `audit: Full`, Analysis Metadata table appended to Conclusion documenting loop count, hypotheses, claim confidence distribution, check results, and evidence scope.
- **Evidence scoping** — `evidence: Restricted` / Default / Extended controls which sources the agent may consult.
- **Checks profile** — `checks: Minimal` (fallacy only) / Standard (fallacy + bias) / Full (+ assumption audit).
- **Confidence floor** — `confidenceFloor: HighOnly` requires High confidence for every recommendation; `Relaxed` allows Low for Type 2.
- New Quick Reference rows for config, audit, evidence scoping.

### Changed
- Calibrating Rigor now references config block (`config: rigor: Type 1`).
- Conclusion stage documentation now mentions metadata table for Full audit.
- Theoretical Foundations extended with self-audit / Tetlock calibration tracking row.
- Version bumped to 1.4.0.

## [1.3.0] — 2026-06-21

### Added
- **Formal Fallacy Check** — 7 structural argumentation fallacies (affirming the consequent, denying the antecedent, false dilemma, circular reasoning, straw man, appeal to authority, correlation ≠ causation). Parallel to Cognitive Bias Check. Triggered during Verification; skipping it is a Red Flag. Integrated into Type 1 rigor requirements.

### Fixed
- **Theoretical Foundations** — All 15 source mappings now include DOI and specific section references. Previously 0 DOIs; now 12 DOIs. Feyerabend's position corrected from "methodological pluralism" to "epistemological anarchism". Kahneman primary source added (Tversky & Kahneman, 1974, *Science*). Sosa works specified by title and year.

### Changed
- Version bumped to 1.3.0.
- Stage 5 (Verification) Do list updated to include fallacy check.
- Type 1 decisions now require both bias check and fallacy check.
- Quick Reference, Common Mistakes, and Red Flags updated.
- README descriptions updated across both copies.

## [1.2.1] — 2026-06-18

### Changed
- Renamed test.md to sample_output_with_notes.md for clarity.

## [1.2.0] — 2026-06-14

### Added
- Assumption Audit: systematic pre-loop assumption identification, risk classification, and testability marking.
- Red Flags: 12 conditions that cancel the loop and restart from Observation.
- Rationalization Table: 10 common excuses with counter-arguments.
- Combining Severity and Confidence decoder table.
- Assumption Audit, Red Flags, and Rationalization Table now in README and Quick Reference.

### Changed
- README refreshed with Security section, independent audit results (Gen Agent Trust Hub, Snyk), and TomeVault install/badge integration.
- Verifiability boosted: all load-bearing Evidence claims now require per-claim confidence annotation.

## [1.1.0] — 2026-06-11

### Added
- Theoretical Foundations section: 15 skill components mapped to academic sources (Popper, Feyerabend, Toulmin, Kahneman, etc.).
- CVSS-style severity: urgency (4 levels) × direction (2 levels).
- Domain Rule Integration with formal rules file format.
- per-claim confidence annotation requirement in Evidence stage.
- Calibrating Rigor by decision reversibility (Type 1 / Type 2 / Trivial).

### Changed
- Confidence system expanded to two dimensions (breadth × quality) with weakest-link rule.

## [1.0.0] — 2026-06-08

### Added
- Initial release: 5-stage reasoning loop (Observation → Hypothesis → Evidence → Conclusion → Verification).
- Cognitive Bias Check (5 biases).
- Stage markers and three implementation options.
- Worked examples (database choice, blog post review).
