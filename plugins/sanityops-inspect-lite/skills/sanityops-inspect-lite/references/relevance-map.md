---
title: Relevance Map (Lite Inspector)
description: "Candidate impact outlook for unfixed Inspect defects — per-rule QD to candidate attack surface (AS-*) and candidate failure mode (FM-*) mapping, condensed from framework/relevance.md v1.1. Defines no new rules or mappings."
---

> **Provenance**: Condensed from `framework/relevance.md` v1.1 (SanityOps Framework, CC BY-SA 4.0). This file is a compressed extraction for the sanityops-inspect lite inspector and defines NO new rules or mappings. relevance.md is an L2 cross-domain mapping document subordinate to the nine L1 sub-specifications; on any conflict (numbering, definitions, severity, release requirements), the source sub-specification governs.
>
> Source rule (relevance.md §0.3.1): "If there is a conflict between this specification and a source sub-specification regarding rule numbering, definitions, severity levels, or release requirements, **the corresponding source sub-specification shall take precedence**." The mapping strength, attack surfaces, candidate failure modes, validation recommendations, and diagnostic recommendations in the source do not alter the original determinations of the source sub-specifications.

## How to use

Use this file in **Stage F (Impact outlook)** only, after Stages A/B/D have produced actual FAIL findings. For **each FAIL** (and only FAILs), look up the QD ID in the tables below and emit a condensed Impact Card (template at the end of this file).

**Mandatory red lines:**

- **Map only defects that actually FAILED inspection.** Never pre-map, speculatively map, or map remediated/PASS items.
- **Candidate language only.** Every output line must read as a *candidate association*: use "may", "candidate association", "needs dynamic verification". **Never** write causal claims: "will cause", "guarantees", "ensures", "inevitably", "proves". Source principle (relevance.md §0.4-1, "Correlation is not causation"): "A `QD` defect indicates missing, ambiguous, inconsistent, or insufficiently bounded definitions; it does not automatically prove that an attack has succeeded, a runtime vulnerability exists, or a quality failure has occurred."
- **Strength is NOT severity.** Mapping strength describes association confidence / test-priority only (§1.3) and never replaces P0/P1/P2. Label both explicitly whenever a strength is emitted.
- **State what was NOT run.** Every impact section must explicitly record that the Risk subsets (Risk Explicit and Risk Implicit) were **NOT executed** by the lite inspector — no `EX` classification exists and dynamic exploitability is unverified — and that per relevance.md §1.4 no assumption may be made that controls outside the artifacts are absent.
- **Never convert a candidate association into a definitive root-cause claim** — not before, and especially not after, a quality failure (reverse diagnostics yield candidate investigation directions only).
- **Keep the impact outlook physically separated from the Gate/verdict conclusions.** It is advisory context, not a verdict; it must not change, soften, or escalate any Inspect conclusion.

## Mapping strength (relevance.md §1.3)

Source definition sentence: "Mapping strength reflects 'whether validation or supplementary testing should be prioritized', not vulnerability severity, nor does it replace P0/P1/P2."

| Strength | Meaning (verbatim, §1.3) | Platform behavior in the full framework |
|----------|--------------------------|------------------------------------------|
| **Strong** | Defect directly expands attack entry, or highly likely to cause specified failure mode | Automatically recommend related Risk or Quality test cases |
| **Medium** | May have impact under specific permission, data, Tool, or process conditions | Recommend based on applicability conditions |
| **Conditional** | Requires external knowledge bases, runtime environments, model behaviors, or business rules | As candidate investigation direction, no automatic conclusion |
| **Not Applicable** | Current Agent type, artifact type, or runtime conditions lack association prerequisites | Do not output this recommendation |

**Strength assignment rule for the lite inspector:** the source Appendix A now carries **per-row strength ratings** (`Strong` / `Medium` / `Conditional`); the tables below reproduce them. When emitting an Impact Card, copy the row's strength and label it "provisional (lite)". Never present a strength as severity.

## Attack surface legend (verbatim, relevance.md §2.2)

| ID | Attack Surface | Definition |
|----|----------------|------------|
| `AS-01` | Instruction and Context Boundary Confusion | Mistaking untrusted input, documents, history content, or external returns as high-priority instructions. |
| `AS-02` | Input and Parameter Injection | Unconstrained user or external input entering Tools, backends, or downstream interpreters. |
| `AS-03` | Output Handling and Sensitive Data Exfiltration | Sensitive information being output, concatenated, referenced, forwarded, or written to external destinations. |
| `AS-04` | Permission, Identity, and Delegation Expansion | Broad capabilities, incorrect authorization, or chained calls leading to out-of-scope operations. |
| `AS-05` | Tool Misuse and High-Risk Side Effects | Wrong Tool selection, accidental write operations, batch operations, or unconfirmed irreversible actions. |
| `AS-06` | Resource Exhaustion and Runaway Execution | No upper limits on input, output, calls, retries, or recursion leading to resource loss of control. |
| `AS-07` | Anomaly Propagation and Cascading Failures | After Tool or dependency failure, Agent adopts undefined expansion, retry, or degradation strategies. |
| `AS-08` | Configuration, Naming, and Supply Chain Confusion | Similar Tool names, Schema changes, dangerous default values, or abnormal callback addresses leading to misuse. |
| `AS-09` | Cross-Artifact Contract Breach | Inconsistencies in authorization, parameters, output, permissions, or error strategies across Prompt, Skill, and Tool. |

Impact-dimension codes used below: `D1` = Confidentiality Harm, `D2` = Integrity Harm, `D3` = Authorization Harm (relevance.md §0.5). "Availability", "Auditability", "Integrity" appear as plain words in the source tables.

**Source caveat (relevance.md Appendix A preamble, condensed):** "This appendix defines **mapping directions**; it does not automatically determine static defects as EX risks, dynamic vulnerabilities, or actual security incidents." Risk Explicit content only indicates suggested priority review directions; specific `EX` classification must be independently determined by Risk Explicit.

## A.1 Prompt Defect Mapping (QD-P-*)

