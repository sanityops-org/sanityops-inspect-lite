---
title: Inspect Cross Checklist (Lite)
description: "Condensed cross-artifact consistency checklist (QD-PS / QD-PT / QD-ST / QD-SS / QD-TT / QD-CYC) for the sanityops-inspect lite inspector — 29 items across six relationship groups, AC-L tiering (7/20/29 applicable items), P0 minimum rule sets (7/9/9), five-state judgment, same-source deduplication, and the scoring note. Extracted from inspect/cross.md v1.0 (2026-10); defines no new rules."
---

# Cross-Artifact Consistency Checklist (Lite Inspector)

> **Provenance**: Condensed from `inspect/cross.md` v1.0 (SanityOps Framework, CC BY-SA 4.0). This file is a compressed extraction for the sanityops-inspect lite inspector and defines NO new rules. On any conflict, `inspect/cross.md` governs.
>
> **Source note (reproduced, not adjudicated)**: the source's version header and §0.3 version history still read `v1.0 / first release`, but the working-tree content has been expanded to **29 items across six relationship groups** (QD-SS and QD-TT added, plus QD-PS-1.7 / QD-PT-2.7 / QD-CYC-1.3) and its terminology was unified from `L1/L2/L3` + `RS/SS` to `AC-L` + `OR-L` — a "version-evidence drift" (content ahead of the version label). This checklist reproduces the **current content** and records the stale version label as-is.

## How to Use

- **Non-blocking mode (lite skill).** Cross runs regardless of upstream single-artifact P0 findings: upstream P0 defects are recorded and carried forward, but do **not** prevent this checklist from executing, and are not a reason to mark the run "NOT EXECUTED". (The upstream source §5.2.1 states an entry condition *"single-artifact inspection passed, no undisposed P0 defects"* with failure handling *"halt cross-artifact inspection"*; this lite skill deliberately runs non-blocking and aggregates P0 findings instead of halting.)
- **Confirm artifact version consistency before starting** (§5.2.2): Prompt, Skill, and Tool version numbers, update timestamps, and sources must be consistent; if not, ask the user before proceeding.
- **Determine the Agent tier (AC-L1/2/3)** and run two layers, do not collapse them:
  1. **Evaluation scope — which items get a verdict** — per §8.3.1 / Appendix A: **AC-L1 = 7 items, AC-L2 = 20, AC-L3 = 29** (the applicable items marked `✓` in Appendix A). Higher tier ⊇ lower tier.
  2. **Blocking core — which FAILs block release** — the tier's P0 minimum rule set (Appendix B): **AC-L1 = 7, AC-L2 = 9, AC-L3 = 9** (the same 9 P0 items for L2/L3). Items evaluated at P1/P2 do not block but must still be reported.
- **Inspect only cross-artifact relationships** — the six relationships defined by the spec (§1.3): QD-PS (Prompt → Skill), QD-PT (Prompt → Tool), QD-ST (Skill ↔ Tool), plus three extended relationships QD-SS (Skill ↔ Skill), QD-TT (Tool ↔ Tool), QD-CYC (circular invocation). Do not re-check single-artifact internal defects owned by QD-P / QD-S / QD-T.
- **For each rule, output: rule ID + verdict (PASS / FAIL with severity) + evidence quoting BOTH artifact locations** (three-tier localization: relationship → artifact → field, §6.1.1).
- **Five-state result per rule** (§5.3.8): **Pass / Defect / Not Applicable (N/A) / Insufficient Evidence / Not Checked**. "Insufficient Evidence" and "Not Checked" must never be recorded as pass; N/A must record its basis (e.g. "no Tool Schema", "Agent has only 1 Skill"); a **basis-less N/A is treated as unchecked** and counted as a failure at the item's default level (N/A discipline).
- **Severity = default level + two upgrades** (§5.4.5, Appendix C): the default level is the Appendix A "Severity Level (default)" value; upgrades come from (a) the rule body's `→ P0` instance-upgrade branch (record the reason + evidence), and (b) the Appendix C.2 `OR-L3` mapping. When Appendix A and a body branch conflict, **Appendix A is the default, the body branch is the upgrade condition**.
- **Verdict rule**: any P0 defect → evaluation FAIL (Appendix B.1). A final report with unremediated P0 defects must follow Step 7: *"Output blocking report; require re-inspection after remediation"*. This FAIL is a cross-stage result only; in this lite non-blocking flow it does **not** halt subsequent stages.
- **Respect the scope exclusions** (below) — a Cross result says nothing about single-artifact internals, runtime behavior, or system-level permission enforcement.

