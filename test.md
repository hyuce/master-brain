# master-brain — Capability Demonstration

> This file exercises every feature of the skill through a single worked scenario.
> Use it to verify the skill is loaded correctly or to study the methodology in action.

---

## Scenario

**A SaaS startup CTO must decide: migrate from AWS to a bare-metal colocation provider to reduce infrastructure costs.**

The current setup runs 45 EC2 instances, RDS ($8,200/mo cloud bill, growing 15%/month). The CTO found a colocation provider offering equivalent hardware for $3,100/mo on a 3-year contract. The board wants a decision in 2 weeks.

This is a one-way door: signing a 3-year colo contract buys fixed capacity; returning to the cloud mid-contract doubles costs and wastes the hardware investment. It cannot be unwound at low cost.

---

## Step 1 — Calibrating Rigor

```
Is the decision reversible at low cost?
├─ No → IRREVERSIBLE (Type 1) — run the FULL loop with high rigor.
│  Reason: 3-year colo contract; mid-contract rollback means
│          paying for unused hardware + returning to cloud pricing.
│  Implication: ≥ 2 hypotheses, explicit bias check, Medium-or-High
│               confidence required before issuing a recommendation.
```

---

## Step 2 — Assumption Audit

Before the loop, identify what the analysis takes for granted.

| # | Assumption | Risk | Testable | Reasoning |
|---|-----------|------|----------|-----------|
| 1 | Colocation uptime will match current AWS uptime (99.95%) | **High** | Yes | Conclusion reverses entirely if colo uptime is significantly worse — lost revenue during outages exceeds cost savings. |
| 2 | The current 3-person engineering team can absorb hardware operations (RAID replacement, firmware patching, physical troubleshooting) without dropping feature velocity | **High** | Partially | Team capacity can be estimated from current workload but cannot be fully proven without a trial period. |
| 3 | AWS pricing will not drop enough in the next 3 years to erase the cost advantage | **Medium** | No | AWS historically drops prices ~5%/year; a 20% drop would narrow the gap but not erase it entirely. |
| 4 | The startup will not need to scale beyond 200 instances within 3 years | **Low** | No | Even if it does, the colo contract still covers a reasonable expansion; over-provisioning risk is low. |

**Assumption #1 and #2 are High-risk.** They will be tested during Evidence. If either cannot be verified, it becomes a load-bearing claim with `Confidence: Low` and the weakest-link rule applies.

---

## Step 3 — The 5-Stage Loop

### Stage 1 — Observation

Raw facts — no interpretation.