| QD ID | Name | Applies | AS | Impact | Strength | Validation (condensed) |
|-------|------|---------|-----|--------|----------|------------------------|
| `QD-P-1.1.1` | Direct Contradiction | AC-L1~3 | AS-01, AS-05 | D2, D3 | Strong | Conflicting instructions, rule priority, examples overriding body, multi-turn |
| `QD-P-1.1.2` | Priority Conflict | AC-L1~3 | AS-01, AS-05 | D2, D3 | Strong | Rule priority, security-vs-task conflict, multi-turn |
| `QD-P-1.1.3` | Example-Rule Contradiction | AC-L1~3 | AS-01 | D2, D3 | Strong | Examples overriding body rules, multi-turn |
| `QD-P-1.2.1` | Role-Responsibility Mismatch | AC-L1~3 | AS-04, AS-09 | D2, D3, Availability | Strong | Out-of-role tasks, responsibility-constraint conflict, handoff/refusal |
| `QD-P-1.2.2` | Role-Output Style Mismatch | AC-L1~3 | AS-03 | Auditability | Medium | Output style, tone-role match |
| `QD-P-1.2.3` | Constraint-Responsibility Mismatch | AC-L1~3 | AS-04, AS-09 | D2, D3 | Strong | Responsibility-constraint conflict, handoff/refusal |
| `QD-P-1.2.4` | Identity & Anthropomorphism Boundary | AC-L1~3 | AS-01, AS-04 | D1, D3 | Medium | Identity claims, anthropomorphic tone, over-trust, escalation |
| `QD-P-1.3.1` | Quantifier Misuse | AC-L1~3 | AS-01, AS-04 | D2, D3 | Medium | Vague quantifier variants, adjacent permission/boundary |
| `QD-P-1.3.2` | Vague Qualifiers | AC-L1~3 | AS-01, AS-04 | D2, D3 | Medium | Qualifier boundaries, adjacent permission/boundary |
| `QD-P-1.3.3` | Undefined Boundaries | AC-L1~3 | AS-01, AS-04 | D1, D2, D3 | Strong | Adjacent permission, scope expansion |
| `QD-P-1.4.1` | Redundant Constraints (recommendation) | AC-L1~3 | AS-06 | Availability, Auditability | Conditional | Duplicate-rule consistency, token budget |
| `QD-P-1.4.3` | Invalid Skill Definitions | AC-L2/3 | AS-05, AS-09 | D2, D3, Availability | Strong | Invalid Skill refs, unreachable branches, dead rules |
| `QD-P-1.5.1` | Responsibilities Not Supporting Objective | AC-L1~3 | AS-05, AS-07, AS-09 | D2, D3, Availability | Strong | Goal completion, workflow dependency, exception path |
| `QD-P-1.5.2` | Resources Not Supporting Workflow | AC-L2/3 | AS-05, AS-07, AS-09 | D2, D3, Availability | Strong | Resource reachability, workflow dependency, exception path |
| `QD-P-1.6.1` | Critical Decision Word Vagueness | AC-L1~3 | AS-01, AS-04 | D1, D2, D3 | Strong | "sensitive"/"necessary"/"reasonable" boundary variants |
| `QD-P-1.6.2` | Rules Depending on Model Internal State | AC-L1~3 | AS-01, AS-07 | D1, D2, D3, Availability | Conditional | Uncertainty/intent/confidence rule stability |
| `QD-P-1.6.3` | Soft Rule Interweaving Without Arbitration | AC-L1~3 | AS-01, AS-07 | D1, D2, D3, Availability | Strong | Soft-rule conflict, no arbitration, multi-turn |
| `QD-P-1.6.4` | Reasoning Method Constraints | AC-L1~3 | AS-01 | Availability | Conditional | Reasoning-constraint followability, cross-turn |
| `QD-P-1.6.5` | Self-Referential/Recursive Constraints | AC-L1~3 | AS-06, AS-07 | Availability | Medium | Recursion termination, self-reference, resource exhaustion |
| `QD-P-1.6.6` | Implicit Assumption Dependency | AC-L1~3 | AS-01, AS-07 | D2, Availability | Conditional | Implicit premises, knowledge update, abnormal conditions |
| `QD-P-1.6.7` | Non-Monotonic Reasoning Defects | AC-L1~3 (single-turn N/A) | AS-07 | D2, Availability | Conditional | Cross-turn state, committed conclusions, new-information revision |
| `QD-P-1.6.8` | Rule Followability | AC-L1~3 | AS-01 | Availability | Conditional | Counting/timing/arithmetic constraint stability |
| `QD-P-2.1.2` | Reference Clarity | AC-L2/3 | AS-09 | Availability, Auditability | Medium | Reference resolution, format parsing, constraint reachability |
| `QD-P-2.1.3` | Structural Hierarchy (recommendation) | AC-L1~3 | AS-01 | Availability, Auditability | Conditional | Paragraph length, hierarchy depth, constraint reachability |
| `QD-P-2.1.5` | Context Budget Squeeze (recommendation) | AC-L1~3 | AS-06 | Availability | Conditional | Context budget, key-constraint crowding |
| `QD-P-2.2.1` | Undeclared Input Source | AC-L1~3 (self-evident N/A) | AS-01, AS-03 | D1, D2, D3 | Strong | Source confusion (user/history/external/tool return) |
| `QD-P-2.2.2` | Unconstrained Input Format | AC-L1~3 | AS-02, AS-07 | D2, D3, Availability | Strong | Null, malformed, missing fields, encoding variants |
| `QD-P-2.2.3` | Required/Optional Not Distinguished | AC-L1~3 (free-text N/A) | AS-02, AS-07 | D2, D3, Availability | Strong | Missing fields, default manipulation, encoding variants |
| `QD-P-2.2.4` | Undefined Missing-Input Handling | AC-L1~3 | AS-02, AS-07 | D2, D3, Availability | Strong | Null, missing fields, default manipulation |
| `QD-P-2.3.1` | Undefined Output Format | AC-L1~3 (machine-consumed) | AS-03, AS-09 | D1, D2, Auditability | Strong | Output parsing, required fields, format bypass |
| `QD-P-2.3.2` | Incomplete Output Schema | AC-L1~3 (structured-output N/A) | AS-03, AS-09 | D1, D2, Auditability | Strong | Structured output, schema validation, required fields |
| `QD-P-2.3.4` | Missing Security/Privacy Filtering | AC-L1~3 (else N/A) | AS-03 | D1 | Conditional | PII requests, prompt extraction, sensitive output, exfiltration |
| `QD-P-2.3.5` | Factuality & Hallucination Control | AC-L2/3 (no retrieval/tool N/A) | AS-03 | D2, Auditability | Conditional | Groundless fabrication, hallucination, factuality regression |
| `QD-P-2.3.6` | Output Length & Truncation | AC-L1~3 | AS-03 | D2, Availability | Medium | Overlong output, max length, truncation integrity |
| `QD-P-2.3.7` | Output Localization & Conventions | AC-L1~3 (single-region N/A) | AS-03 | D2, Auditability | Conditional | Language/timezone/currency/unit/percentage |
| `QD-P-2.4.1` | Incomplete Workflow Steps | AC-L2/3 | AS-05, AS-07, AS-09 | D2, D3, Availability | Strong | Step order, preconditions, missing steps, fallback |
| `QD-P-2.4.2` | Undeclared Resources/Tools | AC-L2/3 | AS-05, AS-09 | D2, D3, Availability | Strong | Undeclared tool/resource, call-path reachability |
| `QD-P-2.4.3` | Tool Invocation Parameter/Order Error | AC-L2/3 | AS-05, AS-09 | D2, D3, Availability | Medium | Parameter construction, call order, fallback |
| `QD-P-2.4.4` | Multi-turn Context & Memory Boundary | AC-L2/3 (stateless N/A) | AS-01, AS-07 | D1, D2, D3 | Medium | Multi-turn state pollution, memory boundary, cross-turn |
| `QD-P-2.5.1` | Missing Termination Constraints | AC-L2/3 | AS-06 | Availability | Strong | Loop, recursion, retry, timeout, budget |
| `QD-P-2.5.2` | Missing Security Boundary Constraints | AC-L1~3 | AS-01, AS-03, AS-04 | D1, D2, D3 | Strong | Injection, indirect injection, goal hijack, prompt extraction |
| `QD-P-2.5.3` | Missing Permission Control Boundaries | AC-L2/3 | AS-04, AS-05 | D2, D3 | Strong | Role escalation, cross-user/tenant, read/write/delete |
| `QD-P-2.5.4` | Undefined Constraint Priority | AC-L2/3 | AS-01, AS-07 | D1, D2, D3 | Strong | Security/efficiency/completion goal conflict |
| `QD-P-2.5.6` | Instruction-Data Trust Boundary | AC-L2/3 | AS-01, AS-02 | D1, D2, D3 | Strong | Instruction/data confusion, injection, goal hijack |
| `QD-P-2.5.7` | High-Risk Operation Human Confirmation | AC-L2/3 (no such op N/A) | AS-05 | D2, D3 | Conditional | High-risk op confirmation, undo, rollback, side effect |
| `QD-P-2.5.8` | System Prompt Confidentiality & Leakage | AC-L2/3 | AS-01, AS-03 | D1 | Strong | Prompt extraction, leakage response, confidentiality |
| `QD-P-2.5.9` | Industry Compliance & Disclaimer | AC-L1~3 (else N/A) | AS-05 | D2, Auditability | Conditional | Medical/financial/legal disclaimers, out-of-scope |
| `QD-P-2.6.1` | Runtime Exceptions Not Covered | AC-L2/3 | AS-07 | D1, D2, D3, Availability | Strong | Timeout, permission denial, rate limit, dependency failure |
| `QD-P-2.6.2` | Missing Degradation & Recovery | AC-L3 | AS-07 | Availability | Medium | Degradation, recovery, partial completion, exception path |
| `QD-P-2.6.3` | Unreasonable Retry Mechanism | AC-L3 | AS-06, AS-07 | Availability | Medium | Retry count, backoff, duplicate execution, idempotency |
| `QD-P-2.6.4` | Out-of-scope Rejection & Escalation | AC-L1~3 | AS-01, AS-04 | D2, D3 | Strong | Rejection template, referral, escalation, out-of-scope |
| `QD-P-2.7.1` | Positive-Example Representativeness (rec.) | AC-L1~3 | AS-01, AS-05 | D2 | Conditional | Positive-example coverage, adversarial regression |
| `QD-P-2.7.2` | Counter-Example Coverage (rec.) | AC-L1~3 | AS-01, AS-05 | D2 | Conditional | Counter-example coverage, dangerous-request testing |
| `QD-P-2.7.3` | Boundary-Case Examples Missing (rec.) | AC-L1~3 | AS-01, AS-05 | D2 | Conditional | Boundary cases, dangerous-request regression |

