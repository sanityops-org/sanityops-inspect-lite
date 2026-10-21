---
name: sanityops-inspect-lite
version: 1.0.0
description: Lite, zero-install inspector for AI Agent logic artifacts (System Prompt, Skill, Tool Schema) based on the SanityOps Framework v1.0. Runs a seven-stage non-blocking flow — single-artifact defect inspection (QD-P/QD-S/QD-T/QC), cross-artifact contract checks (QD-PS/PT/ST/CYC), Gate-0 admission, permission proportionality quick check (QD-PM), explicit-risk audit (EX: NL-A/NL-B/NR-O/NR-S), a candidate-impact outlook (AS-*/FM-* mapping), and a lightweight self-verification pass (Stage G). P0 findings are aggregated and reported but never halt the flow. Detect-only: it never rewrites the inspected artifacts. Use when the user asks to inspect, audit, or check an agent's system prompt, skill definition, or tool schemas.
license: CC BY-SA 4.0
---

# SanityOps Inspect Lite

## 1. What this skill is

A **detect-only** inspector for the three Logic Artifact types in SanityOps Framework v1.0 — **System Prompt**, **Skill**, and **Tool Schema** (Permission as a cross-cutting aspect; embedded code snippets inside them get the lite QC check) — executed by you, the host model, applying the condensed checklists in `references/`. Zero dependencies: no CLI, no scripts, no network calls.

It is **not**:

- The official `sanityops-cli` or any SaaS — condensed minimal rule sets only, not the full specification.
- A security or quality gate — Inspect PASS ≠ Risk PASS ≠ Quality PASS (core.md §1.4); the Risk Implicit and runtime Quality subsets are **not** run (the lite embedded-code QC in Stage A is the only Quality check).
- A repair tool — it never rewrites, patches, or generates fixed versions.
- A runtime auditor — Permission quick mode validates only what is *declared in the artifacts*, not IAM/IdP configuration, PEP/PDP enforcement, or runtime monitoring (permission.md §0.1.4).
- An executing permission — it declares no write, delete, or environment-elevation scope; the skill is read-only static inspection of authorized artifacts and performs no production operation of any kind.

## 2. Inputs

Only the **Artifacts** are mandatory from the user's side. Tier declarations and the version label are optional: when the user cannot provide them, determine tiers from the checklist rules and record the version as "undeclared" — never block the run on their absence.

Ask the user for (do not start until you have them, or record explicitly what is missing):

1. **Artifacts** — any of: System Prompt (text/markdown), Skill definition (e.g. SKILL.md and its referenced files), Tool Schema(s) (JSON Schema / function-calling definitions).
2. **Tier declarations** — the Prompt's Agent Complexity Level (AC-L1/L2/L3), the Skill's risk level (OR-L1/L2/L3), and each Tool's risk level (OR-L1/L2/L3) (per the tier rules in the checklists). If the user cannot declare a tier, apply the tiering rules from the checklists and state the determination.
3. **Version label** for the artifact set (required for the multi-round ledger).

Permission quick mode runs on whichever artifact types are present (Prompt, Skill, and/or Tool Schema), not on a required three-artifact set. Record Permission as NOT EXECUTED with reason "no permission-evidence artifact" only when none of the three types is present at all.

**Input budget (configurable default).** One Agent per run; up to 50 artifacts (1 Prompt + 1 Skill + up to 48 Tool Schemas); per-type size caps — a Tool Schema up to 2,000 tokens, a System Prompt up to 30,000 tokens, a Skill definition (with its referenced files) up to 100,000 tokens; a run total up to 200,000 tokens on a 1M-token host window (scale down proportionally on smaller windows). If exceeded, split the run across rounds, or record the over-limit artifact as `summary` fidelity (Step 0 below).

### Input preparation (not a Stage)

**Object segmentation (before Step 0).** When one input file packages several artifacts — e.g. a test-case document that embeds multiple verbatim artifacts (each marked `### N. <name> (verbatim in full)`) plus container scaffolding — split it first:

