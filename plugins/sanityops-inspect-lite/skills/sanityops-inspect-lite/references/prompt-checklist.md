---
title: System Prompt Inspection Checklist (Lite)
description: "Condensed QD-P inspection checklist for the sanityops-inspect lite inspector: Agent tiering (AC-L1/L2/L3), the complete tiered inspection checklist matrix (48 rows incl. 4 conditional items and 6 ungraded recommendation items) with per-tier severity, cumulative minimum rule sets (L1=7 / L2=17 / L3=25 P0 items) with condensed definitions, four-state results, primary-defect merging, and the scoring note. Extracted from inspect/prompt.md v1.1 (2026-10); defines no new rules."
---

# System Prompt Inspection Checklist (QD-P, Lite)

> **Provenance**: Condensed from `inspect/prompt.md` v1.1 (2026-10) (SanityOps Framework, CC BY-SA 4.0). This file is a compressed extraction for the sanityops-inspect lite inspector and defines NO new rules. On any conflict, `inspect/prompt.md` governs.

## How to use

- **Determine the Agent tier first** (§3.1): **AC-L1** Single-turn Task (single skill, no tools, no multi-turn, no complex reasoning); **AC-L2** Multi-turn Interactive (multi-turn, 1–3 tools, simple workflow ≤ 5 steps, limited state); **AC-L3** Complex Agent (4+ tools, workflow > 5 steps, security-sensitive, requires exception handling). **When an Agent satisfies multiple levels, take the highest** (§3.1.2). State the determination in the report.
- **Symbol convention (§3.1 note)**: throughout the source, bare `L1/L2/L3` **equal** `AC-L1/AC-L2/AC-L3` (Agent complexity level) and must never be read as `OR-L` (operation risk level). System Prompt has no operation-risk dimension; its severity is driven by `AC-L` (core.md §4.1).
- **Inspect the tier's checklist scope** (§3.3.2 matrix below): L1 = 28 items, L2 = 42, L3 = 44. Upward-compatible (L3 ⊃ L2 ⊃ L1, §3.3.1). Items marked `—` do not apply at that tier.
- **Severity is tier-specific** (§3.2.2, §3.2.3): the same defect may upgrade across tiers (e.g. Priority Conflict P1 at L1/L2 → P0 at L3; Undefined Boundaries P1 at L1 → P0 at L2/L3). Apply the severity shown in the inspected tier's column.
- **Minimum rule sets are the compliance basis** (Appendix A): a tier is compliant only if **all of its P0 items pass** — L1: 7; L2: 17 (includes L1's 7); L3: 25 (includes L2's 17). Any P0 failed → Non-compliant. P1 SHOULD be fixed (or carry an `EXC-*` exception, core.md §4.6); P2 MAY be fixed.
- **Four-state result per rule** (§3.3.4): **通过 / 不通过 / 不适用(N/A) / 证据不足**. N/A is removed from the scoring denominator; "证据不足" is recorded "to-be-verified" and counts neither pass nor fail. A fifth category **建议项（不定级）** exists for the 6 recommendation items (see below) — never graded, never counted, waiver-free.
- **Primary-defect merging** (§4.6): when one fact hits multiple semantically overlapping rule IDs, charge only the single **primary** ID and record the rest as "related" (no double deduction). The specific overlap groups are enumerated in source §4.6 — read them there rather than re-deriving.
- **Output one verdict per rule** in the form `QD-P-x.y.z | PASS/FAIL | evidence: file/section/line`, quoting a short excerpt of the offending text for every FAIL.
- **Judgment threshold — a missing declaration is the defect**: for Part 2 completeness/security rules (QD-P-2.x), a rule is FAIL when the declaration it names is **absent** from the Prompt. The "leading-to …" tail describes the **risk mechanism**, not an additional precondition to prove. (Part 1 QD-P-1.x rules detect *written* contradictions/conflicts, so there the defect is the conflict's presence.) Several rules carry tightened applicability preconditions — read the matrix's severity notes before judging.
- **Single-artifact boundary (Stage A)**: judge a Prompt rule against the Prompt alone; a declaration found in a *different* artifact never satisfies a Prompt-level missing declaration. Every FAIL evidence must quote the Prompt text.
- **P0 findings are release-blocking but do not halt the flow** (non-blocking): report every P0 and continue.
- **Detection nature differs by Part**: Part 1 (QD-P-1.x) logic defects are detectable by semantic reading of the prompt text; Part 2 (QD-P-2.x) defects are completeness/security gaps requiring the explicit checklist (the LLM will not proactively notice them). Gray-zone items (QD-P-1.6.x) carry reduced detection-reliability labels (Partially reliable / Unstable / Conditionally reliable, §1.6).

## Agent tiers (§3.1)

| Level | Name | Definition | Determination criteria (take highest when multiple) |
| --- | --- | --- | --- |
| **AC-L1** | Single-turn Task | Single invocation completes; no state retention | Single skill; no tool invocations; no multi-turn dialogue; no complex reasoning |
| **AC-L2** | Multi-turn Interactive | Multi-turn dialogue or simple tool invocations | Multi-turn dialogue; 1–3 tool invocations; simple workflow (≤ 5 steps); limited state management |
| **AC-L3** | Complex Agent | Complex workflow, multi-tool collaboration, strong constraints | 4+ tool invocations; complex workflow (> 5 steps); security-sensitive (privacy/permissions); requires exception handling |

## Complete tiered inspection checklist (§3.3.2)

Severity per tier: 🔴 = P0, 🟡 = P1, 🟢 = P2, — = not inspected at that tier.

**Part 1: Logic Defects Within LLM Autonomous Detection Capability**

| QD-P ID | Defect Name | L1 | L2 | L3 | Severity Variation Notes |
| --- | --- | --- | --- | --- | --- |
| **QD-P-1.1.1** | Direct Contradiction | 🔴 | 🔴 | 🔴 | P0 at all levels |
| **QD-P-1.1.2** | Priority Conflict | 🟡 | 🟡 | 🔴 | L1/L2 P1; L3 P0 |
| **QD-P-1.1.3** | Example-Rule Contradiction | 🔴 | 🔴 | 🔴 | P0 at all levels |
| **QD-P-1.2.1** | Role-Responsibility Mismatch | 🔴 | 🔴 | 🔴 | P0 at all levels |
| **QD-P-1.2.2** | Role-Output Style Mismatch | 🟡 | 🟡 | 🟡 | P1 at all levels |
| **QD-P-1.2.3** | Constraint-Responsibility Mismatch | 🔴 | 🔴 | 🔴 | P0 at all levels |
| **QD-P-1.2.4** | Identity & Anthropomorphism Boundary | 🟡 | 🟡 | 🟡 | P1 at all levels; P0 when user-facing (C-end) |
| **QD-P-1.3.1** | Quantifier Misuse | 🟡 | 🟡 | 🟡 | P1 at all levels |
| **QD-P-1.3.2** | Vague Qualifiers | 🟡 | 🟡 | 🟡 | P1 at all levels |
| **QD-P-1.3.3** | Undefined Boundaries | 🟡 | 🔴 | 🔴 | L1 P1; L2/L3 P0 |
| **QD-P-1.4.3** | Invalid Skill Definitions | — | 🟡 | 🔴 | L1 has no Skills; L2 P1; L3 P0 |
| **QD-P-1.5.1** | Responsibilities Do Not Support Objective | 🔴 | 🔴 | 🔴 | P0 at all levels |
| **QD-P-1.5.2** | Resources Do Not Support Workflow | — | 🔴 | 🔴 | L1 has no workflow; L2/L3 P0 |
| **QD-P-1.6.1** | Critical Decision Word Vagueness | 🟡 | 🟡 | 🟡 | P1 at all levels; P0 when the vague word carries a security/permission decision |
| **QD-P-1.6.2** | Rules Depending on Model Internal State | 🟡 | 🟡 | 🟡 | P1; only when a critical decision relies solely on model self-evaluation with no observable trigger |
| **QD-P-1.6.3** | Soft Rule Interweaving Without Arbitration | 🟡 | 🟡 | 🟡 | P1; P0 when the conflict involves security/permission |
| **QD-P-1.6.4** | Reasoning Method Constraints | 🟡 | 🟡 | 🟡 | P1; only when an unverifiable internal reasoning method is used as compliance assurance |
| **QD-P-1.6.5** | Self-Referential or Recursive Constraints | 🟡 | 🟡 | 🟡 | P1; only non-terminating / unsatisfiable / circular recursion |
| **QD-P-1.6.6** | Implicit Assumption Dependency | 🟡 | 🟡 | 🟡 | P1; only critical, non-obvious, externally-unguaranteed assumptions |
| **QD-P-1.6.7** | Non-Monotonic Reasoning Defects | 🟡 | 🟡 | 🟡 | P1 when cross-turn state or committed conclusions exist; N/A single-turn |
| **QD-P-1.6.8** | Rule Followability | 🟡 | 🟡 | 🟡 | P1; only when the rule demands a model-unreliable capability (counting/timing/arithmetic) |

**Part 2: Defects Requiring Explicit Prompts**

| QD-P ID | Defect Name | L1 | L2 | L3 | Severity Variation Notes |
| --- | --- | --- | --- | --- | --- |
| **QD-P-2.1.2** | Reference Clarity | — | 🟡 | 🟡 | L1 needs no tools; L2/L3 P1 |
| **QD-P-2.2.1** | Undeclared Input Source | 🔴 | 🔴 | 🔴 | P0 at all levels |
| **QD-P-2.2.2** | Unconstrained Input Format | 🟡 | 🔴 | 🔴 | L1 P1; L2/L3 P0 only when structured input is received |
| **QD-P-2.2.3** | Required/Optional Not Distinguished | 🟡 | 🔴 | 🔴 | L1 P1; L2/L3 P0 only when structured input is received |
| **QD-P-2.2.4** | Undefined Missing-Input Handling | 🟡 | 🔴 | 🔴 | L1 P1; L2/L3 P0 |
| **QD-P-2.3.1** | Undefined Output Format | 🟡 | 🔴 | 🔴 | L1 P1; L2/L3 P0 (output parsed by downstream tools) |
| **QD-P-2.3.2** | Incomplete Output Schema | 🟡 | 🔴 | 🔴 | Only when structured output is declared; L1 P1, L2/L3 P0 |
| **QD-P-2.3.4** | Missing Security/Privacy Filtering | — | — | — | Conditional: P0 at all levels when personal data (PII) is involved, else N/A |
| **QD-P-2.3.5** | Factuality & Hallucination Control Declaration | — | 🔴 | 🔴 | L2/L3 P0 (RAG/Tool agents); L1 has no retrieval/tools → N/A |
| **QD-P-2.3.6** | Output Length & Truncation | 🟢 | 🟢 | 🟢 | P2 at all levels |
| **QD-P-2.3.7** | Output Localization & Conventions | — | — | — | Multi-region P1; single-region N/A |
| **QD-P-2.4.1** | Incomplete Workflow Steps | — | 🟡 | 🔴 | L1 has no workflow; L2 P1; L3 P0 |
| **QD-P-2.4.2** | Undeclared Resources/Tools | — | 🔴 | 🔴 | L1 has no tools; L2/L3 P0 |
| **QD-P-2.4.3** | Tool Invocation Parameter/Order Error | — | 🟡 | 🟡 | L2/L3 P1; only wrong parameters/order written in the prompt body |
| **QD-P-2.4.4** | Multi-turn Context & Memory Boundary | — | 🟡 | 🟡 | Multi-turn/cross-session only; L2/L3 P1 |
| **QD-P-2.5.1** | Missing Termination Constraints | — | 🟡 | 🔴 | L1 has no retries; L2 P1; L3 P0 |
| **QD-P-2.5.2** | Missing Security Boundary Constraints | 🔴 | 🔴 | 🔴 | P0 at all levels |
| **QD-P-2.5.3** | Missing Permission Control Boundaries | — | 🟡 | 🔴 | L1 has no permission control; L2 P1; L3 P0 |
| **QD-P-2.5.4** | Undefined Constraint Priority | — | 🟡 | 🔴 | L1 constraints are simple; L2 P1; L3 P0 |
| **QD-P-2.5.6** | Instruction-Data Trust Boundary | — | 🔴 | 🔴 | L2/L3 P0 (missing injection-protection declaration); actual resistance verified by Risk Implicit |
| **QD-P-2.5.7** | High-Risk Operation Human Confirmation | — | — | — | Conditional: P0 when the Agent has side-effect ops (funds/delete/send/production change) at L2/L3, else N/A |
| **QD-P-2.5.8** | System Prompt Confidentiality & Leakage | — | 🟡 | 🟡 | L2/L3 P1 (confidentiality/leakage-response declaration); actual leakage verified by Risk Implicit |
| **QD-P-2.5.9** | Industry Compliance & Disclaimer Boundary | — | — | — | Conditional: P0 in medical/financial/legal domains, else N/A |
| **QD-P-2.6.1** | Runtime Exceptions Not Covered | — | 🟡 | 🔴 | L1 has no tools; L2 P1; L3 P0 |
| **QD-P-2.6.2** | Missing Degradation & Recovery Strategy | — | — | 🟡 | L3 only; P1; "stop and report" is a legitimate disposition |
| **QD-P-2.6.3** | Unreasonable Retry Mechanism | — | — | 🟡 | L3 only; P1 (cap / blind-write retry / unqueried timeout state) |
| **QD-P-2.6.4** | Out-of-scope Rejection & Human Escalation | 🟡 | 🟡 | 🔴 | L1/L2 P1; L3 P0 (rejection template / referral / escalation) |

> **Recommendation items (不定级, ungraded — 6)**: QD-P-1.4.1 (redundant constraints), QD-P-2.1.3 (structural hierarchy), QD-P-2.1.5 (context budget squeeze), QD-P-2.7.1 (positive-example representativeness), QD-P-2.7.2 (counter-example coverage), QD-P-2.7.3 (boundary-case examples). These carry only optimization value — not scored, not blocking, waiver-free (§3.3.2 note).

**Inspection item statistics (§3.3.3)** — also the per-tier item counts (N_P0, N_P1, N_P2) used by the scoring formula:

| Agent Level | Total Items | P0 Items | P1 Items | P2 Items |
| --- | --- | --- | --- | --- |
| **AC-L1** | 28 | 7 | 20 | 1 |
| **AC-L2** | 42 | 17 | 24 | 1 |
| **AC-L3** | 44 | 25 | 18 | 1 |

> **Count scope note** (reproduced, not adjudicated): "Total Items" counts only items applicable at that tier (🔴/🟡/🟢 in §3.3.2); `—` items are excluded. The full §3.3.2 matrix has **48 rows**; L3's 44 = 48 − 4 conditional items (QD-P-2.3.4 / QD-P-2.3.7 / QD-P-2.5.7 / QD-P-2.5.9) that carry no fixed tier severity. L2→L3 adds only 2 items (QD-P-2.6.2, 2.6.3); the rest of the L3 growth is **upgraded severity** (8 items P1→P0: QD-P-1.1.2, 1.4.3, 2.4.1, 2.5.1, 2.5.3, 2.5.4, 2.6.1, 2.6.4), lifting L3 P0 from 17 to 25. QD-P-2.5.5 (resource-call frequency limit) is **deprecated** in this revision (moved to Inspect Cross QD-PT-2.6 / QD-ST-3.4); numbering gaps 2.1.1/2.1.3/2.1.4/2.1.5/2.3.3 are deleted/merged per the version history. On conflict `inspect/prompt.md` governs.

## Minimum rule sets (Appendix A) — condensed

The sets are **cumulative**: L2 ⊇ L1, L3 ⊇ L2 (§A.4.2). Every listed item is a **P0** item at the tier whose set it belongs to.

### L1 minimum rule set — 7 P0 items (§A.1)

| ID | Rule name | Definition (condensed) |
| --- | --- | --- |
| QD-P-1.1.1 | Direct Contradiction | The same object assigned mutually exclusive attributes/directives within the same context |
| QD-P-1.1.3 | Example-Rule Contradiction | Rule definitions in the body contradict the provided examples |
| QD-P-1.2.1 | Role-Responsibility Mismatch | Role identity/capability scope does not match the responsibility requirements |
| QD-P-1.2.3 | Constraint-Responsibility Mismatch | Constraint scope conflicts/mismatches the responsibility's behavioral scope |
| QD-P-1.5.1 | Responsibilities Do Not Support Objective | The defined behaviors cannot accomplish the role's overall objective |
| QD-P-2.2.1 | Undeclared Input Source | The input source (user/system/external data) is not declared |
| QD-P-2.5.2 | Missing Security Boundary Constraints | Security boundary constraints (prohibited ops, sensitive-data access, external-system access) are undefined |

### L2 minimum rule set = L1's 7 + 10 = 17 P0 items (§A.2)

| ID | Rule name | Definition (condensed) |
| --- | --- | --- |
| QD-P-1.3.3 | Undefined Boundaries | Critical behavioral boundaries (authorization/prohibition/handoff) undefined |
| QD-P-1.5.2 | Resources Do Not Support Workflow | A workflow step references a resource/tool that is not defined or authorized |
| QD-P-2.2.2 | Unconstrained Input Format | Input data format/encoding/size limit not declared (structured-input context) |
| QD-P-2.2.3 | Required/Optional Not Distinguished | Required vs optional parameters not distinguished (structured-input context) |
| QD-P-2.2.4 | Undefined Missing-Input Handling | Handling of missing/anomalous input not defined |
| QD-P-2.3.1 | Undefined Output Format | Output data format/structure/encoding not defined |
| QD-P-2.3.2 | Incomplete Output Schema | Output Schema missing critical fields / field types / meanings |
| QD-P-2.3.5 | Factuality & Hallucination Control Declaration | No declaration against fabricating tool results / citing uncalled tools / inventing references |
| QD-P-2.4.2 | Undeclared Resources/Tools | Databases/APIs/tools used are not declared |
| QD-P-2.5.6 | Instruction-Data Trust Boundary | Trust boundary between instructions and external data undeclared (missing injection-protection declaration) |

### L3 minimum rule set = L2's 17 + 8 = 25 P0 items (§A.3)

| ID | Rule name | Definition (condensed) |
| --- | --- | --- |
| QD-P-1.1.2 | Priority Conflict | Multiple "MUST" constraints conflict in scenarios with no priority/arbitration defined |
| QD-P-1.4.3 | Invalid Skill Definitions | A defined Skill has unclear trigger/usage or conflicts so it is never invoked |
| QD-P-2.4.1 | Incomplete Workflow Steps | Workflow step dependencies/order undefined, or a step references an undefined step |
| QD-P-2.5.1 | Missing Termination Constraints | Completion/abandonment conditions, partial-success reporting, or "must-not-claim-success" steps undefined |
| QD-P-2.5.3 | Missing Permission Control Boundaries | Permission scope, elevation conditions, or escalation handling undefined |
| QD-P-2.5.4 | Undefined Constraint Priority | Arbitration rules for potentially-conflicting "SHOULD" constraints undefined |
| QD-P-2.6.1 | Runtime Exceptions Not Covered | Handling of tool-invocation/data-parsing/external-system exceptions undefined |
| QD-P-2.6.4 | Out-of-scope Rejection & Human Escalation | Rejection template, referral target, and escalation conditions undefined |

### Conditional P0 items (§A.3.1, not in the fixed 7/17/25 count)

| ID | Defect Name | Trigger condition | When not triggered |
| --- | --- | --- | --- |
| QD-P-2.3.4 | Missing Security/Privacy Filtering | Processing involves personal data (PII) | N/A |
| QD-P-2.5.7 | High-Risk Operation Human Confirmation | Has side-effect ops (funds/delete/send/production change) at L2/L3 | N/A |
| QD-P-2.5.9 | Industry Compliance & Disclaimer Boundary | Medical/financial/legal regulated domain | N/A |

Triggered → P0, must fix; not triggered → N/A, removed from the denominator (§3.3.4).

## Scoring note

Deduction weights **P0 : P1 : P2 = 5 : 3 : 1** (source: `inspect/prompt.md` §4.1). Total weight $W_{total} = 5\cdot N_{P0} + 3\cdot N_{P1} + 1\cdot N_{P2}$, with $N_{P0},N_{P1},N_{P2}$ from the §3.3.3 table; base deduction unit $base = 100 / W_{total}$; $Score = \max(100 - \text{total deductions}, 0)$.

| Agent Level | $W_{total}$ | $base$ |
| --- | --- | --- |
| **AC-L1** | $5\cdot7+3\cdot20+1\cdot1 = 96$ | $100/96 = 1.04$ |
| **AC-L2** | $5\cdot17+3\cdot24+1\cdot1 = 158$ | $100/158 = 0.63$ |
| **AC-L3** | $5\cdot25+3\cdot18+1\cdot1 = 180$ | $100/180 = 0.56$ |

- **Deductions**: P0 = $base \times 5$; P1 = $base \times 3$; P2 = $base \times 1$. The same inspection item number is deducted only once regardless of instance count (§4.6). Any P0 defect → **FAIL**, with the score still output (§4.1). Cross-tier scores are not comparable (§4.1).
- **Primary-defect merging** (§4.6): one fact hitting multiple overlapping IDs is charged only to the primary ID; the rest are "related". The overlap groups are defined in source §4.6 (read them there; some groups reference IDs that have since been deleted/merged in v1.1).
- The lite inspector reports verdicts and evidence, and computes an indicative numeric score only on request.
