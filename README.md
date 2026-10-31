---
title: SanityOps Inspect Lite — README
description: "Orientation for the sanityops-inspect-lite inspector (SanityOps Framework v1.0): what to read first, where the skill sits among the framework's Inspect / Risk / Quality pillars, what it is and is not, the seven ordered inspection stages, lite-vs-full comparison, upgrade path, worked samples, and FAQ. Non-normative auxiliary guide."
---

# SanityOps Inspect Lite — README

This file is the **auxiliary guide** for the `sanityops-inspect-lite` inspector. It covers the reading order, where the skill sits in the framework, what the seven ordered inspection stages are, how to use it, the lite-vs-full comparison, how to upgrade to the official tooling, worked samples, and FAQ. For the operational rules — inputs, the seven-stage flow, run modes, and the output contract — see [SKILL.md](plugins/sanityops-inspect-lite/skills/sanityops-inspect-lite/SKILL.md).

This is a **non-normative** document. It defines no new rules and never overrides the SanityOps Framework specification or the condensed checklists in `references/`.

---

## What to read first

This package is read by two different audiences, so the entry point differs:

| Reader | Read first | Then |
|---|---|---|
| **A human** adopting, reviewing, or evaluating the package | **this README** — orientation: what it is, where it sits, what it does *not* do | [SKILL.md](plugins/sanityops-inspect-lite/skills/sanityops-inspect-lite/SKILL.md) for the rule-driven flow, then its `references/` and `samples/` |
| **A host model** executing an inspection | **`SKILL.md`** — the operational contract: inputs, the seven ordered stages, run modes, output contract | the `references/*.md` checklists it points to, at the moment each stage runs |

The README is not a substitute for `SKILL.md`, and `SKILL.md` is not an introduction: the README answers *"should I use this, and what exactly is it?"*, while `SKILL.md` answers *"what must I do, in what order, and what must I emit?"*. A human can skip the README and still run the skill, but will miss the positioning in the next two sections.

---

## Background

### Where this skill sits — Inspect is one of three pillars

The SanityOps Framework v1.0 is an open specification (CC BY-SA 4.0) for continuous governance of AI Agents. It has **three pillars**:

| Pillar | Question it answers | Nature |
|---|---|---|
| **Inspect** | Are the agent's *logic artifacts* correctly and safely **defined**? | Static — defect inspection of System Prompt / Skill / Tool Schema |
| **Risk** | Can the agent be *actually exploited*? | Dynamic — Explicit (dangerous-expression classification) and Implicit (shadow-sandbox exploitability validation) |
| **Quality** | Does the agent *actually perform* well? | Dynamic — RAG-Agent / Tool-Agent metrics measured on real runs |

