---
title: Tool Schema Inspection Checklist (Lite)
description: "Condensed QD-T inspection checklist for the sanityops-inspect lite inspector: Tool operation risk tiering (OR-L1/L2/L3), the full QD-T rule set (1.x–5.x, 31 numbered IDs with merges/transfers), per-tier minimum rule sets (20/25/25) with defect levels, and the scoring note. Extracted from inspect/tool.md v1.0 (2026-10 revision); defines no new rules."
---

# Tool Schema Inspection Checklist (QD-T, Lite)

> **Provenance**: Condensed from `inspect/tool.md` v1.0 (2026-10 revision) (SanityOps Framework, CC BY-SA 4.0). This file is a compressed extraction for the sanityops-inspect lite inspector and defines NO new rules. On any conflict, `inspect/tool.md` governs.

## How to use

- **Tier the Tool first** (inspect/tool.md §3.1): read-only query with no state change → **OR-L1**; creates/updates recoverable state → **OR-L2**; irreversible or sensitive operations → **OR-L3**. Determination basis: any write → OR-L2 minimum; irreversible operations → OR-L3; sensitive data → OR-L3. State the determination in the report.
- **Symbol convention (§3.1 note)**: throughout the source, bare `L1/L2/L3` **equal** `OR-L1/OR-L2/OR-L3` (Tool operation risk level) and must never be read as `AC-L` (Agent complexity level). The two models are distinct.
- **Run the tier's minimum rule set** (Appendix A below): OR-L1 = 20 items, OR-L2 = 25, OR-L3 = 25. Sets are cumulative — a higher tier covers all lower-tier items. Must-pass items = the tier's P0 items; P1/P2 items are recommended-pass.
- **Mechanical items**: QD-T-1.x and QD-T-2.x are decided by direct schema reading only (field presence, `required` vs `properties`, `enum` contents, `additionalProperties`, length/format/boundary keywords, `maxItems`, pagination, dependency keywords). Judgment items (QD-T-3.1/3.4 human; QD-T-3.2/3.3/3.6/3.7 and QD-T-4.x semi-automated) require reading description semantics and tool relationships.
- **Output one verdict per rule** in the Stage-A contract form: `QD-T-x.y | PASS/FAIL | evidence: field path / JSON pointer / line`. Every FAIL must quote a short excerpt of the offending schema text.
- **Judgment threshold — a missing field/constraint is the defect**: a rule is FAIL when the schema field or constraint its definition names is **absent** from the Tool Schema. The "leading-to …" tail describes the **risk mechanism**, not an additional precondition to prove. Do not require a demonstrated concrete failure to mark a missing-field rule FAIL.
- **Four-state result (§5.1)**: each rule yields one of **适用 / 不适用 (N/A) / 无法判定 / 已豁免**. N/A items (e.g. QD-T-2.4 for a Tool with no array parameter) are removed from the scoring denominator; "无法判定" (PENDING or insufficient evidence) is counted at the highest possible level and flagged in the report; "已豁免" (Appendix D) is removed from the denominator and flagged.
- **P0 findings are release-blocking but do not halt the flow** (non-blocking): record every P0 FAIL and continue; later stages are not skipped because of P0 findings.
- **Level is tier-specific** (§3.2.2): the same rule carries different levels at different tiers. Use the level shown in the tier's own column.

## Tool operation risk levels (OR-L, §3.1)

| Tier | Designation | Definition | Determination criteria | Typical tools |
| --- | --- | --- | --- | --- |
| **OR-L1** | Low Risk | Read-only query; no state change | Data-read only; no sensitive data; repeatable, no side effects; failure consequence = query failure only | `get_weather`, `get_current_time`, `search_products`, `list_orders` |
| **OR-L2** | Medium Risk | State-changing; recoverable | Creates or updates data; changes recoverable or overwritable; may involve some sensitive data; failure consequence = data quality degradation requiring human intervention | `create_user`, `update_profile`, `send_notification`, `add_to_cart` |
| **OR-L3** | High Risk | Irreversible / sensitive operations | Deletion, payment, permission change or similar; highly sensitive data; irreversible or hard to recover; failure consequence = data loss, financial loss, permission leaks | `delete_user`, `process_payment`, `grant_permission`, `transfer_data` |

## Rule definitions and per-tier levels (§1.2, §2.2, §3.3)