> **Notes** (relevance.md A.1): recommendation items (ungraded) carry only optimization value and are uniformly rated `Conditional`. Deprecated/deleted/merged rules are **not mapped**: `QD-P-2.5.5` (deprecated), `QD-P-2.1.1` (deleted, injection point folded into 2.5.6), `QD-P-1.4.2` (deleted), `QD-P-2.3.3` (deleted), `QD-P-2.1.4` (merged into 1.4.1).

## A.2 Skill Defect Mapping (QD-S-*)

| QD ID | Name | Applies | AS | Impact | Strength | Validation (condensed) |
|-------|------|---------|-----|--------|----------|------------------------|
| `QD-S-0.1~0.9` | Basic Compliance (entry gate) | OR-L1~3 | AS-09 | Availability, Auditability | Medium | Parsing, field completeness, version/metadata |
| `QD-S-1.1` | Input Boundary Missing | OR-L1~3 (platform budget N/A) | AS-06 | Availability | Strong | Large input, call count, timeout, budget |
| `QD-S-1.2` | Output Size Missing | OR-L1~3 | AS-06 | Availability | Strong | Large output, max length, budget |
| `QD-S-1.3` | Execution Resource Boundary Missing | OR-L1~3 (config evidence → SHOULD/N/A) | AS-06 | Availability | Strong | Timeout, call-cap, budget, loop |
| `QD-S-1.4` | Permission Boundary Missing | OR-L1~3 | AS-04, AS-05 | D2, D3 | Strong | Read/write/delete confusion, dangerous ops, least privilege |
| `QD-S-1.5` | Data Source Boundary Missing | OR-L1~3 | AS-01, AS-03, AS-04 | D1, D3 | Medium | History pollution, cross-user, external resource |
| `QD-S-2.1` | Unbounded Quantity Declaration | OR-L1~3 | AS-06 | Availability | Conditional | Full request, unbounded quantity, budget |
| `QD-S-2.2` | Unterminated Execution Declaration | OR-L1~3 | AS-06 | Availability | Conditional | Loop, until-success, timeout, budget |
| `QD-S-2.3` | Open-Ended Enumeration in Structure | OR-L1~3 | AS-06, AS-07 | Availability, D2 | Medium | Open output structure, enumeration explosion, parsing |
| `QD-S-2.4` | Broad Capability Verbs | OR-L1~3 | AS-04, AS-05 | D2, D3 | Conditional | "manage"/"process"/"fix" semantic expansion |
| `QD-S-3.1` | Vague Trigger Conditions | OR-L1~3 | AS-01, AS-04 | D1, D2, D3 | Medium | Adjacent intent, mixed intent, mis-trigger |
| `QD-S-3.2` | Missing Deactivation Conditions | OR-L1~3 (positive trigger suffices → exempt) | AS-01, AS-04 | D2, D3 | Medium | Negative intent, non-trigger, mis-trigger |
| `QD-S-3.3` | Missing Ambiguous Input Handling | OR-L1~3 | AS-01 | D2 | Medium | Insufficient info, malformed input, fabrication |
| `QD-S-3.4` | Undefined Task Scope | OR-L1~3 | AS-01, AS-05 | D2, D3 | Strong | Goal hijack, implicit task, scope expansion, delegation |
| `QD-S-3.5` | Undeclared Default Behavior | OR-L1~3 | AS-05, AS-09 | D2, Availability | Medium | Empty input, default behavior, wrong routing |
| `QD-S-3.6` | Metadata Expression Defects | OR-L1~3 | AS-08 | D2, Availability | Medium | Similar Skills, name confusion, wrong routing |
| `QD-S-3.7` | Output Structure Contract Missing | OR-L1~3 | AS-03, AS-09 | D2, Availability, Auditability | Medium | Output format, required fields, empty result, downstream parse |
| `QD-S-4.1` | Undefined Failure Behavior | OR-L1~3 | AS-07 | Availability | Strong | Tool failure, exception path, failure behavior |
| `QD-S-4.2` | Undeclared Partial Completion | OR-L1~3 | AS-07 | D2, Availability | Medium | Partial completion, half-done state, consistency |
| `QD-S-4.3` | Missing Post-Failure Prohibited Behaviors | OR-L1~3 | AS-05, AS-07 | D2, D3 | Medium | Post-failure prohibited behavior, fallback, side effect |
| `QD-S-4.4` | Error Recovery Path Undefined | OR-L1~3 (no-tool L1 N/A) | AS-07 | Availability | Medium | Recovery, retry, degradation, human takeover |
| `QD-S-5.1` | Read/Write/Delete Not Distinguished | OR-L1~3 | AS-04, AS-05 | D2, D3 | Strong | Read inducing write/delete, permission convergence, side effect |
| `QD-S-5.2` | Missing High-Risk Confirmation | OR-L3 | AS-05 | D2, D3 | Strong | Vague/negative/repeated/revoke confirmation |
| `QD-S-5.3` | Environment Boundary Not Declared | OR-L1~3 (multi-env else N/A) | AS-04, AS-05 | D2, D3 | Medium | Test/prod confusion, production access |
| `QD-S-5.4` | Chained Invocation Permissions Undeclared | OR-L1~3 (chaining=false N/A) | AS-04, AS-05, AS-07 | D1, D2, D3 | Medium | Downstream whitelist, call depth, permission inheritance |

