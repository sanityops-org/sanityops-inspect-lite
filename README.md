# SanityOps Inspect Lite

[![License: CC BY-SA 4.0](https://img.shields.io/badge/License-CC%20BY--SA%204.0-lightgrey.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)](package.json)

A **dependency-free, detect-only inspector** for AI Agent logic artifacts — **System Prompt**, **Skill**, and **Tool Schema**. It reads them and reports defects; it never rewrites them.

It covers the **full Inspect scope** — single-artifact, cross-artifact, permission — **plus the Risk Explicit audit** — using condensed rule sets rather than the full specification. P0 findings are aggregated and reported, but never halt the flow.

## Table of contents

- [An open-source tool from the SanityOps Framework](#an-open-source-tool-from-the-sanityops-framework)
- [What it checks](#what-it-checks)
- [Repository layout](#repository-layout)
- [Quick start](#quick-start)
- [Before you use it — five things to know](#before-you-use-it--five-things-to-know)
- [Two skills, chained](#two-skills-chained)
- [What can be inspected — and the input limits](#what-can-be-inspected--and-the-input-limits)
- [Run modes](#run-modes)
- [Example output](#example-output)
- [Lite vs full](#lite-vs-full)
- [Upgrade path](#upgrade-path)
- [License scope](#license-scope)
- [Contributing samples and rules](#contributing-samples-and-rules)
- [FAQ](#faq)

---

## An open-source tool from the SanityOps Framework

This skill is the free entry point to the [SanityOps Framework](https://github.com/sanityops-org/sanityops-framework) — an open specification (CC BY-SA 4.0) for the continuous governance of AI Agents. The framework has three pillars:

| Pillar | Question it answers | This skill |
|---|---|---|
| **Inspect** | Are the agent's logic artifacts correctly defined? | ✅ Full scope, condensed rules |
| **Risk** | Can the agent actually be exploited? | ⚠️ Explicit audit only |
| **Quality** | Does the agent actually perform well? | ⚠️ Static code-contract only |

The framework's core disclaimer applies throughout: **Inspect PASS ≠ Risk PASS ≠ Quality PASS.** For the full engine, use [`sanityops-cli`](https://github.com/sanityops-org/sanityops-cli) (Apache 2.0) or the SanityOps Platform — see [Upgrade path](#upgrade-path).

---

## What it checks

Seven stages, strictly in order, non-blocking:

| Stage | What it checks |
|---|---|
| **A** | Single-artifact defects — Prompt, Skill, each Tool Schema, plus embedded-code checks |
| **B** | Cross-artifact contracts — Prompt↔Skill, Prompt↔Tool, Skill↔Tool, and circular invocation |
| **C** | Gate-0 — admission to the permission stage (informational, not a verdict) |
| **D** | Permission proportionality — 17-item Quick Mode |
| **E** | Risk Explicit audit — dangerous expressions classified into 34 subcategories |
| **F** | Impact outlook — candidate attack-surface and failure-mode mapping (advisory) |
| **G** | Self-verification — checks the report itself, never the artifacts |

"Lite" refers to **rule depth**, not **scope**: every stage runs, but each uses condensed (minimal) rule sets instead of the full specification. The comparison table is in [Lite vs full](#lite-vs-full).

---

## Repository layout

```
sanityops-inspect-lite/
├── .claude-plugin/marketplace.json        # Claude Code plugin marketplace entry
├── plugins/sanityops-inspect-lite/
│   ├── .claude-plugin/plugin.json         # plugin manifest
│   └── skills/
│       ├── pre-inspect-extract/SKILL.md   # skill 1: extraction layer
│       └── sanityops-inspect-lite/
│           ├── SKILL.md                   # skill 2: seven-stage inspector
│           └── references/                # 8 condensed reference files
├── samples/                               # worked inspection examples
├── pre-inspect-harvester-spec.md          # extraction-layer design rationale
├── quality/code-contract.md               # embedded-code QC source (DRAFT)
├── GET_STARTED.md                         # step-by-step setup
├── LICENSE                                # CC BY-SA 4.0
├── package.json
└── README.md
```

---

## Quick start

Pick one path:

- **Claude Code (recommended):** `/plugin marketplace add sanityops-org/sanityops-inspect-lite`, then `/plugin install sanityops-inspect-lite@sanityops`, then say *"inspect my system prompt"*.
- **Any agent (clone):** `git clone https://github.com/sanityops-org/sanityops-inspect-lite.git`, then install both skills from `plugins/sanityops-inspect-lite/skills/` — `pre-inspect-extract/` and `sanityops-inspect-lite/`.
- **Full step-by-step:** see [GET_STARTED.md](GET_STARTED.md).

No build, no scripts, no runtime dependency — the skill is plain Markdown, installed either by a single plugin command (Claude Code) or by copying the `skills/` directory (any agent). Nothing runs at inspection time; a host model (the agent you run it in) executes the skill.

---

## Before you use it — five things to know

1. **Detect-only.** It reports defects; it never repairs, patches, or generates a fixed version.
2. **Not a security or quality gate.** Risk Implicit and runtime Quality metrics are not run. A PASS says nothing about whether the agent can actually be exploited in the wild.
3. **Permission checks declared scope only** — not IAM/IdP config, PEP/PDP enforcement, or runtime monitoring.
4. **Data stays where you run it.** The skill is pure Markdown with no scripts or network calls. Whether content leaves your environment depends on the host model you use.
5. **The report follows your language.** Ask in Chinese, get the report in Chinese; rule IDs stay as identifiers.

---

## Two skills, chained

This repository ships **two** skills that run in sequence:

```
arbitrary document  →  pre-inspect-extract  →  sanityops-inspect-lite  →  report
                        (pull artifacts out)     (seven-stage inspection)
```

- **[`pre-inspect-extract`](plugins/sanityops-inspect-lite/skills/pre-inspect-extract/SKILL.md)** — finds logic artifacts inside any document, strips the code/markup that carries them, reconstructs a schema from source when necessary, and labels each artifact's completeness.
- **[`sanityops-inspect-lite`](plugins/sanityops-inspect-lite/skills/sanityops-inspect-lite/SKILL.md)** — runs the seven stages on the extracted artifacts; its operational contract (inputs, flow, run modes, output format) lives in that SKILL.md.

The extraction layer labels every artifact with one of six completeness levels, which decides how much of it can be checked:

| Completeness level | Meaning | Checked |
|---|---|---|
| **full original** | Verbatim, complete | Everything |
| **original · variable unexpanded** | Verbatim, but variables expand into rules | Everything except checks on those variables |
| **assembled** | Schema reconstructed from code | Structure and contradictions; absence checks field-by-field |
| **incomplete** | Missing base fields | Only the fields present |
| **summary** | Summarized or transcribed | Contradictions only |
| **not inspectable** | Cannot extract | Nothing; reason recorded |

The extraction layer's design rationale is in [pre-inspect-harvester-spec.md](pre-inspect-harvester-spec.md).

---

## What can be inspected — and the input limits

- **Only three artifact types qualify:** System Prompt, Skill (Markdown with `name`/`description` front matter), Tool Schema (JSON Schema / function-calling definitions). A `## Skills` list or a bare function call is documentation, not an artifact — it is skipped, not inspected.
- **One agent at a time.** Cross-artifact, permission, and risk stages never mix agents. Up to 50 artifacts per run (1 Prompt + 1 Skill + up to 48 Tool Schemas); a run total up to 200,000 tokens.
- **Completeness decides how much is checked.** A verbatim original runs every rule; a summary is checked for contradictions only. Submitting a summary and expecting a full verdict will mislead.

## Run modes

- **One-shot** — the full seven-stage flow in one response; P0s never halt it.
- **Multi-round** — fix artifacts externally and re-run, with a round ledger and Ratchet semantics (a finding closes only by re-scanning the updated artifact, never by assertion).
- **Collection mode** — user-declared; inspects each artifact in isolation for a collection of unrelated artifacts.

---

## Example output

A condensed report looks like this (from SKILL.md Appendix B):

```
Executive summary: Stage A executed (B/C/D vacuous). P0: 0 · P1: 2 · P2: 1.
Top findings:
  QD-P-2.2.1 — input source not declared
  QD-P-2.5.2 — security boundary constraints missing
Status board: Stage A Executed · Stage B Not Executed (no Tool/Skill) · ...
```

Full reports carry a seven-line status board, an executive summary, per-rule verdicts with verbatim evidence, and a self-verification statement. Worked instances are in [`samples/`](samples/).

---

## Lite vs full

| Capability | Lite (this skill) | Full (CLI / Platform) |
|---|---|---|
| Inspect rules | Condensed minimal sets | Full specification |
| Permission | 17-item Quick Mode | Full 43-item set |
| Risk Explicit | 34-subcategory classification + grading | Full classification |
| Risk Implicit | Not run | Run (dynamic exploitability) |
| Quality — runtime | Not run | Run (RAG/Tool-Agent metrics) |
| Remediation | None (detect-only) | `inspect repair` generates fixes |
| Evidence chain | None | Platform archives auditable results |

The comparison is illustrative; on any specific number the owning sub-specification governs.

---

## Upgrade path

- **Open-source CLI** (full Inspect + remediation): [sanityops-org/sanityops-cli](https://github.com/sanityops-org/sanityops-cli) (Apache 2.0).
- **SanityOps Platform** (Risk + Quality + root-cause analysis): <https://demo.sanityops.org/> (invite-only).
- **Enterprise self-hosted**: contact <mailto:hello@sanityops.org>.

---

## License scope

- Rule content (`SKILL.md`, `references/`, `samples/`): **CC BY-SA 4.0**.
- **Using this skill to inspect your own artifacts is not an adaptation** — your artifacts and your project keep their own license. Share-Alike applies only to modified versions of this package.
- **To integrate the capability into a product or service**, use `sanityops-cli` (Apache 2.0) — the recommended path for embedding.

---

## Contributing samples and rules

See [GET_STARTED.md](GET_STARTED.md) for usage; contributions welcome. **Before contributing a sample that quotes a third-party artifact, verify the upstream license permits redistribution under CC BY-SA 4.0** — proprietary or no-derivative material cannot be included.

---

## FAQ

**Is this the official SanityOps tool?**
No. It is a condensed skill-rule package applied by a host model. For the official result, use `sanityops-cli`.

**Can it fix my artifacts?**
No. It is detect-only. The CLI provides remediation via `sanityops-cli inspect repair`.

**Does it check security or quality?**
It runs a lite Risk Explicit scan and a lite static code-contract check, but not Risk Implicit or runtime Quality metrics. Risk Explicit findings are static-document risks, never confirmed exploits.

**Why not just use a linter or a general LLM review?**
Linters check code, not agent logic contracts (prompt text, skill definitions, tool schemas). A general LLM review catches *written* contradictions but misses *missing-declaration* defects — the absence of a security boundary, an input source, or an output format — which is exactly what the completeness checks (Stage A Part 2) are built to catch.

**Where are the authoritative rules?**
In the [SanityOps Framework](https://github.com/sanityops-org/sanityops-framework) sub-specifications. The files in `references/` are condensed extractions and define no new rules.