## Agent complexity levels (AC-L, §5.4.2, per Inspect Prompt)

| Level | Name | Definition |
| --- | --- | --- |
| AC-L1 | Single-turn Task | Single invocation completes; no state retention |
| AC-L2 | Multi-turn Interactive | Multi-turn dialogue or simple tool invocations |
| AC-L3 | Complex Agent | Complex workflow, multi-tool collaboration, strong constraints |

> `AC-L4` (multi-Agent collaboration) is defined in core.md §4.2 but not yet refined here; AC-L4 Agents are inspected as AC-L3 (full 29 items) (§5.4.3 note).

## Complete Inspection Item List (29 items, six groups)

Default severity and AC-L applicability are reproduced from `inspect/cross.md` Appendix A. `✓` = the item gets a verdict at that tier; `-` = not applicable.

### QD-PS — Prompt → Skill (7 items)

| ID | Rule name | Definition (condensed) | Default | AC-L1 | AC-L2 | AC-L3 |
| --- | --- | --- | --- | --- | --- | --- |
| QD-PS-1.1 | Skill Authorization Consistency | Skills declared in the Prompt match the defined Skills (a Skill defined but outside the Prompt's authorization scope) | P0 | ✓ | ✓ | ✓ |
| QD-PS-1.2 | Trigger Condition Consistency | Prompt workflow trigger conditions match the Skill's TRIGGER dimension (allowed_when / forbidden_when) | P1 | - | ✓ | ✓ |
| QD-PS-1.3 | Permission Boundary Consistency | Permission scope authorized by the Prompt covers the Skill's allowed_operations | P0 | ✓ | ✓ | ✓ |
| QD-PS-1.4 | Resource Access Consistency | Resources declared by the Prompt align with the Skill's allowed_resources | P1 | - | ✓ | ✓ |
| QD-PS-1.5 | Failure Handling Consistency | Prompt's exception handling aligns with the Skill's FAILURE dimension (on_empty_input, on_insufficient_input, forbidden_behaviors) | P1 | - | - | ✓ |
| QD-PS-1.6 | Output Constraint Consistency | Prompt's output requirements satisfied by the Skill's OUTPUT dimension (max_total_chars, format, partial_result_allowed) | P2 | - | - | ✓ |
| QD-PS-1.7 | Orchestration Order & Precondition Consistency | The Skill's declared precondition set is satisfied by a prior Prompt workflow step before the Skill is invoked | P1 | - | ✓ | ✓ |

### QD-PT — Prompt → Tool (7 items)

| ID | Rule name | Definition (condensed) | Default | AC-L1 | AC-L2 | AC-L3 |
| --- | --- | --- | --- | --- | --- | --- |
| QD-PT-2.1 | Tool Existence | Tools declared in the Prompt exist in the known Tool Schema list and names are consistent | P0 | ✓ | ✓ | ✓ |
| QD-PT-2.2 | Invocation Specification Consistency | The Prompt's tool invocation spec matches the Tool's Schema (name, description, inputSchema) | P0 | ✓ | ✓ | ✓ |
| QD-PT-2.3 | Parameter Constraint Consistency | The Prompt's parameter constraints align with the Tool's inputSchema (required, properties, maxLength, pattern, …) | P1 | - | ✓ | ✓ |
| QD-PT-2.4 | Permission Granularity Consistency | The Prompt's permission declarations match the Tool's risk level — high-risk Tools (OR-L3) need security boundary declarations in the Prompt | P0 | ✓ | ✓ | ✓ |
| QD-PT-2.5 | Key Error Handling Coverage | The Prompt's exception handling covers all key error types from the Tool | P1 | - | - | ✓ |
| QD-PT-2.6 | Business-Level Frequency Expectation | The Prompt declares a reasonable business-level call frequency / batch expectation for the Tool | P2 | - | - | ✓ |
| QD-PT-2.7 | Parameter Semantics & Unit Consistency | The Prompt's parameter semantics/unit (name, unit, currency, timezone) match the Tool's inputSchema semantics | P1 | - | ✓ | ✓ |

### QD-ST — Skill ↔ Tool (7 items)

| ID | Rule name | Definition (condensed) | Default | AC-L1 | AC-L2 | AC-L3 |
| --- | --- | --- | --- | --- | --- | --- |
| QD-ST-3.1 | Tool Existence | Tools declared by the Skill (EXECUTION dimension or allowed_downstream_skills) exist in the known Tool Schema list and names are consistent | P0 | ✓ | ✓ | ✓ |
| QD-ST-3.2 | Input Constraint Consistency | The Skill's INPUT constraints (max_items, format, if_exceeds) are supported by the Tool's inputSchema (maxLength, pattern, maxItems, …) | P0 | - | ✓ | ✓ |
| QD-ST-3.3 | Required Parameter Preparation | The Tool's required parameters (inputSchema.required) have a source declaration in the Skill (user input or another tool's output) | P0 | - | ✓ | ✓ |
| QD-ST-3.4 | Business-Level Frequency Expectation | The Skill's call-frequency/batch expectation is reasonable in combination with the Tool's capability | P2 | - | - | ✓ |
| QD-ST-3.5 | Output Structure Match | The Skill's OUTPUT expectations (format, max_total_chars) match the Tool's return structure | P1 | - | - | ✓ |
| QD-ST-3.6 | Permission Scope Match | The Skill's permission declarations (allowed_operations / forbidden_operations) cover the Tool's side effects (writes, sensitive data) | P0 | ✓ | ✓ | ✓ |
| QD-ST-3.7 | Key Error Handling Coverage | The Skill's FAILURE dimension covers all key error types from the Tool | P1 | - | - | ✓ |