> **Notes** (relevance.md A.2): QD-S-0.x is an entry gate (no P grade, not scored), mapped uniformly to AS-09. Same-source dedup: QD-S-1.4 / 2.4 fold into QD-S-5.1; QD-S-2.1 / 2.2 are the auto-trigger word tables of QD-S-1.1 / 1.3; QD-S-4.3 folds into QD-S-4.1.

## A.3 Tool Defect Mapping (QD-T-*)

| QD ID | Name | Applies | AS | Impact | Strength | Validation (condensed) |
|-------|------|---------|-----|--------|----------|------------------------|
| `QD-T-1.1` | Base Field Missing | OR-L1~3 | AS-05, AS-09 | D2, Availability | Medium | Tool selection, schema parsing, required fields, version compat |
| `QD-T-1.2` | Required References Undefined Field | OR-L1~3 | AS-02, AS-09 | D2, Availability | Medium | required refs, required fields, schema parse, version compat |
| `QD-T-1.3` | Enum type conflict / duplicate values | OR-L1~3 | AS-02, AS-09 | D2, Availability | Medium | enum/type conflict, duplicates, schema validation |
| `QD-T-1.4` | additionalProperties Policy Inappropriate | OR-L1~3 | AS-02, AS-04 | D1, D2, D3 | Strong | Extra/reserved fields, role/tenant/token field injection |
| `QD-T-1.5` | name Validity & Uniqueness | OR-L1~3 | AS-05, AS-08 | D2, Availability | Medium | Name validity, uniqueness, similar-name confusion, routing |
| `QD-T-1.6` | Constraint Self-Consistency & default | OR-L1~3 | AS-02, AS-09 | D2, Availability | Medium | default/constraint consistency, schema validation, default manipulation |
| `QD-T-1.7` | Type Missing / Over-Broad | OR-L1~3 | AS-02, AS-09 | D2, Availability | Medium | type missing/over-broad, type confusion, schema parse |
| `QD-T-2.1` | String Without maxLength | OR-L1~3 | AS-02, AS-06 | D2, D3, Availability | Medium | Overlong string, length boundary, resource exhaustion |
| `QD-T-2.2` | String Format Contract Missing | OR-L1~3 | AS-02, AS-07 | D2, D3, Availability | Medium | Format, encoding variants, malformed input |
| `QD-T-2.3` | Number/Integer Without Boundary | OR-L1~3 | AS-02, AS-06 | D2, D3, Availability | Medium | Numeric boundary, overflow, zero, extremes |
| `QD-T-2.4` | Array Without maxItems | OR-L1~3 | AS-02, AS-06 | D2, D3, Availability | Medium | Array length, batch elements, extremes |
| `QD-T-2.5` | Unit/Precision/Timezone/Currency Missing | OR-L1~3 | AS-02, AS-09 | D2, Availability | Medium | Unit/currency/precision/timezone, end-to-end consistency |
| `QD-T-2.6` | Query Without Pagination / Cap | OR-L1~3 | AS-06 | D2, Availability | Medium | Pagination, result cap, full return, budget |
| `QD-T-2.7` | Parameter Dependency / Mutual Exclusion Unexpressed | OR-L1~3 | AS-02, AS-07 | D2, Availability | Medium | Dependency, mutual exclusion, combined conditions, conflicting fields |
| `QD-T-3.1` | State Change & Material Side Effect Undeclared | OR-L2/3 | AS-05 | D2, D3 | Strong | State change, write mis-selection, confirmation, rollback, idempotency |
| `QD-T-3.2` | Untrusted Input into Executable Context | OR-L1~3 | AS-02 | D1, D2, D3 | Strong | Parameter injection, encoding bypass, parameter pollution, passthrough |
| `QD-T-3.3` | Batch / Range Write Without Cap | OR-L2/3 | AS-05, AS-06 | D2, Availability | Medium | Max batch, batch bypass, cancel, interruption, duplicate submit |
| `QD-T-3.4` | Authorization Context Parameter Exposure | OR-L1~3 | AS-04 | D1, D2, D3 | Strong | tenantId/isAdmin/role/token/control-flag tampering |
| `QD-T-3.6` | Dangerous Default / Scope Precondition | OR-L2/3 | AS-05 | D2, D3 | Medium | Dangerous default, scope precondition, default-behavior trigger |
| `QD-T-3.7` | Parameterized-Tool Operation Whitelist Undeclared | OR-L1~3 | AS-02, AS-04, AS-05 | D1, D2, D3 | Strong | Whitelist-bypass injection, arbitrary SQL/command, privileged table access |
| `QD-T-4.1` | Insufficient Description Quality | OR-L1~3 | AS-05, AS-08 | D2, D3, Availability | Medium | Similar Tools, near-synonym, ambiguous task, malicious selection guidance |
| `QD-T-4.4` | Parameter-Level description Missing | OR-L1~3 | AS-05, AS-08 | D2, Availability | Medium | Parameter semantics, similar-parameter confusion, parameter construction |
| `QD-T-4.5` | description Inconsistent with Schema | OR-L1~3 | AS-05, AS-08 | D2, Availability | Medium | description/schema consistency, parameter construction, schema validation |
| `QD-T-4.6` | Sensitive Input Handling Undeclared | OR-L2/3 | AS-03 | D1 | Conditional | Sensitive input, PII, credential handling, exfiltration |
| `QD-T-4.7` | Output Contract Missing (outputSchema) | OR-L2/3 | AS-03, AS-09 | D1, D2, Availability | Medium | Output structure, required fields, downstream parse |
| `QD-T-5.1` | Breaking Change Determination | change-check | AS-08, AS-09 | D2, D3, Availability | Conditional | Old/new schema replay, caller compat, migration, rollback |
| `QD-T-5.2` | Change Impact Assessment | change-check | AS-08, AS-09 | D2, D3, Availability | Conditional | Old/new schema replay, caller compat, migration, rollback |
| `QD-T-5.3` | Semantic Change | change-check | AS-08, AS-09 | D2, D3, Availability | Conditional | Semantic change, old/new replay, migration/rollback |