The level shown is the rule's defect level **at that OR-L tier**. `—` = not applicable at that tier; `*` = triggered by parameter feature, not fixed per tier.

### QD-T-1.x — Structural validity (7 rules)

| ID | Rule name | Definition (condensed) | OR-L1 | OR-L2 | OR-L3 | Method |
| --- | --- | --- | --- | --- | --- | --- |
| QD-T-1.1 | Base Field Missing | Tool Schema missing required base fields: `name`, `description`, `inputSchema` (`outputSchema`, `metadata` are optional) | P0 | P0 | P0 | Automated |
| QD-T-1.2 | Required References Undefined Field | `required` lists a field not defined in `properties` (auto). Separate human sub-item: field necessity inconsistent with the business contract (P1) | P0 | P0 | P0 | Automated / Human |
| QD-T-1.3 | Enum type conflict / duplicate values | Duplicate values in `enum` (P2); `enum` value conflicts with declared `type` — an unreachable value (P1). Mixed-type or null-containing `enum` is legal in generic JSON Schema; permissibility depends on the declared dialect (§0.1.6) | P1 / P2 | P1 / P2 | P1 / P2 | Automated |
| QD-T-1.4 | additionalProperties Policy Inappropriate | The `additionalProperties` **strategy** is judged, not keyword presence: closed-contract fields should declare `false`; tags/metadata open dictionaries may omit it. Nested objects checked layer by layer, one level lower than root | P2 | P1 | P1 | Automated |
| QD-T-1.5 | name Validity & Uniqueness | name non-empty, matches `^[a-zA-Z0-9_-]{1,64}$`, unique within the tool list; `verb_noun` structure is style only (P2) | P0 (style P2) | P0 (style P2) | P0 (style P2) | Automated |
| QD-T-1.6 | Constraint Self-Consistency & default | `minimum > maximum`, `minLength > maxLength`, `minItems > maxItems`, `default` violating type/enum/range, `required` coexisting with `default`, unsatisfiable combined constraints | P1 | P1 | P1 (high-risk param P0) | Automated |
| QD-T-1.7 | Type Missing / Over-Broad / Complexity | Missing `type`, multi-type array, unconstrained object, nesting > 4, params > 30, `oneOf/anyOf/allOf/$ref` abuse, nullable union | P2 | P1 | P1 | Automated |

### QD-T-2.x — Parameter constraints (7 rules)

| ID | Rule name | Definition (condensed) | OR-L1 | OR-L2 | OR-L3 | Method |
| --- | --- | --- | --- | --- | --- | --- |
| QD-T-2.1 | String Without maxLength | String parameter lacks `maxLength` (`minLength` not required). Fields with `enum`/`const`/fixed-length `pattern` are exempt | P2 | P1 | P1 (executable/external field P0) | Automated |
| QD-T-2.2 | Format Contract Missing | String needing a specific format lacks `pattern`. `format` is annotation-only by default — prefer `pattern` | P2 | P1 | P1 | Semi-automated |
| QD-T-2.3 | Number/Integer Without Boundary | Number/Integer parameter lacks `minimum`/`maximum` (or exclusive variants) | P2 | P1 | P1 (amount/quantity P0) | Automated |
| QD-T-2.4 | Array Without maxItems | Array parameter lacks `maxItems` (`minItems` not required; empty array is often legal) | P2 | P1 | P1 | Automated |
| QD-T-2.5 | Unit / Precision / Timezone / Currency Missing | Amount in yuan/cents, time in seconds/milliseconds + timezone, date format, ratio vs percentage; amount-type numbers lack `multipleOf` | P2 | P1 | P0 (key params) | Semi-automated |
| QD-T-2.6 | Query Tool Without Pagination / Result Cap | `list_*`/`search_*` tools lack `limit`/`page_size`, or `limit` lacks `maximum` | P1 | P1 | P1 | Automated |
| QD-T-2.7 | Parameter Dependency / Mutual Exclusion Unexpressed | Pair-required, mutual-exclusion, conditional-required, at-least-one relations not expressed via `dependentRequired`/`oneOf` | P1 | P1 | P1 | Semi-automated |

### QD-T-3.x — High-risk operations (7 rules)