### QD-CYC — Circular Invocation (3 items, cross-cutting)

| ID | Rule name | Definition (condensed) | Default | AC-L1 | AC-L2 | AC-L3 |
| --- | --- | --- | --- | --- | --- | --- |
| QD-CYC-1.1 | Cross-Artifact Circular Invocation | A closed loop across Prompt/Skill/Tool invocation relationships (Skill output re-enters Prompt workflow, Tool callback re-triggers Prompt, Tool side effect re-triggers Skill) | P1 | - | ✓ | ✓ |
| QD-CYC-1.2 | Multi-Agent Circular Invocation | A closed loop across multiple Agent roles in the Prompt orchestration (Agent A ↔ Agent B) | P1 | - | - | ✓ |
| QD-CYC-1.3 | Inter-Skill Handoff Loop | Multiple Skills route to each other via handoff forming a non-terminating loop | P1 | - | ✓ | ✓ |

> QD-CYC note (Appendix A.4): default P1. A **definitive closed loop** (explicit bidirectional/cyclic invocation) is determined directly as P0 per Part 4.5.2 — this P0 is not an Appendix C upgrade and is independent of the OR-L3 trigger. QD-CYC-1.2 applies only when the Prompt orchestration declares ≥2 Agent roles, else N/A (record basis).

### QD-SS — Skill ↔ Skill (2 items)