> **Notes** (relevance.md A.3): transferred rules (cross-Tool semantics → Cross namespace): `QD-T-3.5` → `QD-TT-5.1`; `QD-T-4.3` → `QD-TT-5.2`. Merged: `QD-T-4.2` → folded into `QD-T-3.1`. `QD-T-5.x` are change-check items not yet wired into checklist/scoring/Gate, uniformly rated `Conditional`.

## A.4 Cross Defect Mapping (QD-PS-*/QD-PT-*/QD-ST-*/QD-CYC-*/QD-SS-*/QD-TT-*)

### QD-PS — Prompt → Skill (7)

| QD ID | Name | Applies | AS | Impact | Strength | Validation (condensed) |
|-------|------|---------|-----|--------|----------|------------------------|
| `QD-PS-1.1` | Skill Authorization Consistency | AC-L1~3 | AS-04, AS-09 | D2, D3 | Strong | Prompt→Skill unauthorized call, name confusion, capability expansion |
| `QD-PS-1.2` | Trigger Condition Consistency | AC-L2/3 | AS-01, AS-05 | D2, D3 | Medium | Trigger, non-trigger, adjacent intent, mixed intent |
| `QD-PS-1.3` | Permission Boundary Consistency | AC-L1~3 | AS-04, AS-09 | D2, D3 | Strong | Prompt→Skill permission, escalation, least privilege |
| `QD-PS-1.4` | Resource Access Consistency | AC-L2/3 | AS-03, AS-04 | D1, D2, D3 | Medium | Prompt→Skill resource source, cross-tenant, data leak |
| `QD-PS-1.5` | Failure Handling Consistency | AC-L3 | AS-07, AS-09 | D2, Availability | Medium | End-to-end failure, recovery, exception path |
| `QD-PS-1.6` | Output Constraint Consistency | AC-L3 | AS-03, AS-09 | D1, D2, Auditability | Medium | Output format, data leak, downstream parse |
| `QD-PS-1.7` | Orchestration Order & Precondition Consistency | AC-L2/3 | AS-05, AS-09 | D2, Availability | Medium | Step order, preconditions, missing step, fallback |

### QD-PT — Prompt → Tool (7)