| ID | Rule name | Definition (condensed) | OR-L1 | OR-L2 | OR-L3 | Method |
| --- | --- | --- | --- | --- | --- | --- |
| QD-T-3.1 | State Change & Material Side Effect Undeclared | Write/side-effect not declared when the name/description reads as read-only (`verify_order` that actually deducts), or the side effect (irreversible / external / financial) is entirely undeclared. A name clearly expressing write intent (`update_user`) needs no fixed warning wording. Recommended: structured annotations `readOnlyHint`/`destructiveHint`/`idempotentHint` | — | P1 | P0 | Human |
| QD-T-3.2 | Untrusted Input into Executable Context | Parameter feeding SQL/shell/code/template-expression/file-path/URL context. URL/path parameters without constraint (domain whitelist, protocol restriction, directory prefix) are an explicit sub-item. Detected by executable-semantic feature + human confirmation of parameterization/sandbox/whitelist | P1* | P0* | P0* | Semi-automated |
| QD-T-3.3 | Batch / Range Write Without Cap | Batch/range write has no limit, covering array params (`ids`), filter conditions (empty filter = all), wildcards (`*`) | — | P1 | P0 | Semi-automated |
| QD-T-3.4 | Authorization Context Parameter Exposure | Tool exposes parameters that should be system-controlled (`tenantId`, `admin`, `token`) | P0* (by feature) | P0* (by feature) | P0 | Human |
| QD-T-3.5 | Cross-Tool Data Flow | One tool's output fed as another's input causing unintended flow | transferred → cross QD-TT-5.1 (P2 lead retained) | transferred | transferred | Semi-automated |
| QD-T-3.6 | Dangerous Default / Scope Precondition | `force: default true`, `dry_run: default false`, `scope: default ALL`, empty filter = all, wildcard widening scope | — | P1 | P0 | Automated + Human |
| QD-T-3.7 | Parameterized-Tool Operation Whitelist Undeclared | Open-ended tools (`sql_query`/`exec`/`http_request`/`eval`) whose operation object is set by argument, with no declared whitelist (table/command/domain). Permission set is non-enumerable ("unclosed set") — a blocking form, per inspect/permission.md §2.2.5 | P0 | P0 | P0 | Semi-automated |

### QD-T-4.x — Semantic clarity (7 rules)

| ID | Rule name | Definition (condensed) | OR-L1 | OR-L2 | OR-L3 | Method |
| --- | --- | --- | --- | --- | --- | --- |
| QD-T-4.1 | Insufficient Description Quality | Description missing required elements (action, object, scope/timeliness, key parameter semantics, return summary); FAIL only when ≥2 are missing. Instruction injection in description is Risk Explicit NR-S-5, not this item | P2 | P1 | P1 | Semi-automated |
| QD-T-4.2 | Side Effect Undeclared | **Merged into QD-T-3.1**; number retained, no longer used independently | — | — | — | — |
| QD-T-4.3 | Multi-Tool Semantic Competition | Multiple tools with semantically similar name/description | transferred → cross QD-TT-5.2 (P2 lead retained) | transferred | transferred | Semi-automated |
| QD-T-4.4 | Parameter-Level description Missing | A property has no `description` | P2 | P1 | P1 | Automated |
| QD-T-4.5 | description Inconsistent with Schema | Within one tool: description says "max 10" but schema allows 100; description names a nonexistent parameter; example value not in `enum`; `get_*` actually writes | P1 | P1 | P1 | Semi-automated |
| QD-T-4.6 | Sensitive Input Handling Undeclared | Password/key/ID-number as legal input should declare `writeOnly`/secret handling (not logged, not echoed) | — | P1 | P1 | Semi-automated |
| QD-T-4.7 | Output Contract Missing (outputSchema) | Tool lacks `outputSchema`; upgraded to P0 when a cross-tool data flow (cross QD-TT-5.1) exists | — | P1 | P1 (cross-flow P0) | Semi-automated |

### QD-T-5.x — Schema change & compatibility (Part 4; change-management rules)

| ID | Rule name | Definition (condensed) | Level | Method |
| --- | --- | --- | --- | --- |
| QD-T-5.1 | Breaking Change Determination | Identify changes that break existing calls; level per §4.3.1 matrix, not a fixed P level. Uncontrolled breaking change = P0 ("controlled" = version isolation/parallel version or migration guarantee, declared in the change note) | §4.3.1 matrix | Automated |
| QD-T-5.2 | Change Impact Assessment Missing/Wrong | Change note fails to declare caller/data/system impact, or declared impact contradicts facts | OR-L1 P1 / OR-L2 P1 / OR-L3 P0 | Semi-automated |
| QD-T-5.3 | Semantic Change | Structure unchanged but description/tool name/default/unit/query-scope/side-effect changed; deprecated fields need a `deprecated` marker with transition | OR-L1 P1 / OR-L2 P1 / OR-L3 P0 | Automated + Human |