- **Artifact segment** — each marked verbatim block is one candidate object.
- **Container scaffolding** — metadata blocks, `Project Structure`, `Logical Artifact Inventory` headers, `Evaluation Dimensions`, `Scoring Criteria`, `Test Suggestions`, `Notes`, and other non-artifact sections — is not an artifact and is not inspected.
- Apply Step 0 and type classification **per artifact segment**, never to the container as a whole. Container scaffolding does not trigger S1/S2/S3 against any segment.
- The term "artifact" in Stages A–E refers to these segments.
- If the file has no explicit segment boundary, treat the whole file as one candidate object (existing behavior).

**Step 0 — Fidelity classification (run first, per object).** For each candidate object, determine its fidelity tier before type classification:

- **`full`** — provably the original artifact: the user declares "this is the original file", or the object is a verbatim artifact with no summary/meta/framework signals.
- **`summary`** — any of: **S1** self-declared summary/excerpt/transcribed (meta-level wording such as `Collection Method: summary`, `structure analysis`, `This is a summary`, or the equivalent meta-level wording in the artifact's own language); **S2** the object contains non-artifact sections (e.g. `Evaluation Dimensions`, `Scoring Criteria`, `Test Suggestions`, `Collection Date`/`Method`, `Relationship`); or **S3** the object has no complete structural signature.
- **Default fallback**: when `full` cannot be confirmed, treat as `summary`.
- **Hard rules**: do not keyword-scan artifact *content* for "summary" (only meta-level self-description counts); a user declaration of "original file" overrides S1/S2/S3; apply per-object, never as a single whole-batch verdict.

**Fidelity consequence (global — applies to every stage A–G, not just Stage A).**
- `full` → run all rules in every stage.
- `summary` → in every stage, run only **contradiction-based** findings (FAIL reason = "A conflicts with / is inconsistent with / forms an unterminated loop"); **skip absence-based** findings (FAIL reason = "missing / not declared / undefined / no X"). This skips e.g. QD-T-1.1/1.2/1.4/2.1/4.1, QD-P-2.2.2/2.2.3/2.7.x, QC-0.3/3.2 and like absence rules.
- **Stage E (EX) is bound by the same gate**: under `summary`, the risk audit must not read example code, safety measures, or missing declarations as "explicit risk"; only a risk expression that is verbatim, self-contained, and itself contradicts a declared boundary qualifies. Inference-based findings (cross-object chains, "read-only is only natural language", "scope undefined", etc.) are skipped under `summary`.

**Type classification.** After object segmentation, classify each artifact segment (or each file, when no segmentation applies) into one of the three types by structural signature — record the type and the basis:

- **Skill** → markdown with `name`/`description` front matter (e.g. `SKILL.md`).
- **Tool Schema** → JSON Schema / function-calling definition (`parameters` + `description`).
- **System Prompt** → otherwise, prose/markdown instructions.

**Why a structural header is required (not optional).** A Skill or Tool without its header cannot be loaded by the runtime — it cannot run, so there is no production-grade artifact to inspect. A `## Skills` list (e.g. `- browser: search docs`) only *describes* a capability; a bare function call (e.g. `await fetchTickets({...})`) only *invokes* a tool. Both are documentation, not artifacts. Inspecting them for a "missing contract" is a category error: it reports phantom defects against text that was never meant to be run, and the real production artifact (which does carry a header) will differ from this description anyway. Such prose is therefore skipped, not inspected.

Edge cases: an archive → expand first, then group files by agent; a script → treat as a referenced file, not an artifact, **except** when it embeds an in-scope logic-artifact literal (e.g. `system_message = """..."""`), which is classified as the corresponding artifact type and tiered by Step 0. If a file cannot be unambiguously classified, or the inputs appear to mix multiple agents, ask the user before starting Stage A. Never run Stage B/C/D/E across different agents' artifacts.

## 3. Governing sources and hard precedence

- The eight files in `references/` are **condensed extractions** of the SanityOps sub-specifications. They define **no new rules**.
- On any conflict (numbering, definitions, levels, thresholds, wording), the source sub-specification governs:
  - QD-P ← `inspect/prompt.md` · QD-S ← `inspect/skill.md` · QD-T ← `inspect/tool.md` · QD-PS/PT/ST/SS/TT/CYC ← `inspect/cross.md` · QD-PM ← `inspect/permission.md` · QC ← `quality/code-contract.md` (DRAFT — the lite `code-contract-checklist.md` is the operational source) · EX ← `risk/explicit.md` · impact mapping ← `framework/relevance.md`
- Only output rule IDs that exist in the references. **Never invent QD, EX, or QC IDs, rule names, or levels.**
- This package is licensed CC BY-SA 4.0, attributed to the SanityOps Framework.

## 4. Inspection flow — seven stages, strictly in order, non-blocking

Run stages **in order, never in parallel, never out of order**. After every stage, emit that stage's report before moving on. P0 findings **never halt the flow**: they are aggregated, reported in the executive summary and status board, and carried forward to later stages.

Plain-language note for the end user: **P0 = "must fix before release" (a severity level)**; **non-blocking = the inspection does not stop mid-run** — every stage still runs and all P0s are summarized at the end. A P0 is still a real defect to fix; "non-blocking" only means the scan completes instead of halting at the first problem.

A stage is skipped (Not Executed) only for concrete structural reasons — missing artifacts, or the unsatisfied Gate-0 structural condition (G0-4) — never because of P0 findings.

### Stage A — Single-artifact inspection (QD-P / QD-S / QD-T / QC)

1. For the System Prompt → `references/prompt-checklist.md`.
2. For the Skill → `references/skill-checklist.md`. **Run the QD-S-0 baseline checks first** (they gate the rest of the Skill checklist).
3. For each Tool Schema → `references/tool-checklist.md` (tier the Tool first, per the checklist's tier rules).
4. For code snippets embedded in any artifact (Prompt text, Skill steps, Tool Schema description/default/example fields) → `references/code-contract-checklist.md`. **Run the QC-0 baseline first** (it gates QC-1 through QC-4). QC inspects embedded snippets only — never standalone source files.
5. Verdict per rule: `QD-x-y | PASS/FAIL | evidence: file, section/line`. Every FAIL must quote the offending artifact text (short excerpt).
6. **Single-artifact boundary (hard)**: inspect each artifact segment in isolation. A declaration missing from the segment under inspection is a defect of *that* segment — do not treat a declaration present in a different segment as satisfying it. Evidence for every FAIL must quote the inspected segment itself, never a sibling segment.
7. Collect all P0 findings and carry them forward. P0 findings do **not** halt the flow — continue to Stage B in the same run, unless the user asked for Stage A only.

### Stage B — Cross-artifact inspection (QD-PS / QD-PT / QD-ST / QD-SS / QD-TT / QD-CYC)

1. Use `references/cross-checklist.md`: evaluate the tier's inspection scope per cross.md §5.4.3 / §8.3.1 (AC-L1 = 7 items, AC-L2 = 20, AC-L3 = 29, with the explicit per-tier item composition given in the checklist), applying Appendix B's tier P0 set (7/9/9) as the blocking compliance core. The checklist documents the source's stale version label (v1.0 header vs expanded 29-item content); do not resolve it by inventing items.
2. Inspect only cross-artifact relationships — the six relationships defined by the spec (Prompt↔Skill, Prompt↔Tool, Skill↔Tool, Skill↔Skill, Tool↔Tool, and the cross-cutting QD-CYC circular-invocation check at L2/L3). Do **not** re-check single-artifact internal defects.
3. Any P0 → record it and continue to Stage C; P0 findings do **not** halt the flow.

### Stage C — Gate-0 (admission to Permission)

Gate-0 **is not an inspection item**: it produces no numbers and no score; "not passed" ≠ FAIL ≠ permission problems (permission.md §9.2.2). Verify the four admission conditions G0-1..G0-4 defined in `references/permission-checklist.md` §3.2. In this non-blocking flow, G0-1 (no unfixed P0 in Stage A) and G0-2 (no unfixed P0 in Stage B) are **recorded as informational findings and do not gate**; G0-3 (responsibility boundary statements exist and are locatable) is **non-blocking** — its absence only downgrades the 1.x items to N/A; G0-4 (each Tool Schema's name/description are complete, tool-level only) remains the only structural admission condition for Stage D.

- **Report every G0 condition with its result and reason, whether it passes or fails** — never a bare `Passed`. G0-4 is reported as `complete` / `vacuous (no Tool)` / `incomplete`; G0-3 as `locatable` / `absent → 1.x N/A`. A reader must be able to tell *how* each condition passed, not just that the gate opened.
- G0-4 passes → proceed to Stage D (G0-1/G0-2 findings and any G0-3 absence, if any, are carried forward in the report; a G0-3 absence N/A's the 1.x items).
- G0-4 fails → for the Permission stage output **only** the "Not Executed" block specified in `references/permission-checklist.md` §3.3 (subset / status / unsatisfied IDs / blocking reasons / remediation actions), and stop. Output nothing else for Permission — no defect list, no score, no partial conclusions.
- Wording red line: the Permission stage was **"Not Executed"** — never report it as FAIL, and never write "Permission = ERROR/BLOCKED" at stage level (§9.2.4 hierarchy expression norm; see also §6 of this file).

### Stage D — Permission proportionality (QD-PM, Quick Mode)

1. Run Permission on the artifact types that are present (Prompt, Skill, and/or Tool Schema), not on a required three-artifact set. Items that depend on an absent type are N/A with basis (e.g. "no Tool Schema"). Record NOT EXECUTED only when no permission-evidence artifact exists at all.
2. Run the **17-item Quick Mode set** defined in `references/permission-checklist.md` §4 (the lite scope; the full spec has 43 items). For each item output `QD-PM-x.y | Applicable or N/A | PASS/FAIL | evidence`, and mark every verdict **"provisional (lite quick mode)"**.
3. Apply the checklist's classification binding (its §2): applicability is a structural fact from AC-L; severity is a risk fact from OR-L (basic mapping OR-L3→P0 / OR-L2→P1 / OR-L1→P2, with fixed-P0 items 3.4/5.4/6.3/6.5 and each item's inline severity rule). Never use AC-L to grade severity or OR-L to decide applicability. Every FAIL carries the classification-basis line `<operation> → <OR-L> → <rule> → <severity>`. N/A verdicts record the determination basis (a basis-less N/A is treated as failure per the checklist).
4. Include the checklist's mandatory scope statement (its §5): declared proportionality only — not IAM/PEP/runtime enforcement.

### Stage E — Explicit-risk audit (EX: NL-A / NL-B / NR-O / NR-S)

1. Use `references/explicit-checklist.md`. Scan the backend-logic artifacts (System Prompt, Skill, Tool Schema) for **explicitly expressed** risk — risk readable directly in static text, without running the Agent. This stage is detect + classify + grade, not PASS/FAIL inspection.
2. For each discovered risk item, emit one block: `EX-x.y | verbatim risk expression | impact type labels (D1/D2/D3) | severity grade (Explicit·P0/P1/P2) | trigger condition label | disposition`. Every finding must quote the offending artifact text (short verbatim excerpt).
3. Run the checklist's three-phase workflow (§11): Phase 1 discovery & classification (scan NL-A → NL-B → NR-O → NR-S, assign `EX` codes); Phase 2 severity determination (assign D1/D2/D3 impact type labels for traceability, grade by worst consequence into `Explicit·P0/P1/P2`, mark the trigger condition label `conditional` / `unconditional`); Phase 3 output (compile the report).
4. Red lines (mandatory):
   - Only emit `EX` codes that exist in `references/explicit-checklist.md` — **never invent EX IDs, subcategories, or levels**.
   - Write the severity grade **with the domain prefix** `Explicit·P0/P1/P2` — never bare `P0` — to distinguish Risk Explicit grading from Inspect defect disposition (`Inspect·P0`).
   - Assess only content **explicitly declared** in the risk text — never infer implicit risks (those belong to Risk Implicit, which is **not** run here).
   - Static text only: an EX finding is a static-document risk, **not** a confirmed runtime exploit. State explicitly that Risk Implicit (dynamic exploitability) was **not** run.
   - Keep this stage's report **physically separated** from the Inspect verdicts — it is a risk scan, not a defect verdict; an `Explicit·P0/P1/P2` grade is a Risk-Explicit assessment, never a security conclusion.
   - Risk Explicit produces a grade, not a numeric score: the §6.5 indicative score (deduction weights) applies to Inspect only, never to Risk Explicit.

### Stage F — Impact outlook (candidate mapping, advisory only)

1. For **each FAIL** from Stages A/B/D, look up `references/relevance-map.md`.
2. Emit a condensed Impact Card per mapped defect: defect ID → candidate attack surface(s) (AS-*) → candidate failure mode(s) (FM-*) → candidate RAG-Agent quality metric(s) → strength (Strong/Medium/Conditional).
3. Red lines (mandatory):
   - Candidate language only: "candidate association", "may", "needs dynamic verification". **Never** "will cause", "inevitably", "guarantees".
   - Strength ≠ severity. Label both explicitly.
   - State that Risk Implicit and the runtime Quality subsets were **not run** (Risk Explicit ran in Stage E; the lite embedded-code QC ran in Stage A), and that per relevance.md §1.4 no assumption is made that controls outside the artifacts are absent.
   - Keep this section **physically separated** from the stage verdicts (advisory context, not a verdict).
   - v1 scope: single-defect mapping only; defect chains (SC-*/QC-*) are out of scope.

### Stage G — Self-verification (lightweight, in-line + closing)

This stage verifies **the inspector's own report**, not the artifacts. It catches the host model's own execution errors — fabricated rule IDs, red-line wording, and output-contract violations — without re-inspecting the artifacts or re-reading the references from scratch. It keeps the zero-install posture (no scripts, no extra tools).

**In-line checks (run as you write each verdict/conclusion — never deferred to the end):**

1. **ID back-closure**: every `QD-*` / `QC-*` / `EX-*` ID you emit must exist in `references/`; every Stage F `AS-*` / `FM-*` mapping must exist in `references/relevance-map.md`. Confirm at write time; never emit an ID you cannot locate.
2. **Wording red lines**: never write causal language in Stage F ("will cause", "inevitably", "guarantees"); never declare the artifacts "secure" or "high quality"; never write "Permission = ERROR/BLOCKED". Check each concluding sentence as you write it.

**Closing checks (run once, at the end, on the assembled report):**

3. **Output contract**: the status board has exactly the seven required lines (§6.4); the report header carries all five provenance facts (§6.7); the executive summary is present (§6.1); the report is written in English (§6.9); and — zero-tolerance — the assembled report contains no CJK characters anywhere, including inside quoted evidence (§6.10).

**Closing reconciliation — judgment-quality pass (run once, at the end, on the report's own verdicts):**

This pass re-reads **the report's verdicts** — not the artifacts, not the references from scratch — to catch over-inspection and grading errors that the in-line checks miss. It applies only rules already present in the checklists (the five-state N/A discipline, conditional-trigger preconditions, OR-L3 upgrade mapping, instance-upgrade branches, same-source dedup). It never invents rules, never re-scores, and never rewrites artifacts. On an inconsistency, append a reconciliation note and re-emit that segment — never silently change a verdict.

4. **Tier self-audit**: re-confirm the AC-L (complexity → which items apply) vs OR-L (risk → severity) determination against the tiering rules. Flag mixed cases (e.g. an AC-L2 Agent carrying OR-L3 Tools) and any place one model was used where the other was required.
5. **N/A basis re-verification**: for every N/A, confirm a determination basis is recorded. Flag (a) a FAIL on a rule whose applicability precondition ("only when …") was not met, and (b) a conditional item left N/A without basis when its trigger was actually present.
6. **Upgrade justification audit**: for every P0 reached by an upgrade (OR-L3 mapping, or a rule's `→ P0` instance branch), confirm the trigger actually fired; conversely, flag a P1/P2 that a recorded trigger should have lifted.
7. **Cross-stage consistency**: a fact graded in more than one stage (e.g. a Skill permission boundary in Stage A QD-S-5.x vs Stage D QD-PM) must carry the same severity, respecting INHERITED and same-source dedup; flag any divergence.

Reconciliation notes are **non-blocking**: they do not re-open or re-run stages, and they are not new findings — they only flag an already-recorded verdict as over- or under-applied.

**On finding a self-error**: correct it in place and re-emit that segment. This stage fixes the report only — it never rewrites the inspected artifacts (the detect-only rule still holds).

**Output**: append a short self-verification statement after the status board, e.g. `Self-verification: ID back-closure OK · red-line wording OK · output contract OK · reconciliation OK` (list any self-corrections and reconciliation notes made).

## 5. Run modes

**Mode 1 — One-shot full flow (non-blocking)**: run A → B → C → D → E → F → G in one response. P0 findings are aggregated across stages and reported in the executive summary and status board; they **never** halt the flow. A stage is skipped only for concrete structural reasons (missing artifacts, or Gate-0 G0-4 unsatisfied), never for P0 findings.

**Mode 2 — Multi-round iteration**: the user fixes artifacts (outside this skill) and re-runs. Maintain a round ledger:

- Round number, artifact version, date, per-stage findings count (P0/P1/P2), list of open vs closed findings (matched by rule ID + evidence location).
- **Ratchet semantics**: for the same artifact version, a finding once recorded may only be closed by re-running its check on the updated artifact — never by user assertion. A version change resets the baseline (record it as a new baseline; keep prior rounds in the ledger).

**Anti-bypass rule (applies to all modes)**: a verbal "already fixed" never closes a P0 — only a re-scan of the updated artifact passing the same rule does. And in **no mode** does this skill generate fixed artifacts.

## 6. Output contract (hard)

1. **Executive summary** — emitted at the top of every full-flow report. A compact block: overall status (executed stages + P0/P1/P2 counts + Gate result + Risk-Explicit `Explicit·P0/P1/P2` summary), up to five top findings (`rule ID — one-line defect`), what was NOT run (Risk Implicit / runtime Quality), and an upgrade pointer (full inspection via `sanityops-cli` — see https://github.com/sanityops-org/sanityops-framework/blob/main/framework/try-the-tools.md). Never states a security or quality conclusion; reuse Stage F candidate wording.
2. **Per-rule verdicts**: `rule ID | PASS/FAIL | evidence location in artifact`. FAIL lines include a short verbatim excerpt of the offending text.
3. **No fabricated IDs** — only IDs present in `references/`.
4. **Status board** — always emitted, exactly these seven lines:

```
Stage A (Single-artifact):  Executed / Not Executed — <reason>
Stage B (Cross):            Executed / Not Executed — <reason>
Stage C (Gate-0):           Passed / Not passed — G0-4: <complete | vacuous (no Tool) | incomplete>; G0-3: <locatable | absent → 1.x N/A>
Stage D (Permission QD-PM): Executed / Not Executed — <reason>
Stage E (Risk Explicit EX): Executed / Not Executed — <reason>
Stage F (Impact outlook):   Executed / Not Executed — <reason>
Stage G (Self-verification): Executed / Not Executed — <reason>
```

5. **Blocking wording red line**: a stage that did not run is reported as **Not Executed** with a concrete structural reason — the specific Gate-0 condition ID (G0-4) or "no permission-evidence artifact" (Stage D). P0 findings alone are never a reason for a stage to be Not Executed in this non-blocking flow. Never report a non-executed stage as FAIL, and never write "Permission = ERROR/BLOCKED" — the correct form is "Permission did not execute due to Gate-0 not satisfied".
6. **Indicative score** only if the user asks: apply the checklist's stated deduction weights; label it "indicative (lite minimal set), not the full-spec score".
7. **Report header (mandatory on every report and every stage-only report)** — exactly these provenance facts:
   - Host model name (self-declared by the model executing this skill, e.g. `Host model: <vendor/model>`);
   - Skill package: `sanityops-inspect-lite v1.0.0` (the `version` value of this SKILL.md front matter);
   - Specification baseline: `SanityOps Framework v1.0 (2026-10 revision)`;
   - The non-official-result statement, verbatim in substance: *"This result is produced by the skill rule package applied by the host model; it is NOT a conclusion of the official SanityOps engine. For a full inspection use sanityops-cli."*
   - Inspection date and the artifact version label supplied by the user;
   - Fidelity tier per artifact set: `full` or `summary` (per-object) — when `summary`, state that absence-based rules were not run.

   The header is never omitted to save tokens, and must not imply the result is an official engine/SaaS conclusion.

8. **Self-verification statement** — after the status board, append the Stage G statement (ID back-closure / red-line wording / output contract), including any in-place self-corrections made. This is a self-check of the report, not a verdict on the artifacts.
9. **Report language (follow the user, hard)** — the report is written in the same language the user used to make the request. This covers all prose, findings, evidence explanations, the executive summary, the report header, the status board, and every templated block. **Rule IDs and labels are identifiers and are never translated**: `QD-*`, `QC-*`, `EX-*`, `AS-*`, `FM-*`, `G0-*`, severity labels `P0/P1/P2`, and tier labels `AC-L*`/`OR-L*` stay as-is regardless of the report language.
10. **Evidence excerpt language** — quote artifact text verbatim in its original language; render any explanatory gloss or finding summary in the report's language. (This replaces the former English-only and zero-CJK hard rules.)
11. **Terminology quick reference (mandatory)** — every report opens, directly after the executive summary, with a compact glossary mapping the ID families to plain language in the report's language (see Appendix A): `QD-*` = single/cross/permission defect · `QC-*` = embedded-code defect · `EX-*` = explicit risk · `AS-*`/`FM-*` = candidate impact mapping · `AC-L*`/`OR-L*` = agent/operation risk level · `P0/P1/P2` = blocking/warning/advisory.

## 7. Hard Do-Not list

- Never rewrite, patch, or generate fixed versions of any artifact.
- Never accept verbal fix claims without a re-scan.
- Never skip, reorder, or parallelize stages; run A → B → C → D → E → F → G in order.
- Never invent QD, EX, or QC IDs, rule names, levels, thresholds, or mappings.
- Never causalize relevance output; never present mapping strength as severity.
- Never claim the artifacts are "secure" or "high quality" — Inspect PASS ≠ Risk PASS ≠ Quality PASS; an EX finding is a static-document risk, never a confirmed exploit, and Permission quick mode says nothing about runtime enforcement.
- Never run on artifacts the user is not authorized to inspect; when in doubt, ask.
- Never run cross-artifact (Stage B), Permission (Stage C/D), or cross-object Risk-Explicit (Stage E NL-B) checks across artifacts from different agents.

## 8. Reference files

| File | Content | Source of authority |
|---|---|---|
| `references/prompt-checklist.md` | QD-P tiered checklist (AC-L1/2/3) + minimum rule sets (7/17/25 P0) | inspect/prompt.md v1.1 |
| `references/skill-checklist.md` | QD-S-0 baseline (9 items) + minimum rule sets (8/23/24) + six-step flow + Review Schema | inspect/skill.md v1.1 |
| `references/tool-checklist.md` | Tool operation-risk tiering (OR-L) + QD-T rule set + minimum rule sets (20/25/25) | inspect/tool.md v1.0 (2026-10 revision) |
| `references/code-contract-checklist.md` | QC-0 baseline + QC-1/2/3/4 embedded-code checks | quality/code-contract.md (DRAFT; checklist is the operational source) |
| `references/cross-checklist.md` | QD-PS/PT/ST/SS/TT/CYC checklist (29 items, 6 groups) + P0 minimum sets (7/9/9) | inspect/cross.md v1.0 (2026-10, content ahead of version label) |
| `references/permission-checklist.md` | Gate-0 (G0-1..4) + AC-L/OR-L binding + QD-PM Quick Mode 17 items | inspect/permission.md v1.0.2 |
| `references/explicit-checklist.md` | EX classification (NL-A/NL-B/NR-O/NR-S) + impact type labels (D1/D2/D3) + severity grading (Explicit·P0/P1/P2) + trigger condition label | risk/explicit.md v1.0 (2026-10 revision) |
| `references/relevance-map.md` | QD → candidate AS-*/FM-* mapping + candidate RAG-Agent metric mapping + red lines | framework/relevance.md v1.1 |

## 9. Provenance and license

Condensed from the SanityOps Framework v1.0 (2026-10 revision), maintained by Sanity AI Labs, license **CC BY-SA 4.0**. This package is a derived work under the same license; it defines no new rules. Specification source: `sanityops-org/sanityops-framework` — the implementing CLI (`sanityops-org/sanityops-cli`, Apache 2.0) is NOT required by this skill.

## 10. Further reading & upgrade

- **Upgrade to the full inspection**: the open-source `sanityops-cli` and the SanityOps Platform provide the complete Inspect ruleset plus the Risk (Explicit/Implicit) and Quality subsets, remediation, and version-level evidence chains. See https://github.com/sanityops-org/sanityops-framework/blob/main/framework/try-the-tools.md.
- **Background, lite-vs-full comparison, worked samples, FAQ**: see `README.md` (ships with this skill package).

## Appendix A — Terminology quick reference

| Family | Means | Plain meaning |
|---|---|---|
| `QD-P` / `QD-S` / `QD-T` | Inspect defect IDs | a defect in the Prompt / Skill / Tool Schema |
| `QD-PS/PT/ST/SS/TT/CYC` | Cross-artifact defect IDs | a conflict between two artifacts |
| `QD-PM` | Permission defect IDs | a permission-proportionality problem |
| `QC-*` | Code-contract defect IDs | a defect in an embedded code snippet |
| `EX-*` | Risk Explicit IDs | an explicitly-expressed risk (static text) |
| `AS-*` / `FM-*` | Impact-map IDs | candidate attack surface / failure mode (advisory) |
| `AC-L1/L2/L3` | Agent complexity level | how complex the agent is (drives which rules apply) |
| `OR-L1/L2/L3` | Operation risk level | how risky the operations are (drives severity) |
| `P0 / P1 / P2` | Severity | blocking / warning / advisory |
| `full` / `summary` | Fidelity | original file (all rules) vs summary (contradictions only) |

## Appendix B — Minimal example

**Input (what the user provides):** a single `system_prompt.md` — the only mandatory input; tiers/version are optional.

**Output (what the report looks like, condensed):**

```
Executive summary: Stage A executed (B/C/D vacuous). P0: 0 · P1: 2 · P2: 1.
Top findings:
  QD-P-2.2.1 — input source not declared
  QD-P-2.5.2 — security boundary constraints missing
...
Status board: Stage A Executed · Stage B Not Executed (no Tool/Skill) · ...
```

The user only needs to read the executive summary; the per-rule verdicts and Impact Cards are the detail behind it.
