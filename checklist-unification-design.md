# Unified Checklist Overhaul — Design Decision Record

> Status: **Finalized (v2 fully rolled out)**. This document is the single source of design truth for the unified checklist overhaul; it is for reference when overhauling the three consumers (skill / CLI / platform), and is also used to carry context across conversations/windows.
>
> Companion artifacts: `_meta.md` + the 8 domain checklists have all been produced in full per this record (commit `a21a67a`), paths in §6.

## 1. Background and goals

**Current problems**: the skill's 8 `references/*.md` files each condense a source spec, with inconsistent fields/conventions, and "a source model change requires updating the checklist, SKILL.md, and README in multiple documents", with high drift risk (historical lessons are recorded in the project memory).

**Goal**: maintain **one unified checklist as the single source of truth**, shared by the three consumers (skill / CLI / platform) for normalization, and derive what each needs from it. Eradicate drift.

**Scope**: this round changes only skill and CLI; platform reuses the same checklist but its implementation is already fixed and is out of scope for this overhaul (field design still takes platform's full functionality as the ceiling).

## 2. Settled decisions (consensus list)

### 2.1 Single source of truth and file structure

- The checklist is **carried in one md file, with no generation, no synchronization, no copies**.
- Physical structure has two layers:

```
_meta.md                       — shared meta document: field definitions, judgment conventions, N/A discipline, severity rules, fail-safe (small, loaded once)
prompt-checklist.v2.md         ┐
skill-checklist.v2.md          │
tool-checklist.v2.md           ├ domain check items (pure check items, loaded by stage)
code-contract-checklist.v2.md  │
cross-checklist.v2.md          │
permission-checklist.v2.md     │
explicit-checklist.v2.md       │
relevance-map.v2.md            ┘
```

- **No `id-index.md` produced**: Stage G's back-closure reads each domain's §2 index directly (reasons in §5 "Rejected").
- Iron rule: **md is the source of truth; any change is made only here**.

### 2.2 Field model (all fields, shared by the three consumers)

Principle: **fields take the most complete consumer (platform) as the ceiling; "whether to use" trimming happens on the consumer side, not on the field side.**

| Field | Meaning | skill | CLI | platform |
|---|---|---|---|---|
| `id` / `name` / `definition` | Traceability anchor + judgment definition | ✓ | ✓ | ✓ |
| `check_engine` | `program` / `model` / `hybrid` | ignore | ✓ routing | ✓ routing |
| `applicability` | Structural facts (decide "whether a check runs") | ✓ | ✓ | ✓ |
| `severity` | Full mapping by L (decides "how severe") | ✓ | ✓ | ✓ |
| `program_rule` | Program judgment rules | ignore | ✓ | ✓ |
| `model_prompt` | Model judgment points | ✓ | ✓ | ✓ |
| `score_weight` | Scoring weight / whether counted in the denominator | ignore | ignore | ✓ |
| `examples` | Positive / counter-examples | ✓ | ✓ | ✓ |
| `cross_refs` | Same-source dedup / merge / transfer relationships | ✓ | ✓ | ✓ |

### 2.3 ID system: immutable

- Check item IDs reuse `QD-*` / `QC-*` / `EX-*` / `AS-*` / `FM-*` and `P0/P1/P2` labels, **without changing a single character**.
- Merged/transferred items are recorded with status markers (`merged` / `transferred`); numbering is retained, never renumbered.

### 2.4 P and L: decoupled, L only feeds P

- In the original framework P is bound to L (tier-specific severity), and needs decoupling.
- New design: **L only decides the value of P, no longer decides "whether to check"**.
- L determination is **moved forward to sanityops-extract-only**, determined once.

### 2.5 L merging: separated at the engine layer, merged at the product layer

- **Engine layer**: AC-L (complexity) and OR-L (operation risk) **keep two dimensions, not merged into one** (merging would compute P incorrectly, e.g. AC-L1 + OR-L3 cannot be expressed).
  - AC-L: single-turn / multi-turn / complex / multi-agent, decides "which items to check" (original duty, now mostly taken over by the structural-fact gate) + Prompt-domain P.
  - OR-L: read-only / reversible write / irreversible, decides Skill/Tool-domain P.
- **Product layer**: merged into **one unified label** shown to the user (Prompt→AC-L; Skill/Tool→OR-L; Permission→max).
- One sentence: merging happens in "how to show it to the user", not in "how the engine computes".

### 2.6 Structural-fact gate (no item dropped, core)

- "Whether a check runs" is decided by **observable structural facts** (`has_tool` / `has_workflow` / `has_write_semantics`, etc.), **not by L**.
- This guarantees: **if L is wrong, at worst P is off by one grade; an item is never dropped**.
- Fail-safe defaults: `on_absent → N/A` (basis must be written), `on_unknown → judge` (default to check).
- N/A is the exit for "structurally not applicable", **not "missing"**.
- **Key precondition (point-1 paradox, must follow)**: structural facts judge "what the artifact claims to do / what signs appear", and **cannot** be "whether a declaration is complete". Otherwise it cancels its own item — e.g. if the applicability of `QD-P-2.4.2 Tool Not Declared` is judged by "is a tool declared", it can never catch the "not declared" defect.

### 2.7 IR two layers (pre-extraction output)

sanityops-extract-only outputs a **two-layer** IR:

- **Object layer**: objects actually extracted with a complete signature (how many P / S / T).
- **Reference layer**: references mentioned / depended on in the artifact but not yet formed.

Cross-object checks (e.g. "tool existence") look at the **reference layer**, to avoid missing "should exist but does not" defects.

- **Materialize to disk**: the IR is the output of sanityops-extract-only and is **materialized as a Markdown file** (default `{input_basename}.ir.md`) for audit traceability (distinguish "extraction error" from "inspection error") and reuse by CLI/platform; a copy is also kept in-context for chained handoff.
- **Machine-readable core**: the IR embeds a `<!-- ir-core -->` JSON block carrying only the machine-readable fields — `objects[].{type,name,completeness,L,source,mismatch}`, `not_inspectable[].{name,reason}`, and `references[].{name,referenced_by,relationship}`; the program engine parses only this block (the same pattern as the checklist's `program-check`).

### 2.8 check_engine (three values)

- `program`: deterministic rules, executable by the program engine (e.g. missing fields, syntax parsing).
- `model`: semantic judgment, requires model reasoning (e.g. contradictory instructions, description quality).
- `hybrid`: program pre-screening + model confirmation. **Prioritize convergence during development** — if pure model is possible, use pure model, or put screening rules in the program block and confirmation points in the main text.

Consumption by the three consumers:
- skill: ignores check_engine, single full-model reasoning (program items are also "simulated-judged" by the model).
- CLI / platform: program goes to the program engine, model to the model engine, hybrid is two-stage.

### 2.9 Scoring

- skill and CLI **do not score and do not map to AI-application levels**; they output only "check item → P → evidence".
- But the `score_weight` field is **retained** (platform-only), not deleted.
- Global weight: P0 : P1 : P2 = 5 : 3 : 1.

### 2.10 Reference-level display (avoid false alarm)

- Check items omitted by the original spec due to L are **downgraded to reference level** for display (a display-layer concept, not entering the data layer).
- **The original P must be traceable**: annotate "this item is P0 under L3", to avoid "downgrade = hidden missed detection".

### 2.11 Self-check (anti-hallucination, layered)

- **Program self-check** (CLI only): guards only against "formal errors" (whether the ID exists, whether P is legal, evidence non-empty, format). **Cannot prevent "misattribution"** (semantic mismatch).
- **Model triangle check** (skill + CLI model engine):
  - Forward: `artifact + checklist → defect ID`; reverse: `defect ID + defect content → does the checklist definition match`.
  - **Must be a separate task / independent context** — mixing it into the detection context degrades self-check into rubber-stamping.
  - Only check the small number of already-detected IDs item by item.
- Conclusion: **program handles form, model handles semantics; complementary, not substitutes**. The skill has no program, so it can only do a soft self-check (triangle check + ID back-closure).

### 2.12 Missed detection

- No systematic solution (cannot do a full rescan).
- Two paths: ① accumulate counter-examples in practice; ② **turn more items into program items** (program items enumerate in full by nature and do not miss).

### 2.13 Loading strategy (token efficiency)

- **The meta document is loaded once**, reused across stages.
- **Incremental loading by stage and by artifact type**: load only the domain currently needed, do not preload unrelated domains.
- **Domain check items stay complete; no "applicability pre-screening deletion of items"** (otherwise it misses "should have declared but did not declare" defects); rely on the structural-fact gate to produce N/A at check time (judge quickly, do not dig deep).
- **CLI removes program items** (goes to the program engine); the skill does not remove them (full model).
- The IR reference layer is loaded as **input**, not into the checklist.

### 2.14 Per-domain conversion mechanical checklist (follow item by item when rolling out in full)

1. Extract ID/name/severity verbatim from the source; use the "rule definition table" as the authoritative source for name.
2. check_engine: `Automated→program`, `Human→model`, `Semi-automated→hybrid`; when the source Method contradicts the definition, correct the source per "prioritize convergence" and leave a log.
3. applicability.fact uses observable structural facts; "missing X field / not declared" checks use `is_*` (type facts, always true) rather than `has_*` (point-1-paradox prevention).
4. severity notation: `P0/P1/P2`, `∅`(N/A), `P1→P0(conditional)`, `P1·P2`(dual sub-defect), `P0(style P2)`; source `*`(by-feature) merged into applicability.fact.
5. Wrap program items in JSON with `<!-- program-check -->`, containing only the program + hybrid pre-screening summary; model items never enter.
6. Three-column closure: §2 index / §3.1 model / §3.2 hybrid confirmation / §4 program block, ID sets identical.
7. tombstone (merged/transferred) in a separate section, numbering retained.
8. Scoring: `score_weight` retained; change-check and tombstone not counted in the denominator.
9. examples suspended, marked as placeholder.
10. Domain-specific L: Prompt uses AC-L, Skill/Tool uses OR-L, Permission uses max; the severity table header explicitly marks the prefix.

## 3. Key correction history (lessons, prevent repetition)

1. **Format selection: YAML → md**. Once leaned toward YAML single-source + program-generated md, later corrected as an over-design to "accommodate a few program items". Correct: md is the source of truth; program items are carried separately in an embedded `<!-- program-check -->` JSON block, and the program engine parses only these blocks.
2. **Scoring field: do not delete**. Once the "skill/CLI do not score" rule was wrongly implemented as "delete the score_weight field", violating "fields take platform as the ceiling". Correct: fields retained, behavior trimmed on the consumer side.
3. **P and L**: in the original framework P is tier-specific (bound to L), not an independent fixed value; after decoupling, scenario sensitivity is preserved by "structural-fact gate + L only feeds P".
4. **Point-1 paradox**: applicability judges "what the artifact claims to do", not "whether a declaration is complete".
5. **`*` (by-feature) semantic attribution**: in the source, 3.2 carries `*` on all three tiers, while 3.4 marks `P0*(by feature)` in OR-L1/L2 and `P0` (no star) in OR-L3, self-contradictory. Decision: `*` semantics is uniformly merged into `applicability.fact` (3.2→has_executable_param, 3.4→has_authz_param), and severity stores only "the value after a hit" (3.4 takes P0/P0/P0). Basis: 3.4 is essentially feature-triggered (a defect exists only with an authz parameter); the missing star at L3 is a source shorthand.
6. **name authoritative source**: the source "rule definition table" and "Appendix B index table" have inconsistent names (1.7/2.5/2.6/2.7/3.1). Decision: use the **rule definition table** as the authoritative source for name (more complete information, consistent with name = traceability anchor).
7. **Method→check_engine source correction**: 1.4 (Automated→hybrid), 4.1/4.6/5.2 (Semi-automated→model) — four places deviate from the source Method column, because the source Method column contradicts its own definition. Accept the source correction and leave a reclassification log in the sample.

## 4. Pending / suspended items

- **Counter-examples (examples)**: suspended, to be discussed later, not handled for now.
- **Whether the unified L ruler should add L4** (multi-agent scenario): undetermined; decide L1–L3 or L1–L4 by whether multiple agents are involved.
- **CLI/platform full-body carry**: the program engine parses only `ir-core`, which carries no artifact body — CLI/platform cannot get the artifact text from `ir-core` alone. Undecided: add a `body` field to `ir-core`, or have CLI/platform read the human-readable `Full verbatim body` from the `.ir.md` object layer.

## 5. Rollout progress and next steps

**Completed**:
- [x] tool domain sample `tool-checklist.v2.md` (pilot sample; the field model was confirmed and rolled out to all domains)
- [x] Roll out the remaining 7 domains + `_meta.md` (prompt / skill / code-contract / cross / permission / explicit / relevance)
- [x] Modify sanityops-extract-only: add upfront L determination + two-layer IR output
- [x] Modify the main SKILL.md: the flow references the new checklist approach, Stage G self-check upgraded to triangle check, scoring removed, reference-level display added
- [x] Synchronize README / GET_STARTED / `quality/code-contract.md` / `package.json`

**Rejected**:
- `id-index.md`: **not produced**. Stage G's ID back-closure / forward coverage read each domain's §2 index directly; creating another index is adding another copy that must be synchronized, violating the "no generation, no synchronization, no copies" principle.

**Remaining (non-blocking)**:
1. `examples` (positive/counter-examples) all suspended, to be added by domain later.
2. The CLI / platform side (another repo `sanityops-cli`) is implemented separately per this decision record + development guidance, not in this repository.

## 6. Related files

- Domain files (`_meta.md` + 8 `*.v2.md`): `plugins/sanityops-inspector/skills/sanityops-inspect-only/references/`
- pre layer: `plugins/sanityops-inspector/skills/sanityops-extract-only/SKILL.md`
- main skill: `plugins/sanityops-inspector/skills/sanityops-inspect-only/SKILL.md`