> **QD-T-5.x scope note** (reproduced, not adjudicated): the 5.x rules belong to Part 4 (Schema change & compatibility) and are triggered only when there is an old/new Schema comparison object. They are **outside the static check denominator** (20/25/25, Appendix B note) — do not emit them in a single-artifact Stage-A verdict.

## Minimum rule sets (Appendix A)

The sets are **cumulative** (a higher tier covers all lower-tier items). "Must-pass" = the tier's P0 items; the rest are recommended-pass.

### OR-L1 minimum set — 20 items (P0: 5, P1: 6, P2: 9)

- **Must-pass (P0, 5)**: QD-T-1.1 · 1.2 · 1.5 · 3.4 · 3.7
- **P1 (6)**: QD-T-1.3 · 1.6 · 2.6 · 2.7 · 3.2 · 4.5
- **P2 (9)**: QD-T-1.4 · 1.7 · 2.1 · 2.2 · 2.3 · 2.4 · 2.5 · 4.1 · 4.4

### OR-L2 minimum set — 25 items (P0: 6, P1: 19, P2: 0)

- **Must-pass (P0, 6)**: QD-T-1.1 · 1.2 · 1.5 · 3.2 · 3.4 · 3.7
- **P1 (19)**: QD-T-1.3 · 1.4 · 1.6 · 1.7 · 2.1 · 2.2 · 2.3 · 2.4 · 2.5 · 2.6 · 2.7 · 3.1 · 3.3 · 3.6 · 4.1 · 4.4 · 4.5 · 4.6 · 4.7

### OR-L3 minimum set — 25 items (P0: 10, P1: 15, P2: 0)

- **Must-pass (P0, 10)**: QD-T-1.1 · 1.2 · 1.5 · 2.5 · 3.1 · 3.2 · 3.3 · 3.4 · 3.6 · 3.7
- **P1 (15)**: QD-T-1.3 · 1.4 · 1.6 · 1.7 · 2.1 · 2.2 · 2.3 · 2.4 · 2.6 · 2.7 · 4.1 · 4.4 · 4.5 · 4.6 · 4.7

## Appendix B index — QD-T complete numbering (inspect/tool.md)