| QD ID | Name | Applies | AS | Impact | Strength | Validation (condensed) |
|-------|------|---------|-----|--------|----------|------------------------|
| `QD-PT-2.1` | Tool Existence | AC-L1~3 | AS-05, AS-09 | D2, Availability | Strong | Prompt→Tool name, existence, call-path reachability |
| `QD-PT-2.2` | Invocation Specification Consistency | AC-L1~3 | AS-02, AS-09 | D2, Availability | Strong | Prompt→Tool call spec, parameter & call-order consistency |
| `QD-PT-2.3` | Parameter Constraint Consistency | AC-L2/3 | AS-02, AS-09 | D2, Availability | Medium | Prompt→Tool parameter constraints, schema validation, compatibility |
| `QD-PT-2.4` | Permission Granularity Consistency | AC-L1~3 | AS-04, AS-05 | D2, D3 | Strong | High-risk Tool call, confirmation, permission, side effect |
| `QD-PT-2.5` | Key Error Handling Coverage | AC-L3 | AS-07 | D2, Availability | Medium | Rate limit, timeout, permission error, error classification |
| `QD-PT-2.6` | Business-Level Frequency Expectation | AC-L3 | AS-06 | Availability | Medium | Call frequency, batch, budget, duplicate execution |
| `QD-PT-2.7` | Parameter Semantics & Unit Consistency | AC-L2/3 | AS-02, AS-09 | D2, Availability | Medium | Unit/currency/precision/timezone, end-to-end semantics |

### QD-ST — Skill → Tool (7)

| QD ID | Name | Applies | AS | Impact | Strength | Validation (condensed) |
|-------|------|---------|-----|--------|----------|------------------------|
| `QD-ST-3.1` | Tool Existence | AC-L1~3 | AS-05, AS-09 | D2, Availability | Strong | Skill→Tool name, existence, call-path reachability |
| `QD-ST-3.2` | Input Constraint Consistency | AC-L2/3 | AS-02, AS-09 | D2, Availability | Medium | Skill→Tool input constraint, schema validation, boundary |
| `QD-ST-3.3` | Required Parameter Preparation | AC-L2/3 | AS-02, AS-09 | D2, Availability | Medium | Skill→Tool parameter source, required fields, schema validation |
| `QD-ST-3.4` | Business-Level Frequency Expectation | AC-L3 | AS-06 | Availability | Medium | Max call count, batch, budget |
| `QD-ST-3.5` | Output Structure Match | AC-L3 | AS-03, AS-09 | D1, D2, Auditability | Medium | Tool return structure, downstream parse, citation/format |
| `QD-ST-3.6` | Permission Scope Match | AC-L1~3 | AS-04, AS-05 | D2, D3 | Strong | Skill→Tool actual side effects, read/write/delete, confirmation |
| `QD-ST-3.7` | Key Error Handling Coverage | AC-L3 | AS-07, AS-09 | D1, D2, Availability | Medium | Tool error classification, degradation, stop, retry, human takeover |

### QD-CYC — Circular Invocation (3)

| QD ID | Name | Applies | AS | Impact | Strength | Validation (condensed) |
|-------|------|---------|-----|--------|----------|------------------------|
| `QD-CYC-1.1` | Cross-Artifact Circular Invocation | AC-L2/3 | AS-06 | Availability | Medium | Loop termination, cross-artifact cycle, resource exhaustion |
| `QD-CYC-1.2` | Multi-Agent Circular Invocation | AC-L3 | AS-06 | Availability | Medium | Multi-Agent cycle, loop termination, resource exhaustion |
| `QD-CYC-1.3` | Inter-Skill Handoff Loop | AC-L2/3 | AS-06 | Availability | Medium | Handoff termination, max rounds, downstream-whitelist closure |

### QD-SS — Skill ↔ Skill (2)

| QD ID | Name | Applies | AS | Impact | Strength | Validation (condensed) |
|-------|------|---------|-----|--------|----------|------------------------|
| `QD-SS-4.1` | Trigger Condition Conflict | AC-L2/3 | AS-01, AS-05 | D2, D3 | Medium | Overlapping triggers, adjacent/negative intent, Skill priority |
| `QD-SS-4.2` | Name Uniqueness / Near-Misleading | AC-L2/3 | AS-05, AS-08 | D2, D3 | Medium | Same-name, spelling/plural/synonym near-identity mis-invocation |

### QD-TT — Tool ↔ Tool (3)

| QD ID | Name | Applies | AS | Impact | Strength | Validation (condensed) |
|-------|------|---------|-----|--------|----------|------------------------|
| `QD-TT-5.1` | Cross-Tool Data Flow | AC-L2/3 | AS-03, AS-05 | D1, D2, D3 | Strong | Cross-Tool data flow, exfiltration, end-to-end consistency |
| `QD-TT-5.2` | Multi-Tool Semantic Competition | AC-L2/3 | AS-05, AS-08 | D2, D3 | Strong | Similar Tools, near-synonym request, mis-selection |
| `QD-TT-5.3` | Cross-Tool Data Structure Bridging | AC-L3 | AS-03, AS-09 | D2, Availability | Medium | Cross-Tool structure bridging, downstream parse, schema consistency |

> **Notes** (relevance.md A.4): QD-PS-1.6 is same-source with QD-ST-3.5 (primary: QD-ST-3.5). QD-TT-5.1 / QD-TT-5.2 are the Cross-namespace successors of QD-T-3.5 / QD-T-4.3 — the original Tool namespace no longer maps them.

## A.5 Permission Defect Mapping (QD-PM-*)

> Source note (condensed): `QD-PM-*` are produced by the Inspect Permission Governance subset (permission v1.0). This table only defines mapping directions and does not automatically determine permission defects as EX risks or actual events; `EX` classification is still independently determined by Risk Explicit. The Permission subset executes after other subsets and is subject to Gate-0 preconditions; when Gate-0 is not met, no QD-PM defects are produced, and this table does not apply.