| ID | Rule name | Definition (condensed) | Default | AC-L1 | AC-L2 | AC-L3 |
| --- | --- | --- | --- | --- | --- | --- |
| QD-SS-4.1 | Trigger Condition Conflict | Multiple Skills' triggers overlap, activating several Skills on the same input | P1 | - | ✓ | ✓ |
| QD-SS-4.2 | Name Uniqueness | Multiple Skills share the same or near-identical name, causing mis-invocation | P2 | - | ✓ | ✓ |

> QD-SS note (Appendix A.5): QD-SS-4.1 default P1, upgrades to P0 when an OR-L3 Skill is involved; QD-SS-4.2 default P2, upgrades to P1 when OR-L3 is involved, and to P0 when the near-identity would mis-invoke an OR-L3 Skill.

### QD-TT — Tool ↔ Tool (3 items)

| ID | Rule name | Definition (condensed) | Default | AC-L1 | AC-L2 | AC-L3 |
| --- | --- | --- | --- | --- | --- | --- |
| QD-TT-5.1 | Cross-Tool Data Flow | A Tool's sensitive output is consumed by another Tool for a dangerous operation, forming a data-leak chain | P1 | - | ✓ | ✓ |
| QD-TT-5.2 | Multi-Tool Semantic Competition | Multiple Tools' name/description are near-identical, causing mis-invocation | P1 | - | ✓ | ✓ |
| QD-TT-5.3 | Cross-Tool Data Structure Bridging | A Tool's output/inputSchema types are incompatible and the Skill has no conversion step | P2 | - | - | ✓ |

> QD-TT note (Appendix A.6): QD-TT-5.1 default P1, upgrades to P0 when "sensitive output + dangerous consumption" co-occur; QD-TT-5.2 default P1, upgrades to P0 when behavioral difference is large.

## Minimum Rule Sets (Appendix B, P0-only)

Core principles (B.1): AC-L1 ⊂ AC-L2 ⊆ AC-L3 (P0 set is monotonic; AC-L3's P0 set equals AC-L2's because all L3-added items are P1/P2); minimum sets contain only P0 items; any P0 defect → FAIL.

**AC-L1 minimum rule set (7 P0)** — QD-PS-1.1, 1.3 · QD-PT-2.1, 2.2, 2.4 · QD-ST-3.1, 3.6

**AC-L2 minimum rule set (9 P0)** — the above 7 + QD-ST-3.2, 3.3

**AC-L3 minimum rule set (9 P0)** — same as AC-L2 (no new P0 items at L3).

## Severity upgrade (Appendix C)

Default levels are the Appendix A values. Two upgrade channels apply (record the reason + evidence in the report):

- **OR-L3 upgrade (Appendix C.2)** — when the Tool or Skill is OR-L3:
  - P1 → P0: QD-PS-1.2, 1.4, 1.5, 1.7 · QD-PT-2.3, 2.7 · QD-ST-3.5 · QD-SS-4.1 · QD-TT-5.1, 5.2 · QD-CYC-1.1, 1.2, 1.3
  - QD-SS-4.2: P2 → P1 (→ P0 when it would mis-invoke an OR-L3 Skill)
  - No upgrade (`—`): QD-PT-2.5, QD-ST-3.7 (P1 stays P1)
  - P2 items with no OR-L3 upgrade (stay P2): QD-PS-1.6, QD-PT-2.6, QD-ST-3.4, QD-TT-5.3