| ID | Rule name | Category | Level (OR-L1/2/3) | Method |
| --- | --- | --- | --- | --- |
| QD-T-1.1 | Base Field Missing | Structural Validity | P0 / P0 / P0 | Automated |
| QD-T-1.2 | Required References Undefined Field | Structural Validity | P0 / P0 / P0 | Automated / Human |
| QD-T-1.3 | Enum type conflict / duplicate values | Structural Validity | P1 / P2 (all tiers) | Automated |
| QD-T-1.4 | additionalProperties Policy Inappropriate | Structural Validity | P2 / P1 / P1 | Automated |
| QD-T-1.5 | name Validity & Uniqueness | Structural Validity | P0 (style P2) | Automated |
| QD-T-1.6 | Constraint Self-Consistency & default | Structural Validity | P1 / P1 / P1 (high-risk param P0) | Automated |
| QD-T-1.7 | Type Missing / Complexity | Structural Validity | P2 / P1 / P1 | Automated |
| QD-T-2.1 | String Without maxLength | Parameter Constraints | P2 / P1 / P1 (executable/external P0) | Automated |
| QD-T-2.2 | Format Contract Missing | Parameter Constraints | P2 / P1 / P1 | Semi-automated |
| QD-T-2.3 | Number/Integer Without Boundary | Parameter Constraints | P2 / P1 / P1 (amount/quantity P0) | Automated |
| QD-T-2.4 | Array Without maxItems | Parameter Constraints | P2 / P1 / P1 | Automated |
| QD-T-2.5 | Unit / Precision / Timezone / Currency | Parameter Constraints | P2 / P1 / P0 (key params) | Semi-automated |
| QD-T-2.6 | Query Without Pagination / Cap | Parameter Constraints | P1 / P1 / P1 | Automated |
| QD-T-2.7 | Dependency / Mutual Exclusion Unexpressed | Parameter Constraints | P1 / P1 / P1 | Semi-automated |
| QD-T-3.1 | State Change & Side Effect Undeclared | High-Risk Operations | — / P1 / P0 | Human |
| QD-T-3.2 | Untrusted Input into Executable Context | High-Risk Operations | P1* / P0* / P0* | Semi-automated |
| QD-T-3.3 | Batch / Range Write Without Cap | High-Risk Operations | — / P1 / P0 | Semi-automated |
| QD-T-3.4 | Authorization Context Parameter Exposure | High-Risk Operations | by feature / by feature / P0 | Human |
| QD-T-3.5 | Cross-Tool Data Flow | High-Risk Operations | transferred → cross QD-TT-5.1 | Semi-automated |
| QD-T-3.6 | Dangerous Default / Scope Precondition | High-Risk Operations | — / P1 / P0 | Automated + Human |
| QD-T-3.7 | Parameterized-Tool Operation Whitelist Undeclared | High-Risk Operations | P0 / P0 / P0 | Semi-automated |
| QD-T-4.1 | Insufficient Description Quality | Semantic Clarity | P2 / P1 / P1 | Semi-automated |
| QD-T-4.2 | Side Effect Undeclared (merged → QD-T-3.1) | Semantic Clarity | merged | — |
| QD-T-4.3 | Multi-Tool Semantic Competition | Semantic Clarity | transferred → cross QD-TT-5.2 | Semi-automated |
| QD-T-4.4 | Parameter-Level description Missing | Semantic Clarity | P2 / P1 / P1 | Automated |
| QD-T-4.5 | description Inconsistent with Schema | Semantic Clarity | P1 / P1 / P1 | Semi-automated |
| QD-T-4.6 | Sensitive Input Handling Undeclared | Semantic Clarity | — / P1 / P1 | Semi-automated |
| QD-T-4.7 | Output Contract Missing (outputSchema) | Semantic Clarity | — / P1 / P1 (cross-flow P0) | Semi-automated |
| QD-T-5.1 | Breaking Change Determination | Schema Changes | §4.3.1 matrix | Automated |
| QD-T-5.2 | Change Impact Assessment Missing/Wrong | Schema Changes | P1 / P1 / P0 | Semi-automated |
| QD-T-5.3 | Semantic Change | Schema Changes | P1 / P1 / P0 | Automated + Human |

> **Index scope note** (reproduced, not adjudicated): QD-T-5.x are change-management rules triggered only on version change; they are outside the §5.2.1 static check denominator (20/25/25). QD-T-4.2 is a tombstone (merged into 3.1); QD-T-3.5 / QD-T-4.3 are tombstones (transferred to Inspect Cross QD-TT-5.1 / QD-TT-5.2, with a P2 lead retained here).

## Scoring note (§5, optional)

Weights **P0:P1:P2 = 5:3:1**; Total Weight = (tier P0 count × 5) + (P1 × 3) + (P2 × 1); Base Score = 100 / Total Weight. Item counts come from §3.3 (not Appendix B — the 5.x rules are excluded).

| OR-L tier | Counted items | P0 | P1 | P2 | Total weight | Base score |
| --- | --- | --- | --- | --- | --- | --- |
| **OR-L1** | 20 | 5 | 6 | 9 | 5×5 + 6×3 + 9×1 = 52 | 100/52 ≈ 1.9231 |
| **OR-L2** | 25 | 6 | 19 | 0 | 6×5 + 19×3 = 87 | 100/87 ≈ 1.1494 |
| **OR-L3** | 25 | 10 | 15 | 0 | 10×5 + 15×3 = 95 | 100/95 ≈ 1.0526 |

Per failed item deduct Base × 5 / 3 / 1 (highest severity, once per item number regardless of instance count); Score = max(100 − deductions, 0); **any P0 → FAIL** (score still output). Four-state results remove N/A and waived items from the denominator; "无法判定" is counted at the highest possible level and flagged. Compute an indicative score only if the user asks, and label it "indicative (lite minimal set), not the full-spec score".

> **Source notes** (reproduced, not corrected): the source marks §5.2 scoring as PENDING (E-2) pending cross-spec alignment of scoring differentiation, and QD-T-5.1–5.3 as not counted in the static denominator (see review P0-05). The lite inspector reproduces the source's published values verbatim — it defines no new rules and does not silently "fix" source numbers; on conflict `inspect/tool.md` governs.