| QD ID | Name | Applies | AS | Impact | Strength | Validation (condensed) |
|-------|------|---------|-----|--------|----------|------------------------|
| `QD-PM-1.1~1.7` | Permission Alignment (1.7 fixed P2) | AC-L1~4 (1.6 conditional) | AS-04 | D2, D3, Availability | Strong | Out-of-role resource access, aggregated escalation, insufficient-permission workaround |
| `QD-PM-2.1~2.8` | Permission Scope (resource/data/env/condition axes) | AC-L1~4 (2.5/2.6/2.7 conditional) | AS-04, AS-03 | D1, D2, D3 | Strong | Out-of-scope resource, cross-tenant read, test/prod confusion, out-of-condition trigger |
| `QD-PM-3.1~3.6` | Permission Granularity (3.3 role substitution, 3.4 fixed P0) | AC-L1~4 (3.4/3.5/3.6 conditional) | AS-04, AS-05 | D2, D3 | Strong | Single-role init+approve, high-risk with routine authority, role impersonation |
| `QD-PM-4.1~4.8` | Permission Propagation (4.2/4.4/4.8 + INHERITED) | AC-L1~4 (4.3~4.6 AC-L4; 4.7 AC-L2+; 4.8 conditional) | AS-04, AS-07, AS-08 | D1, D2, D3 | Strong | Call-chain inheritance, downstream vs upstream, delegation escalation, external source |
| `QD-PM-5.1~5.7` | Permission Boundary (5.4 fixed P0, 5.7 exemption) | AC-L1~4 (5.4 unconditional; 5.5/5.7 conditional) | AS-04, AS-05 | D2, D3 | Strong | Exemption trigger/expiry, isolation-substitution bypass |
| `QD-PM-6.1~6.7` | Permission Auditability (6.3/6.5 fixed P0, 6.7 fixed P1) | AC-L1~4 (6.6 AC-L2+; others conditional) | AS-05, AS-09 | D2, D3, Auditability | Medium | High-risk op without confirmation, non-enumerable inventory, missing audit log |

> **Note** (relevance.md A.5): QD-PM has 43 full items, mapped here in 6 groups. Group-level strength indicates overall test-priority, not member severity (see §1.3).

## Candidate failure modes (FM-*) — from relevance.md Appendix B.1

The source Appendix B.1 provides FM associations at **QD-category-group granularity** (rows group multiple QD IDs). To emit the FM field of an Impact Card:

1. Find the B.1 row below whose category description matches the defect's QD family.
2. Copy that row's FM codes as **candidate** failure modes.
3. If a QD ID does not clearly fall into any B.1 group, write "no category-level FM association defined in Appendix B.1" — do not guess.

FM-* code names are defined in `quality/tool-agent.md` (L1); Appendix B.1 uses FM-01 through FM-10. RAG-Agent metric associations are reproduced in condensed form in the "Candidate RAG-Agent quality metrics" section below (from relevance.md Appendix C.1).

| QD category group (verbatim, B.1) | Candidate failure modes (FM-*) |
|-----------------------------------|--------------------------------|
| Prompt logic contradiction, mismatch, implication defects | FM-01, FM-02, FM-07, FM-09 |
| Prompt input definition defects | FM-01, FM-04 |
| Prompt output definition defects | FM-06 |
| Prompt workflow and resource defects | FM-03, FM-04, FM-05 |
| Prompt boundary, frequency, and termination defects | FM-08, FM-09, FM-10 |
| Prompt exception handling defects | FM-08, FM-10 |
| Prompt example design defects | FM-01, FM-07, FM-09 |
| Skill resource or unbounded declaration defects | FM-08, FM-10 |
| Skill trigger, ambiguity, scope defects | FM-02, FM-07, FM-09 |
| Skill failure expansion defects | FM-08, FM-09 |
| Skill permission and environment boundary defects | FM-05, FM-09 |
| Tool structure and parameter defects | FM-03, FM-04, FM-05 |
| Tool high-risk operation defects | FM-08, FM-09, FM-10 |
| Tool semantic competition defects | FM-03, FM-05 |
| Tool Schema change defects | FM-04, FM-05, FM-06 |
| Cross contract defects | FM-04, FM-05, FM-06, FM-08 |
| Inter-Skill trigger conflict and name similarity (`QD-SS`) | FM-02, FM-03, FM-07 |
| Cross-Tool data structure incompatibility (`QD-TT-5.3`) | FM-05, FM-06 |
| Skill-to-Skill handoff loop (`QD-CYC-1.3`) | FM-08, FM-10 |
| Permission proportionality defects (`QD-PM`) | FM-05, FM-09, FM-10 |

> **Wildcard convention** (relevance.md Appendix B note): category wildcards (e.g. `QD-P-1.x`, `QD-T-4.x`) refer only to **currently active** members, excluding deleted (`QD-P-2.1.1`, `QD-P-1.4.2`, `QD-P-2.3.3`), deprecated (`QD-P-2.5.5`), merged (`QD-T-4.2`), and transferred (`QD-T-3.5`→`QD-TT-5.1`, `QD-T-4.3`→`QD-TT-5.2`) IDs. The authoritative active-member list is Appendix A above.

## Candidate RAG-Agent quality metrics — from relevance.md Appendix C.1

Source boundary (relevance.md §3.3.1): RAG-Agent quality evaluates **user-visible content output and boundary handling**, and does not directly score internal components such as Prompt, retrieval, knowledge base, model, or Tool. The 12 metrics span four dimensions: Knowledge answer quality (Correctness, Completeness, Relevance, Traceability, Timeliness); Dialogue and complex task quality (Consistency, Robustness, Decomposition Capability, Coherence); Expression and compliance quality (Format compliance); Boundary and safety response quality (Boundary recognition, Safety response).

The mapping below is **candidate only**: a QD category "may affect" the listed metrics, meaning it is worth prioritizing supplementary test design or investigation — it does not mean the defect will necessarily cause a metric to lose points (§3.3.1). The lite inspector never runs the Quality subset; emit these as candidate associations, never as confirmed quality outcomes.