- Current cloud spend: $8,200/mo (EC2 $5,100, RDS $1,800, bandwidth $600, other $700)
- Growth rate: 15% month-over-month (traffic growing, not adding services)
- Colo quote: $3,100/mo fixed on a 3-year contract ($111,600 total commitment)
- Colo specs: equivalent compute (48 cores), equivalent storage (48 TB SSD), managed network
- Colocation is 4 hours from the office (same city, different district)
- Team: 3 backend engineers, 1 DevOps SRE, all experienced with Linux but not with bare-metal hardware ops
- Current uptime SLA from AWS: 99.95% (effective 99.98% over the past 18 months per internal monitoring)
- Colo SLA: 99.9% (power and network only — server hardware failures are the startup's responsibility)
- Board deadline: 14 days
- CTO's personal bias to note: the last AWS outage (February 2026, 23-minute API degradation) caused a tense board call; CTO mentioned "leaving AWS" during that call
- No regulatory or compliance constraints (SaaS product, no HIPAA/PCI)

---

### Stage 2 — Hypothesis

Generate ≥ 2 candidate explanations or paths.

| # | Hypothesis | Description |
|---|-----------|-------------|
| **A** | **Stay on AWS, optimize** | Renegotiate reserved instances, rightsize instances, add savings plans. No operational changes. |
| **B** | **Full migration to colocation** | Move all workloads to the colo provider on the 3-year contract. Operate all hardware in-house. |
| **C** | **Hybrid: keep critical services on AWS, migrate batch/stateless workloads** | Reduce the cloud footprint by ~40% (stateless workers, CI runners, staging environments) to colo; keep RDS and customer-facing API on AWS. |
| **D** | **Switch to a smaller cloud provider (e.g., DigitalOcean, Hetzner Cloud)** | Stay in a managed cloud but cut costs by moving to a lower-tier provider with simpler offerings. |

---

### Stage 3 — Evidence

For each hypothesis, list supporting and contradicting evidence. Annotate every load-bearing claim.

#### Hypothesis A — Stay on AWS, optimize

| # | Claim | Support/Contradict | Confidence | Source |
|---|-------|--------------------|------------|--------|
| A1 | Reserved instances can reduce EC2 costs by 30–40% | Supports | **High** | AWS pricing page; current RI discount documented at 34% for 1-year commitment. Directly verifiable. |
| A2 | The startup's feature velocity depends on managed services (RDS backups, auto-scaling, IAM) | Supports | **High** | Engineering velocity metrics from the last 6 months show no DevOps incidents that would have required bare-metal intervention. Team has zero bare-metal ops experience. |
| A3 | Even with optimization, AWS will remain 2x more expensive than colo | Supports B/D over A | **High** | After 34% RI reduction: cloud ≈ $5,400/mo. Colo: $3,100/mo. Cloud is still 1.74x colo. Math: $8,200 × 0.66 = $5,412. |
| A4 | No operational risk from migration | Supports | **Medium** | Staying means zero migration risk; but the current architecture has no special AWS lock-in (standard VMs, standard PostgreSQL). Migration risk is lower than it looks. |

#### Hypothesis B — Full migration to colocation

| # | Claim | Support/Contradict | Confidence | Source |
|---|-------|--------------------|------------|--------|
| B1 | Colo costs ($3,100/mo) are 62% cheaper than current cloud ($8,200/mo) | Supports | **High** | Vendor quote (dated last week) vs. AWS billing console (current month). Math: (8200−3100)/8200 = 62.2%. Directly verifiable. |
| B2 | Colo uptime SLA (99.9%) is lower than AWS (99.95%), and the startup is responsible for hardware failures | Contradicts | **High** | Both SLAs are documented in provider contracts. AWS covers the full stack; colo covers power and network only — a failed disk or PSU is the startup's problem. The gap is 4.4 hours/year of additional potential downtime. |
| B3 | The 3-person engineering team can absorb hardware operations without dropping feature velocity | **Unverified** | — | No data available. The team has zero bare-metal ops experience. The SRE has managed colocated servers at a previous company but was not responsible for hardware replacement. This is a load-bearing claim — if it cannot be verified, it caps the conclusion at `Confidence: Low`. |
| B4 | Mid-contract rollback cost is prohibitive | Supports A/C over B | **High** | 3-year contract has no early-termination clause. If the startup needs to return to the cloud after 12 months, they pay the remaining $74,400 with zero hardware value recovered (colo hardware depreciates rapidly). |
| B5 | The 4-hour drive to the colo adds operational latency for physical troubleshooting | Contradicts | **Medium** | No on-site staff. A failed PSU requires a 4-hour drive (8-hour round trip). The colo offers remote-hands service at $150/hour but only for standard tasks (cable reseating, visual inspection) — not for advanced diagnostics or hardware replacement. |

#### Hypothesis C — Hybrid (partial colo, keep AWS for critical services)

| # | Claim | Support/Contradict | Confidence | Source |
|---|-------|--------------------|------------|--------|
| C1 | Cutting 40% of cloud workload reduces the bill by 40% (±5%) | Supports | **High** | AWS billing dashboard shows per-instance costs. Stateless workers + staging + CI runners account for 38% of EC2 costs. Math: 38% of $5,100 = $1,938 saved. |
| C2 | Keeping RDS and customer-facing API on AWS eliminates the uptime risk of colocation | Supports | **High** | If critical path stays on AWS, colo uptime only affects non-customer-facing workloads. A colo outage would slow batch jobs, not break the product. |
| C3 | Running two locations (AWS + colo) adds architectural complexity | Contradicts | **Medium** | The team will need to manage two network topologies, two monitoring stacks, and two incident-response playbooks. This is real cost in engineering time but not quantified in current data. |
| C4 | The cost advantage is smaller than full migration | Contradicts (but weaker than the ops risk of B) | **High** | Hybrid saves $1,938/mo vs. full migration's $5,100/mo. But the ops risk of the hybrid model is dramatically lower — the team still has AWS for critical services. |

#### Hypothesis D — Switch to a smaller cloud provider

| # | Claim | Support/Contradict | Confidence | Source |
|---|-------|--------------------|------------|--------|
| D1 | Smaller cloud providers (Hetzner, DO) offer similar compute at 50–60% of AWS cost | Supports | **High** | Public pricing pages: Hetzner CCX63 (48 cores, 192 GB) at $810/mo vs. comparable AWS EC2 at ~$2,100/mo. Directly verifiable. |
| D2 | Smaller providers lack managed database services at AWS RDS quality | Contradicts | **High** | DigitalOcean managed PostgreSQL is simpler than RDS but lacks point-in-time recovery, read replicas, and IAM integration that the startup currently uses. Migration would require database work. |
| D3 | Migration effort is lower than colocation | Supports | **Medium** | Moving from one cloud to another cloud preserves the managed-service model. No hardware procurement, no physical ops. Estimated migration: 4–6 weeks vs. 8–12 weeks for colocation. |

---

### Stage 4 — Conclusion

Synthesize from the evidence.

#### Primary finding

**Hypothesis B (full colo migration) is not recommended** because the load-bearing claim B3 (Ops team capacity) cannot be verified at a sufficient confidence level. Claim B3 is `Confidence: Low (unverified)` — it is the weakest link in the chain. Per the weakest-link rule, the entire Hypothesis B conclusion must be capped at `Confidence: Low`. Type 1 decisions must not be issued with Low confidence.

Therefore, **Major Gap** has been identified (the team lacks bare-metal operations capacity to safely execute a full colocation migration without risking feature velocity and uptime). **Confidence: High** (team history documented: zero bare-metal ops incidents, zero hardware troubleshooting experience, SRE's prior colo experience excluded hardware replacement).

#### Secondary finding

**Hypothesis C (Hybrid model)** is the recommended path:

- Migrate stateless workers, CI runners, and staging environments to the colo provider (saves ~$1,938/mo).
- Keep RDS, the customer-facing API, and production database on AWS (preserves the uptime SLA and managed-service reliability).
- Re-evaluate full migration after 12 months of colo operations experience.

Recommendation: should adopt the hybrid model (Hypothesis C); should defer full migration until the team has 12 months of operational data from managing the colo footprint; must not sign a 3-year full-migration colo contract at this stage.

Therefore, **Strength** has been observed (the hybrid model captures 38% of the cost savings with minimal operational risk, and generates data for a future full-migration decision). **Confidence: Medium** (most claims are High-confidence; the architectural complexity claim C3 is Medium-confidence and may materialize in practice).

Therefore, **Note** has been recorded (Hypothesis D, smaller cloud provider, remains viable as a future migration path but introduces database migration complexity without matching the colo cost savings; not recommended at this stage).

---

### Stage 5 — Verification

Test the conclusion adversarially.

#### Test 1 — What would falsify the Hybrid conclusion?

> "What would have to be true for Hypothesis C to be wrong?"

- The colo provider's SLA is not met in practice (99.9% SLA breached in the first 6 months).
- The architectural complexity (claim C3) proves to be higher than estimated, consuming more engineering time than the $1,938/mo saved.
- AWS drops prices enough that the 38% savings evaporate.

**Assessment:** All three falsification conditions are testable within the first 12 months. The hybrid model functions as a **live experiment** — if any condition triggers, the team can decide with real data. **Pass.**

#### Test 2 — What if the data is wrong?

> "What if the team's velocity metrics understate bare-metal readiness?"

Even if the team could learn hardware ops, the 4-hour drive to the colo adds a latency that AWS eliminates. The startup has no on-site staff; remote-hands service at $150/hour does not cover hardware replacement. This structural constraint is independent of team skill.

**Pass** — the geographic constraint remains regardless of skill assumptions.

#### Test 3 — Conflict-of-interest check

The CTO mentioned "leaving AWS" during a board call after an outage. Is the current analysis anchored on that emotional event?

The February 2026 AWS outage is one data point in 18 months of monitoring. The average uptime over the full period (99.98%) is better than the AWS SLA. The outage does not justify a migration — cost data does. The analysis correctly separates the two signals.

**Pass** — the bias check (below) should confirm this.

#### Test 4 — Domain rules

No domain rules file is supplied. Verification falls back to logical consistency and stated requirements.

#### Verification result

**Pass.** The conclusion holds under adversarial testing. The hybrid model's primary vulnerability (architectural complexity) is acknowledged with `Confidence: Medium` — the reader knows where to verify.

---

### Loop-back simulation — What if Verification had failed?

> This section demonstrates the **no-new-evidence exit condition** (added in v1.1.2).

**Hypothetical scenario:** Suppose the Verification had rejected the Hybrid conclusion because "the colo provider's uptime history cannot be independently verified." The agent would return to Observation:

- **Observation (Loop 2):** No new data is available — the colo provider is privately held, publishes no public uptime history, and refuses to share internal monitoring logs.
- **Evidence (Loop 2):** Same claims, same confidence levels. No new evidence gathered.
- **Verification (Loop 2):** "If no new evidence is available, exit with the current confidence level; document the limitation."

**Exit:** The conclusion would stand at the current confidence level with the note: *"Therefore, **Note** has been recorded (conclusion assumes colo uptime matches SLA; this could not be independently verified)."* The reader is warned; the analysis ends cleanly instead of oscillating.

---

## Cognitive Bias Check

Run before issuing the Conclusion and again during Verification.

| Bias | Self-check | Result |
|------|------------|--------|
| **Confirmation bias** | Did I seek evidence against the hybrid model, or only for it? | **Pass** — B3 and C3 are explicitly contradictory evidence. The analysis did not suppress the ops risk or the complexity risk. |
| **Anchoring** | Am I overweighting the $8,200 AWS bill as the baseline? | **Pass** — the analysis compared colo cost to both current AWS ($8,200) and optimized AWS ($5,400). Both baselines were used. |
| **Sunk cost** | Am I defending AWS because the startup is already invested in it? | **Pass** — the analysis recommends migration (Hybrid C), not staying. The recommendation is a change, not a defense of the status quo. |
| **Availability** | Am I overweighting the February 2026 AWS outage? | **Pass** — the 18-month uptime average (99.98%) was used, not the single outage. The CTO's emotional bias was explicitly noted in Observation and checked in Verification. |
| **Overconfidence** | Is the Medium confidence justified, or am I relying on intuition? | **Pass** — the weakest-link rule was applied explicitly. The only Medium-confidence claim (C3, complexity risk) is flagged. No claim exceeded its evidence. |

**All five checks pass.** The analysis is procedurally sound.

---

## Final Output

### Risk classification
**Type 1** (irreversible: 3-year colo contract, no early termination)

### Assumptions
4 assumptions audited; 2 High-risk. Both tested during Evidence. #1 (uptime) verified; #2 (Ops capacity) unverified — became the weakest link for Hypothesis B.

### Findings

1. **Major Gap** — Full colocation migration is not recommended. The team lacks bare-metal operations capacity (claim B3: `Confidence: Low`). Type 1 decision cannot be issued at Low confidence. **Confidence: High.**

2. **Strength** — Hybrid model (partial colo, keep critical AWS services) captures 38% of cost savings ($1,938/mo) with minimal operational risk. Works as a live experiment generating data for a future full-migration decision. **Confidence: Medium** (architectural complexity claim is Medium-confidence).

3. **Note** — Smaller cloud provider (Hetzner/DigitalOcean) is viable for future evaluation but introduces database migration complexity; not recommended now.

### Recommendation
- **Should** adopt the hybrid model: migrate stateless workers, CI runners, and staging to the colo provider.
- **Should** keep RDS, production database, and customer-facing API on AWS.
- **Must** re-evaluate full migration after 12 months of colo operations data.
- **Must not** sign a 3-year full-migration colo contract at this stage.

### Loop count
1 (single loop; Verification passed on first attempt)

### Bias check
5/5 passed

---

*This document was generated as a demonstration of the master-brain skill v1.2.0. Every structural feature of the skill — Calibrating Rigor, Assumption Audit, the full 5-stage loop, per-claim confidence annotation, weakest-link rule, severity and confidence labels, adversarial Verification, the no-new-evidence exit simulation, and the Cognitive Bias Check — is exercised in this single scenario.*
