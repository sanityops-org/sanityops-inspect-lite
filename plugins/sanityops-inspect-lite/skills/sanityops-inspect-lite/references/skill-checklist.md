---
title: Skill Inspection Checklist (Lite Inspector)
description: Condensed QD-S checklist for the sanityops-inspect lite inspector — QD-S-0 baseline gate (9 items), L1/L2/L3 minimum rule sets (8/23/24 semantic items), the five-state judgment model, the effective-constraints & compensating-controls principle, the six-step inspection flow, the six-dimension Review Schema, and fixed-deduction scoring. Extracted from inspect/skill.md v1.1; defines no new rules.
---

> Condensed from `inspect/skill.md` v1.1 (SanityOps Framework, CC BY-SA 4.0). This file is a compressed extraction for the sanityops-inspect lite inspector and defines NO new rules. On any conflict, `inspect/skill.md` governs.

# Skill Inspection Checklist (QD-S)

## 1. How to use

- **Run the QD-S-0 baseline FIRST** (Section 2). It is the entry gate. **Relaxed failure handling (v1.1)**: only a *completely unparseable* Skill (QD-S-0.3 fails) stops QD-S-1–5 inspection; any other QD-S-0 defect (missing field, empty content, local conflict) does not stop downstream semantic inspection, but the report must mark "baseline not passed", list the defective items, and record the overall conclusion as **"assessment incomplete (NEEDS_REVIEW)"** — confirmed QD-S defects are still reported individually (core.md §6.2).
- **Determine the Skill risk level (OR-L1/L2/L3) next** (§7.3.2): irreversible/delete operations → L3; reversible/isolated writes, external-system access, or sensitive data → at least L2; otherwise L1 (text-processing only). When the mounted tool/permission context is unavailable, output "tier undetermined" and do **not** force a tier (§7.3.2, §7.5).
- **Apply the matching minimum rule set** from Section 4 (cumulative: higher tier ⊇ lower tier). Never emit IDs not in this file.
- **One verdict line per rule**: `QD-S-x.y | PASS/FAIL | evidence: file, section/line`. Every FAIL must quote the offending artifact text (short verbatim excerpt).
- **Five-state result per rule** (§7.4): **Pass / Defect / Not Applicable (N/A) / Insufficient Evidence / Not Checked**. "Not Checked" and "Insufficient Evidence" must never be recorded as pass; N/A items are not scored but must record the N/A basis (e.g. QD-S-5.3 no multi-environment, QD-S-5.4 `allow_skill_chaining=false`); "Insufficient Evidence" is output as "to-be-verified" and must not be graded.
- **Judgment threshold — a missing declaration is the defect**: a rule is FAIL when the declaration its definition names is **absent** from the Skill. The "allowing / leading-to …" tail describes the **risk mechanism**, not an additional precondition to prove. (QD-S-2.x are the exception: they detect *written* unbounded wording, so there the defect is the wording's presence.)
- **Effective constraints & compensating controls** (§7.5, cross-cutting severity principle): distinguish **Missing Constraint / Constraint Conflict / Insufficient Evidence (to-be-confirmed)**. Traceable in-artifact constraints (Tool Schema fields, `allowed_operations`, declared scope/service-account) may inform severity; external Harness/RBAC/platform controls must **not** be used to close a static defect (they are only `compensating_controls` under Exception, affecting disposition not defect existence); on insufficient evidence, output "to-be-verified", never a forced grade.
- **Severity semantics** (§0.2.4): P0 blocking (MUST NOT release); P1 warning (strongly recommended); P2 advisory (MAY release). P0 is release-blocking but in this lite non-blocking flow does not halt downstream stages.
- **Risk-level semantics** (§0.2.3): L1 = text-processing only, no tool; L2 = resource access, may involve sensitive data; L3 = operation execution, irreversible.
- **The final Skill report is organized by the six Review Schema dimensions** (IDENTITY / TRIGGER / INPUT / EXECUTION / OUTPUT / FAILURE, Section 6), followed by a defect list sorted by priority (P0 first).

## 2. QD-S-0 Baseline Compliance (entry gate) — 9 items

QD-S-0 verifies platform minimum structure only ("can it be parsed and inspected"), not semantic quality. Result is pass/fail and is **not** scored (§11.3). QD-S-0.9 is SHOULD-level and does not count toward the pass/fail gate.

| ID | Name | Definition (condensed) | Method |
| --- | --- | --- | --- |
| QD-S-0.1 | Required Field Presence | Required fields (`name`, `description`, …) are present | Automated |
| QD-S-0.2 | Field Content Non-Empty | Required fields carry non-empty, non-blank content | Automated |
| QD-S-0.3 | Syntax Parsability | Definition parses as valid syntax (YAML frontmatter, Markdown structure) | Automated |
| QD-S-0.4 | Field Type Validity | Each field's value matches its expected type (enum/boolean/number/list) | Automated |
| QD-S-0.5 | No Conflicting Declarations | Body, examples (few-shot), failure strategy, and permission declarations are mutually consistent; on conflict, report both sides — never silently pick the looser or stricter side | Automated (candidate) → human review |
| QD-S-0.6 | Minimum Structural Completeness | Has locatable minimum structure (referencable locations, inspectable units) | Automated |
| QD-S-0.7 | Dependency & Resource Existence | Referenced scripts/templates/downstream Skills exist and parse. Hardcoded credentials/abnormal callback URLs/PII are Risk Explicit (NR-S-9 / NR-S-7 / NR-S-10), cross-referenced as EX — not recorded under QD-S-0 | Automated |
| QD-S-0.8 | Metadata Completeness | `version`, `owner`, `changelog` are present and traceable (configurable as platform-level requirement) | Automated |
| QD-S-0.9 | description Length Budget | `description` does not exceed token budget (SHOULD, complements QD-S-3.6) | Automated |

## 3. Defect levels and risk levels (reference)

**Defect levels (P)** — source §0.2.4:

| Level | Meaning | Release recommendation |
| --- | --- | --- |
| P0 🔴 | Blocking: a directly realizable resource runaway or behavioral overflow path exists | MUST NOT release; MUST be fixed |
| P1 🟡 | Warning: defect impacts behavioral quality; predictable unintended behavior exists | Strongly recommended to fix |
| P2 🟢 | Advisory: minor specification compliance issue | Recommended to fix; MAY release |

**Skill risk levels (OR-L)** — source §0.2.3:

| Level | Designation | Definition |
| --- | --- | --- |
| L1 (OR-L1) | Low Risk | Text-processing only; no tool invocations |
| L2 (OR-L2) | Medium Risk | Resource access; may involve sensitive data |
| L3 (OR-L3) | High Risk | Operation execution; irreversible operations |

## 4. Minimum rule sets (§7.2, Appendix B; cumulative)

Rule names are from the source numbering. The Level column gives the rule's defect level **at that tier**. L1 has no P0 items; L2/L3 P0 items are the must-pass core.

### 4.1 L1 (Low Risk) — 8 semantic items (P0: 0, P1: 7, P2: 1)

Must-pass item: QD-S-0 (Baseline Compliance, Section 2).

| ID | Name | Definition (condensed) | Level |
| --- | --- | --- | --- |
| QD-S-1.1 | Input Boundary Missing | No upper limit declared on the number/size of inputs, allowing unbounded input | P1 |
| QD-S-1.2 | Output Size Missing | No upper limit declared on output size, risking output bloat / context overflow | P1 |
| QD-S-2.1 | Unbounded Quantity Declaration | Uses unbounded words (`all / every / entire / complete`) without a quantity cap | P1 |
| QD-S-3.1 | Vague Trigger Conditions | Activation conditions overly broad; multiple unrelated intents can trigger the Skill | P1 |
| QD-S-3.3 | Missing Ambiguous Input Handling | No handling declared for incomplete/malformed input; LLM auto-completes or infers | P1 |
| QD-S-3.4 | Undefined Task Scope | Missing `non_goals` — does not state what the Skill does NOT do | P2 |
| QD-S-3.6 | Metadata Expression Defects | `name`/`description` overly broad, inaccurate, or inconsistent with the body | P1 |
| QD-S-4.1 | Undefined Failure Behavior | No handling declared for execution failure; LLM explores alternative paths | P1 |

Optional (recommended, not required): QD-S-3.2, QD-S-3.7.

### 4.2 L2 (Medium Risk) — 23 semantic items (P0: 5, P1: 17, P2: 1)

Must-pass item: QD-S-0.

**P0 items (5)** — must pass:

| ID | Name | Definition (condensed) |
| --- | --- | --- |
| QD-S-1.1 | Input Boundary Missing | No upper limit on input number/size |
| QD-S-1.3 | Execution Resource Boundary Missing | Tool invocations/retries declared with no retry count, invocation count, or timeout |
| QD-S-2.1 | Unbounded Quantity Declaration | Unbounded words (`all/every/entire/complete`) without a quantity cap |
| QD-S-2.2 | Unterminated Execution Declaration | `keep trying / until success / ensure complete / persist until` without a termination condition |
| QD-S-4.1 | Undefined Failure Behavior | No handling declared for execution failure |

**P1 items (17)**: QD-S-1.2, 1.4, 1.5 · QD-S-2.3, 2.4 · QD-S-3.1, 3.2, 3.3, 3.4, 3.5, 3.6, 3.7 · QD-S-4.2, 4.3 · QD-S-5.1, 5.3, 5.4

**P2 item (1)**: QD-S-4.4

> Source note: QD-S-5.2 (high-risk confirmation) is N/A at L2 (no irreversible operations, §11.3).

### 4.3 L3 (High Risk) — 24 semantic items (P0: 17, P1: 6, P2: 1)

Must-pass item: QD-S-0. Full inspection (all QD-S-1 through QD-S-5).

**P0 items (17)**:

| Group | P0 items |
| --- | --- |
| QD-S-1 | 1.1, 1.3, 1.4, 1.5 |
| QD-S-2 | 2.1, 2.2, 2.4 |
| QD-S-3 | 3.1, 3.2, 3.3, 3.5 |
| QD-S-4 | 4.1, 4.3 |
| QD-S-5 | 5.1, 5.2, 5.3, 5.4 |

**P1 items (6)**: QD-S-1.2 · QD-S-2.3 · QD-S-3.4, 3.6, 3.7 · QD-S-4.2

**P2 item (1)**: QD-S-4.4

## 5. Condensed rule definitions (QD-S-1.x ~ QD-S-5.x)

| ID | Name | Definition (condensed) |
| --- | --- | --- |
| QD-S-1.1 | Input Boundary Missing | No upper limit on the number/size of inputs |
| QD-S-1.2 | Output Size Missing | No upper limit on output size (bloat / context overflow) |
| QD-S-1.3 | Execution Resource Boundary Missing | Tool invocations/retries with no retry count, invocation count, or timeout |
| QD-S-1.4 | Permission Boundary Missing | No read/write/delete distinction, no prohibited-operation declaration |
| QD-S-1.5 | Data Source Boundary Missing | No permitted/prohibited data sources; LLM may gather from history or external systems |
| QD-S-2.1 | Unbounded Quantity Declaration | Unbounded words without a quantity cap |
| QD-S-2.2 | Unterminated Execution Declaration | `keep trying / until success / ensure complete / persist until` without termination |
| QD-S-2.3 | Open-Ended Enumeration in Structure | `etc. / and others / such as` letting the LLM add items |
| QD-S-2.4 | Broad Capability Verbs | `manage / handle / coordinate / automate / process / address` without `non_goals` |
| QD-S-3.1 | Vague Trigger Conditions | Activation conditions overly broad; unrelated intents trigger |
| QD-S-3.2 | Missing Deactivation Conditions | Not declared when the Skill should NOT trigger |
| QD-S-3.3 | Missing Ambiguous Input Handling | No handling for incomplete/malformed input; LLM auto-completes |
| QD-S-3.4 | Undefined Task Scope | Missing `non_goals`; LLM expands boundaries on its own |
| QD-S-3.5 | Undeclared Default Behavior | Critical branches (empty input, format error, insufficient data) have no defined handling |
| QD-S-3.6 | Metadata Expression Defects | `name`/`description` broad, inaccurate, or inconsistent with the body |
| QD-S-3.7 | Output Structure Contract Missing | Output format / required fields / empty-result representation not declared; LLM self-selects structure. (1.2 covers size, 2.3 covers open enumeration — this item covers the output structure contract.) |
| QD-S-4.1 | Undefined Failure Behavior | No handling for execution failure; LLM explores alternative paths |
| QD-S-4.2 | Undeclared Partial Completion Legitimacy | Whether "partial completion is acceptable" not declared |
| QD-S-4.3 | Missing Post-Failure Prohibited Behaviors | Not stated what is prohibited after failure |
| QD-S-4.4 | Error Recovery Path Undefined | Per-exception recovery paths not defined; "safe-terminate-and-report" is a legitimate disposition |
| QD-S-5.1 | Read/Write/Delete Not Distinguished | Operating on data without distinguishing read/modify/delete scopes |
| QD-S-5.2 | Missing High-Risk Confirmation | Irreversible operations (delete/overwrite/exfiltrate/pay) have no confirmation gate |
| QD-S-5.3 | Undeclared Environment Boundaries | Not declared whether production is prohibited / which environments are permitted |
| QD-S-5.4 | Undeclared Chained Invocation Permissions | Invoking other Skills without declaring permitted/prohibited downstream scope |

## 6. Standard inspection flow (§7.3, six steps)

1. **Baseline Compliance Check** — run QD-S-0 (0.1–0.9). Only a completely unparseable Skill (0.3 fails) stops; otherwise mark "baseline not passed" + "assessment incomplete (NEEDS_REVIEW)" and continue.
2. **Skill Risk Level Determination** — L1/L2/L3 per §7.3.2; if tool/permission context unavailable, output "tier undetermined".
3. **Select Inspection Checklist** — L1 = QD-S-0 + 8 items; L2 = 5 P0 + 17 P1 + 1 P2; L3 = full (24 items).
4. **Automated Inspection** — QD-S-0.1–0.9, quantity-boundary keyword scanning (1.1–1.3), unbounded-language detection (2.1–2.3).
5. **Semi-Automated and Human Inspection** — QD-S-3.x / 4.x / 5.x via the six-dimension Review Schema + inferential verification.
6. **Output Inspection Report** — defect list sorted by priority + (full spec) optimized Skill and comparison; the lite inspector is detect-only (never emits an optimized artifact).

## 7. Review Schema (Part 8) — six review dimensions

Organize the final Skill report's findings under these dimensions.

| Dimension | Review Question | Associated Defect Types |
| --- | --- | --- |
| **IDENTITY** | What does this Skill do, not do, and what can it access? | QD-S-3.4, QD-S-5.1 |
| **TRIGGER** | When should it trigger, and when should it not? | QD-S-3.1, QD-S-3.2 |
| **INPUT** | Are input size, format, and overflow behavior clearly defined? | QD-S-1.1, QD-S-3.3 |
| **EXECUTION** | Are execution boundaries, retries, tool invocations, and chained calls controlled? | QD-S-1.3, QD-S-5.4 |
| **OUTPUT** | Are output size, structure, and partial completion strategy clearly defined? | QD-S-1.2, QD-S-2.3, QD-S-3.7 |
| **FAILURE** | How are failures, empty input, insufficient input, and user dissatisfaction handled? | QD-S-4.x |

Review items per dimension (MUST = gap-analysis core; source §8.2–8.7):

- **IDENTITY**: purpose (MUST), non_goals (MUST), allowed_resources (MUST), forbidden_resources (MUST), forbidden_operations (MUST), allowed_operations (SHOULD).
- **TRIGGER**: allowed_when (MUST), forbidden_when (MUST), ambiguity_policy (MUST — ask_user / stop / infer), may_expand_scope (MUST).
- **INPUT**: max_items (MUST), if_exceeds (MUST — process_first_n / reject / ask), max_chars (SHOULD), format (SHOULD), safe_defaults (SHOULD).
- **EXECUTION**: max_tool_calls (MUST), max_retries (MUST), timeout_seconds (MUST), allow_task_expansion (MUST), if_budget_exceeded (MUST), allow_skill_chaining (SHOULD), allowed_downstream_skills (MUST when chaining=true), require_user_confirmation_for (MUST when irreversible operations are involved).
- **OUTPUT**: max_total_chars (MUST), allow_additional_sections (MUST), partial_result_allowed (MUST), if_input_insufficient_for_section (MUST), format (SHOULD), max_items_per_section (SHOULD).
- **FAILURE**: on_empty_input (MUST — ask_user / stop / generate_template), on_insufficient_input (MUST — generate_partial / ask_user / stop), forbidden_behaviors (MUST — no auto-completion, no inference, no scope expansion), on_user_dissatisfied (SHOULD — rewrite limits, escalation confirmation).

## 8. Scoring note (Part 11, optional)

**Fixed deduction** (v1.1): P0 = 25, P1 = 15, P2 = 5 (keeps the 5:3:1 ratio; does not vary with item count). Score = max(100 − total deductions, 0); **if any P0 exists, cap the score at 60** and Gate = FAIL. **Same-source deduplication** (§11.6): one root cause / one harm path is charged once even if it matches multiple rule IDs. **Comparability**: scores are for same-tier longitudinal comparison only — never cross-tier.

| Level | Semantic items | P0 | P1 | P2 |
| --- | --- | --- | --- | --- |
| **L1** | 8 | 0 | 7 | 1 |
| **L2** | 23 | 5 | 17 | 1 |
| **L3** | 24 | 17 | 6 | 1 |

Gate is an Inspect-local conclusion (not a framework-level release decision); unchecked/insufficient-evidence items are not counted as pass. Compute an indicative score only if the user asks, and label it "indicative (lite minimal set), not the full-spec score".