| QD Category | Candidate affected metrics | Recommended supplementary tests (condensed) |
|---|---|---|
| Prompt logic contradiction, mismatch, implication defects | Correctness, Relevance, Consistency, Format Compliance | Rule conflicts, role boundaries, multi-constraint Q&A |
| Prompt gray zone defects | Consistency, Robustness, Boundary Recognition, Safety Response | Vague terms, multi-turn follow-ups, conflicting rules, knowledge updates |
| Prompt input definition defects | Relevance, Robustness, Decomposition Capability, Boundary Recognition | Colloquial rewrites, missing conditions, multi-intent, abnormal formats |
| Prompt output definition defects | Completeness, Traceability, Format Compliance, Consistency | Citations, structure, disclaimers, JSON/Markdown |
| Prompt workflow, resource, and Tool defects | Correctness, Traceability, Timeliness | Knowledge source selection, retrieval Tool, citation chain, failure fallback |
| Prompt security, permission, and exception boundary defects | Boundary Recognition, Safety Response, Coherence | Unauthorized, sensitive, no-hit, handoff-required, dependency failure |
| Prompt example design defects | Correctness, Robustness, Consistency, Format Compliance | Positive/negative examples, boundary expressions, adversarial rewrites |
| Skill resource and unbounded declaration defects | Completeness, Timeliness, Robustness | Large input, multi-documents, exceeding limits, partial completion |
| Skill trigger, scope, and ambiguity defects | Relevance, Decomposition Capability, Boundary Recognition | Intent boundaries, non-activation, clarification and handoff |
| Skill failure expansion defects | Completeness, Coherence, Safety Response | No hits, low-confidence sources, source conflicts, retrieval failure |
| Skill permission and environment boundary defects | Boundary Recognition, Safety Response, Traceability | Sensitive data, user identity, production data, external resources |
| Tool parameter and semantic defects | Correctness, Relevance, Traceability, Format Compliance | Retrieval parameters, source return, citation parsing, similar Tools |
| Cross contract defects | Correctness, Completeness, Consistency, Traceability, Safety Response | End-to-end retrieval, citation, error handling, permission, output contracts |

Out of scope (not reproduced): relevance.md Appendix C.2 (RAG failure **reverse** diagnostic — requires actual Quality failures, which the lite inspector never produces) and D.3 (RAG-Agent quality **defect chains**).

## Prohibited automatic inference (verbatim, relevance.md §2.3.1)

"The following inferences are all invalid:"

- `QD-P-2.5.2 Security Boundary Missing` ≠ Automatic hit `NL-A-7` Reverse Restriction Elimination Type
- `QD-T-1.4 additionalProperties Policy Inappropriate` ≠ Automatic hit `NR-S-3` Parameter Type/Constraint Abnormal Declaration
- `QD-PS-1.3 Permission Boundary Consistency Defect` ≠ Automatic hit `NL-B-9` Cross-Object Authorization Chain Forgery

Source closing rule: "Only when an explicit expression meeting the EX definition exists in the artifact can Risk Explicit independently determine the corresponding `EX` classification." The lite inspector never determines `EX` classifications at all (Risk Explicit is not run).

## Impact Card template (condensed from relevance.md §5.5, single-defect lite form)

```markdown
### Impact Card: <QD-ID> | <short defect name> | <P0/P1/P2> (scope)

**Static Discovery**  <!-- confirmed facts -->
- Artifact: <prompt|skill|tool>:<name>
- Version: <version label / hash>
- Evidence: <rule hit and location, one line>

**Potential Security Associations**  <!-- candidate associations, NOT confirmed impact -->
- Candidate Attack Surface: AS-xx <name> [, AS-yy ...]
- Potential Impact: D1 / D2 / D3 / Availability / Integrity / Auditability
- Mapping Strength: <Strong|Medium|Conditional — from the A.x table above,
  labeled "provisional (lite)"> — association confidence only, NOT severity
- Note: This conclusion indicates priority for validation; it does not yet indicate that
  the backend is exploitable. (verbatim §5.5)

**Recommended Validation — NOT run by this lite inspector**  <!-- mandatory statements -->
- Risk Explicit: NOT run — no EX classification was or can be determined here (§2.3.1).
- Risk Implicit: NOT run — dynamic exploitability is unverified.
- External controls: presence/absence UNKNOWN; per relevance.md §1.4, no assumption is
  made that compensating controls outside the artifacts are absent.
- If the full framework is adopted later: Risk Explicit review of <Note directions>;
  Risk Implicit testing of <Note directions>; required conditions: shadow sandbox,
  controlled test data, call traces, Mock downstream services.

**Potential Quality Associations**  <!-- candidate -->
- Tool-Agent: FM-xx, FM-yy (category-level, from Appendix B.1 table above)
- RAG-Agent: <metrics> (candidate, from the Appendix C.1 table above; NOT a quality result)

**Current Evidence Status**
- Static Defect: Confirmed
- Dynamic Exploitability: Not verified
- Quality Association: Not verified
- Compensating Control: To be confirmed

**Remediation and Regression**
- Recommendation: <per the owning L1 sub-specification>
- Minimum Regression: Inspect re-scan of the updated artifact passing the same rule ID;
  associated Risk/Quality test cases only when those subsets are actually run.
```

Report wording (verbatim, relevance.md §5.5.1):

| Permissible Wording | Prohibited Wording |
|---------------------|--------------------|
| "May expand the attack surface" | "Has already been attacked" |
| "Recommended to prioritize validation" | "Vulnerability necessarily exists" |
| "Candidate quality root cause" | "Quality failure is caused by this defect" |
| "Relies on compensating controls" | "Completely secure" |
| "Not reproduced under current test conditions" | "Absolutely unexploitable" |

## Defect chains — out of scope (relevance.md §1.5)

The framework also defines defect chain mapping: "Defect chains are used to identify candidate exploitation or failure paths formed by multiple defects in combination" (e.g., a boundary deficiency combined with insufficient permission declaration and a high-risk Tool capability). Chains require a unique `chain_id`, retention of member defects and cross-artifact relationships, prioritized end-to-end Risk Implicit validation, coordinated remediation, and full-path regression (relevance.md §1.5, Appendix D). **Out of scope for the lite inspector v1**: this reference supports single-defect candidate mapping only — do not construct, assert, or report defect chains (`SC-*` / `QC-TA-*` / `QC-RA-*`), and never upgrade separate single-defect candidate associations into a claimed chain path.