**`sanityops-inspect-lite` is a lightweight practice of the Inspect pillar plus a condensed Risk Explicit subset and a lite static code-contract (QC) check.** It never runs Risk Implicit or the runtime Quality pillar (RAG-Agent / Tool-Agent metrics); where it touches those it emits *candidate* associations, never results (see [The seven ordered inspection stages](#the-seven-ordered-inspection-stages)). The framework's core disclaimer applies throughout: **Inspect PASS ≠ Risk PASS ≠ Quality PASS.**

Within Inspect itself, the three single-artifact checks (**System Prompt**, **Skill**, **Tool Schema**) are the *starting point*, not the whole subset — Permission is a cross-cutting aspect, and cross-artifact consistency, explicit-risk audit, and the impact outlook are checked as separate stages (see [The seven ordered inspection stages](#the-seven-ordered-inspection-stages)).

## What it is — and what it is not

**What it is.** A zero-install, detect-only inspector for the three Logic Artifact types — **System Prompt**, **Skill**, and **Tool Schema** — executed by the host model, which applies the condensed checklists in `references/`. No executable, no install step, no network call.

**What it is not:**

- **Not the official engine.** A condensed skill-rule package applied by a host model; it covers only the minimal rule sets, not the full specification. The official tooling is `sanityops-cli` and the SanityOps Platform (see [Upgrade path](#upgrade-path)).
- **Not a security or quality gate.** Inspect PASS ≠ Risk PASS ≠ Quality PASS; the Risk Implicit and runtime Quality subsets (RAG-Agent / Tool-Agent metrics) are **not** run by this skill — only the static embedded-code contract check (QC) is run, as part of Stage A.
- **Not a repair tool.** It never rewrites, patches, or generates fixed versions of any artifact.
- **Not a runtime auditor.** Permission quick mode validates only what is *declared in the artifacts* — not IAM/IdP configuration, PEP/PDP enforcement, or runtime monitoring.

## The seven ordered inspection stages

The skill runs **seven stages, strictly in this order** — never in parallel, never out of order. The three single-artifact checks are the *first* of seven, not the whole inspection.

| Stage | Subset | What it checks | Gating |
|---|---|---|---|
| **A** | QD-P / QD-S / QD-T / QC | **Single-artifact** defects — one System Prompt, one Skill (QD-S-0 baseline runs first), each Tool Schema, plus embedded-code contract checks (QC-0 baseline runs first) | — |
| **B** | QD-PS / QD-PT / QD-ST / QD-SS / QD-TT / QD-CYC | **Cross-artifact** contracts — Prompt↔Skill, Prompt↔Tool, Skill↔Tool, Skill↔Skill, Tool↔Tool, plus cross-cutting circular invocation (QD-CYC, L2/L3 only) | — |
| **C** | Gate-0 | Admission to Permission (conditions G0-1..G0-4). *Not an inspection item* — produces no score and no defect list | G0-3/G0-4 fail → Permission is "Not Executed" |
| **D** | QD-PM | **Permission** proportionality, Quick Mode (the 17 highest-risk of the full 43 items); requires the full artifact set | — |
| **E** | EX (NL-A / NL-B / NR-O / NR-S) | **Explicit-risk audit** — classify dangerous expressions into the 34 subcategories, assign impact type labels (D1/D2/D3), grade by worst consequence into `Explicit·P0/P1/P2`, mark the trigger condition label. Static text only; never a confirmed exploit | — |
| **F** | relevance map | **Impact outlook** — for each FAIL, *candidate* attack surface (`AS-*`), failure mode (`FM-*`), and quality-metric associations. Advisory only, kept physically separate from verdicts | — |
| **G** | self-verification | **Self-verification** — verifies the inspector's own report (ID back-closure, red-line wording, output contract), in-line as each verdict is written plus a closing pass. Fixes the report only, never the artifacts | — |

**Why the order matters.** C (Gate-0) gates D; E is the explicit-risk scan (static, independent of the QD findings); F consumes the FAILs from A/B/D; G then self-verifies the emitted report (it never re-inspects the artifacts). The flow is **non-blocking**: P0 findings are aggregated and reported but never halt the flow — a stage is reported **"Not Executed"** only for structural reasons (missing artifacts, or Gate-0 G0-3/G0-4 unsatisfied), never because of P0 findings. Three consequences worth internalizing:

- A single-artifact PASS does **not** mean the artifact set is consistent — Stage B exists precisely for the conflicts that surface only when artifacts are combined.
- A Permission PASS is scoped to what is **declared in the artifacts**; it says nothing about IAM/PEP configuration or runtime enforcement.
- Stages **B, D, E and F** (cross-artifact, permission, explicit-risk audit, impact outlook) are first-class parts of the inspection, not optional add-ons. Reading this package as "just the three single-artifact checks" is the most common misreading.

## How to use

The skill is executed by a **host model** (the agent you run it in). There is no executable, no install step, and no network call.

### Install as a Claude Code plugin

This repo doubles as a Claude Code plugin marketplace. Add the marketplace, then install the plugin:

```
/plugin marketplace add sanityops-org/sanityops-inspect-lite
/plugin install sanityops-inspect-lite@sanityops
```

The plugin is a **pure skill** — no hooks, no MCP servers, no scripts. Once installed, it auto-triggers when you ask to inspect, audit, or check an agent's System Prompt, Skill, or Tool Schema.

### Install as a standalone skill

For a non-Claude-Code agent, the skill package is `plugins/sanityops-inspect-lite/skills/sanityops-inspect-lite/` — the `SKILL.md` plus the `references/` checklists it points to.

1. **Get the package.** Clone this repo:

   ```
   git clone https://github.com/sanityops-org/sanityops-inspect-lite.git
   ```

2. **Load it into an agent.**
   - **Agent skill (recommended):** install the `plugins/sanityops-inspect-lite/skills/sanityops-inspect-lite/` folder (or just its `SKILL.md` + `references/`) as a skill. The `description` in `SKILL.md`'s front matter auto-triggers it when the user asks to inspect, audit, or check logic artifacts.
   - **Paste-in fallback:** provide `SKILL.md` together with the `references/` files as instructions — the model reads the checklists while running the flow.

3. **Provide the inputs** (SKILL.md §2): the artifacts to inspect (System Prompt / Skill / Tool Schema), the tier declarations (Prompt/Skill complexity level AC-L and each Tool's operation-risk level OR-L), and a version label. The model then runs the seven-stage flow and emits the report.

## What can be inspected — and the input limits

Three things are worth knowing *before* you hand artifacts over; ignoring them is the main way to get misleading output.

**1. Only three artifact types qualify.** The skill inspects a System Prompt, a Skill (Markdown with `name`/`description` front matter, e.g. `SKILL.md`), and Tool Schemas (JSON Schema / function-calling definitions with `parameters` + `description`). A `## Skills` list, a bare function call, or prose that merely *describes* a capability is documentation, not an artifact — it has no structural header, so it cannot be loaded, and the skill skips it rather than reporting phantom defects against it (SKILL.md §2 "Type classification").

**2. One agent at a time, with bounded inputs.** All artifacts in a run must belong to the same agent; the cross-artifact, permission, and risk stages never mix agents. Input budget (SKILL.md §2 "Input budget"): up to **50 artifacts** (1 Prompt + 1 Skill + up to 48 Tool Schemas); per-type size caps — Tool Schema ≤ 2,000 tokens, System Prompt ≤ 30,000 tokens, Skill definition (incl. its referenced files) ≤ 100,000 tokens; a run total up to **200,000 tokens** on a 1M-token host window (scaled down on smaller windows). Over the limit, split the run into rounds, or the artifact is treated as `summary` fidelity.

**3. Fidelity decides how much is checked.** A provably original, verbatim artifact (`full`) runs every rule. A summary, excerpt, or transcription (`summary`) is checked for contradictions only — absence-based findings are skipped (SKILL.md §2 "Step 0"). Submitting a summary and expecting a full verdict will mislead. Output size is not capped; the report is emitted in full.

## Lite vs full comparison

| Capability | Lite inspector (this skill) | Full (CLI / SanityOps Platform) |
|---|---|---|
| Inspect rule coverage | Condensed minimal sets (QD-P/QD-S/QD-T; Cross AC-L1 7 / AC-L2 20 / AC-L3 29 items) | Full specification |
| Permission check | 17-item Quick Mode | Full 43-item set |
| Risk Explicit (`EX-*`) | Lite subset (34-subcategory classification + D1/D2/D3 impact labels + `Explicit·P0/P1/P2` grading + trigger condition label) | Run (full dangerous-expression classification) |
| Risk Implicit | Not run | Run (shadow-sandbox dynamic exploitability validation) |
| Quality — static code contract (QC) | Lite subset (embedded-code syntax / resource / pattern checks) | Run (full code-contract rule set) |
| Quality — runtime (RAG-Agent / Tool-Agent) | Not run (candidate `FM-*`/metric mapping only) | Run (4-dimension 12 metrics, three-layer gates, Ratchet) |
| Defect chains (`SC-*` / `QC-TA-*` / `QC-RA-*`) | Out of scope | Supported |
| Remediation / repair | None (detect-only) | CLI `inspect repair` generates fixed artifacts |
| Version-level evidence chain | None | Platform archives results into a traceable, auditable evidence chain |

The comparison above is illustrative; on any specific number, the owning sub-specification governs.

## Upgrade path

- **Open-source CLI** (full Inspect + suggestions + remediation): [sanityops-org/sanityops-cli](https://github.com/sanityops-org/sanityops-cli) (Apache 2.0).
- **SanityOps Platform** (Risk + Quality + correlated root-cause analysis + version evidence chain): live demo at <https://demo.sanityops.org/> (invite-only early access; request a code via <mailto:hello@sanityops.org>).
- **Enterprise self-hosted** (code-reviewable under NDA): contact <mailto:hello@sanityops.org>.

Install and quick-start instructions are in [Try the Tools](https://github.com/sanityops-org/sanityops-framework/blob/main/framework/try-the-tools.md).

## Worked samples

Instance outputs of the lite inspector (non-normative `F-*`-level records) live in `samples/`:

| Sample | Subject |
|---|---|
| [gpt-researcher.md](samples/gpt-researcher.md) | gpt-researcher Prompt + Tool — Stage A QD-P / QD-T |
| [crewai-markdown-validator.md](samples/crewai-markdown-validator.md) | crewAI-examples markdown_validator Prompt + Tool |
| [tau-bench-airline.md](samples/tau-bench-airline.md) | sierra-research/tau-bench airline (Prompt + 14 Tool Schemas) |
| [tau-bench-retail.md](samples/tau-bench-retail.md) | sierra-research/tau-bench retail (Prompt + 16 Tool Schemas) |
| [self-check-report.md](samples/self-check-report.md) | Dogfooding: the lite inspector applied to its own package (Round 1 + Round 2) |

## FAQ

**Is this the official SanityOps tool?**
No. It is a condensed skill-rule package applied by a host model. For the official result, use `sanityops-cli` (see [Upgrade path](#upgrade-path)).

**Can it fix my artifacts?**
No. It is detect-only. The CLI provides remediation via `sanityops-cli inspect repair`.

**Does it check security or quality?**
It runs a lite Risk Explicit scan (static dangerous-expression classification with `Explicit·P0/P1/P2` grading) and a lite static code-contract check (QC, on embedded code snippets) but not Risk Implicit or the runtime Quality metrics. Risk Explicit findings are static-document risks, never confirmed exploits; Risk Implicit (dynamic exploitability) and the runtime Quality subsets (RAG-Agent / Tool-Agent metrics) are run by the full tooling, not by this skill.

**Where are the authoritative rules?**
In the nine L1 sub-specifications under `inspect/`, `risk/`, and `quality/`, plus `framework/relevance.md`. The files in `references/` are condensed extractions and define no new rules.