- **Instance upgrade (each rule body's `→ P0` branch, §5.4.5)** — e.g. a definitive QD-CYC closed loop is P0 directly.

> General rule (Appendix C.2 note): a default-P2 item caps at P1 under OR-L3; it rises to P0 only when the item's own rule declares "would mis-invoke an OR-L3 operation" (currently only QD-SS-4.2).

## Same-source deduplication (§8.5.2)

When one fact hits multiple IDs in the same group below, charge once at the highest level; keep all IDs and evidence, mark the primary ID, and record the group. The denominator is unchanged (always the tier's applicable-item count).

| Group | IDs | Merge condition (same fact) |
| --- | --- | --- |
| Tool existence | QD-PT-2.1 ↔ QD-ST-3.1 | The same nonexistent Tool name referenced by both Prompt and Skill |
| Permission boundary | QD-PS-1.3 ↔ 1.4 ↔ QD-ST-3.6 | Evidence on the same `allowed_operations`/`allowed_resources` declaration, closable by one fix |
| Parameter constraint | QD-PT-2.3 ↔ QD-ST-3.2 | The same constraint gap on the same parameter |
| Output contract | QD-PS-1.6 ↔ QD-ST-3.5 | The same structural mismatch on the same output (primary: QD-ST-3.5) |
| Error handling | QD-PT-2.5 ↔ QD-ST-3.7 | The same key error type uncovered on both sides |

> QD-PT-2.7 has no Skill-side same-source counterpart and is excluded from dedup. Outside these groups, items are scored separately; when uncertain, treat as not-same-source and note the basis.

## Scoring note (§8.3, optional)

Weights **P0 : P1 : P2 = 5 : 3 : 1**; base = 100 / total weight. Per failed item deduct base × 5 / 3 / 1 (once per item number regardless of instance count, §8.5.1); any P0 → FAIL (score still output). Cross scoring is independent of single-artifact scoring and is never combined with it (§8.3 note).

| AC-L | Applicable items | P0 | P1 | P2 | Total weight | Base |
| --- | --- | --- | --- | --- | --- | --- |
| **AC-L1** | 7 | 7 | 0 | 0 | 5×7 = 35 | 100/35 ≈ 2.857 |
| **AC-L2** | 20 | 9 | 10 | 1 | 5×9 + 3×10 + 1×1 = 76 | 100/76 ≈ 1.316 |
| **AC-L3** | 29 | 9 | 15 | 5 | 5×9 + 3×15 + 1×5 = 95 | 100/95 ≈ 1.053 |

Compute an indicative score only if the user asks, and label it "indicative (lite minimal set), not the full-spec score".

## Inspection Process (cross.md Part 5.1.1 — eight steps)

1. **Prerequisite Verification** — confirm artifact version consistency. Upstream single-artifact P0 findings do **not** halt this lite run (record and carry forward).
2. **Artifact Relationship Identification** — extract Skill and Tool declaration lists (Prompt QD-P-2.4.2; Skill EXECUTION dimension / allowed_downstream_skills). On extraction failure, flag and continue as advisory.
3. **QD-PS Check (Prompt → Skill)** — QD-PS-1.1–1.7.
4. **QD-ST Check (Skill ↔ Tool)** — QD-ST-3.1–3.7.
5. **QD-PT Check (Prompt → Tool)** — QD-PT-2.1–2.7.
6. **Extended-relationship checks (L2/L3)** — QD-CYC-1.1/1.2/1.3, QD-SS-4.1/4.2, QD-TT-5.1/5.2/5.3.
7. **Impact Assessment** — identify linked defects; apply OR-L3 upgrade and instance-upgrade; adjust remediation priority.
8. **Output Report** — defect list + remediation guidance; unremediated P0 → *"Output blocking report; require re-inspection after remediation"*.

## Does NOT Check (Scope Exclusions)

- Does not duplicate single-artifact inspection content owned by QD-P / QD-S / QD-T (§0.1.2).
- Does not re-discover defects already covered by single-artifact inspection; Cross focuses on **hidden defects under reasonable expression** — each artifact individually valid, but semantic conflicts/gaps emerge when combined (§0.1.3).
- Does not prescribe how to design "good" cross-artifact relationships (§0.1.2).
- Does not monitor cross-artifact behavior at runtime (§0.1.2 / §0.1.3).
- Does not replace system-level permission control mechanisms (§0.1.2).
- Tool/Skill existence checks are declaration-only: static inspection verifies "declaration existence"; runtime "actual availability" is verified by Risk Implicit (notes to QD-PT-2.1 / QD-ST-3.1).
