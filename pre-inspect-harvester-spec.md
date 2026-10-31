---
title: Pre-Inspection Extraction Layer — Design Rationale
version: 0.5.3
status: Explanatory document; the rules live in pre-inspect-extract/SKILL.md
execution_mode: pure-prompt (no scripts, zero dependencies)
---

# Pre-Inspection Extraction Layer — Design Rationale

## 0. What this document is

**It contains no rules.** There is exactly one source of rules: [pre-inspect-extract/SKILL.md](plugins/sanityops-inspect-lite/skills/pre-inspect-extract/SKILL.md). The glossary is also at the top of that file.

This document answers a different question: **why it is designed this way.** Each decision is followed by its basis, drawn from review discussion and from re-running against real cases. Read this before changing a rule, so that already-validated pitfalls are not re-entered.

---

## 1. Why this layer is needed

All seven stages of `sanityops-inspect-lite` assume that "the artifact has already been correctly separated out". But what a user hands us is usually a single document mixing metadata, evaluation dimensions, code, and examples. The artifact may be wrapped in an f-string, or there may be no ready-made schema at all — only something that has to be assembled from Zod, decorators, or a Go builder.

If this layer does its job poorly, all seven downstream stages collapse: either the code shell gets inspected as if it were a prompt, producing a pile of phantom defects, or the object is declared "not inspectable" and skipped, which is the same as not inspecting at all.

So this layer is deliberately narrow: **find the artifact, pull it out, state how completely it was pulled out, and keep verbatim evidence. It never judges defects.**

---

## 2. Key decisions and their basis

| # | Decision | Basis |
|---|---|---|
| 1 | Ship as a standalone skill, pure prompt, no scripts | The host environment does not make it convenient to introduce programmatic capability inside an inspection skill; reliability is instead carried by "verbatim copy + self-evidence against the original" |
| 2 | Fixed five-step flow: segment → classify → extract → set completeness level → attach evidence | Every step produces something independently checkable; when something goes wrong, it can be located to a step |
| 3 | Everything in a document falls into four classes: artifact / carrier / scaffolding / referenced file | Without this separation, carriers get treated as artifacts. In real cases, prompts are very often wrapped inside functions and strings |
| 4 | Only three classification tiers, plus one "unrecognizable" bucket | Tier 1 is directly readable, tier 2 is wrapped in code, tier 3 requires assembly. Judged by "is this text carried by code", not by enumerating syntax |
| 5 | **Classification does not enumerate syntax** | The first version listed only `f"""` and backtick templates; re-running showed that docstrings (dspy.Signature) and plain quoted strings were both missed entirely |
| 6 | Only two extraction methods: verbatim copy and assembly | Verbatim copy introduces no error; assembly is unavoidably lossy, so it must be labelled separately |
| 7 | **Assembly re-assembles; it never translates** | Asking a model to "understand Zod semantics and regenerate a JSON Schema" fabricates fields, loses `.describe()`, and is not reproducible. Replaced by field-by-field copying plus fixed mappings |
| 8 | Loss of a description field is treated as a top-priority incident | Tool Schema inspection rests almost entirely on `description`; lose the description and every conclusion is wrong |
| 9 | Six completeness labels: full original / original · variable unexpanded / assembled / incomplete / summary / not inspectable | The label directly determines how many checks can run downstream; a reader must see at a glance "how far can this be trusted" |
| 10 | A separate "original · variable unexpanded" label | Re-running found prompts that put rules inside variables (e.g. `{reference_prompt}`). The literal string does not contain those rules, so submitting it as a full original would mass-produce false failures |
| 11 | A separate "incomplete" label | Some cases have only name/description and no schema. Calling them "not inspectable" throws away the description quality that could still be checked; calling them "complete" turns "inputSchema missing" into a phantom defect. A separate tier checks only the fields that exist |
| 12 | **For an assembled artifact, decide checkability field by field; do not reject the whole object** | An earlier version skipped all absence-type checks on assembled artifacts, which was too conservative. If the evidence excerpt covers a field's complete definition, that is enough to conclude "the original really does not declare it" — evidence quality decides the conclusion, not the label |
| 13 | Treat `.refine()` as a genuine defect candidate | `.refine()` normally does not make it into the JSON Schema generated at runtime. When a description promises a constraint that the schema does not express, that is a real QD-T-4.5 problem and must not be erased as "cannot be assembled" |
| 14 | Evidence is an excerpt, not a line number or checksum | Without scripts, model-reported line numbers are unreliable; an excerpt can be checked by a human and matched directly against the original |
| 15 | A document is assumed to hold one agent by default; when multiple agents are detected, neither auto-split nor auto-return — label it "agent attribution to be confirmed" and hand it to the user (same agent, continue / declare collection mode to inspect artifacts one by one) | When multiple agents share a document, cross-artifact checks cross wires; auto-splitting easily splits wrongly, while auto-returning is too harsh for users doing batch single-artifact inspection. Let the user declare intent; the pre-skill does not decide on their behalf |
| 16 | If it cannot be recognized, mark it not inspectable rather than force a verdict | A forced verdict produces phantom defects, which is worse than not inspecting |
| 17 | Segmentation must report the artifact count | One file often contains several artifacts (e.g. several tools in one file, or several prompt fragments in one file). A missed cut is a missed inspection |

---

## 3. Known limitations (cannot be removed by changing rules)

| Limitation | Impact |
|---|---|
| Pure prompt, no deterministic parsing | Long texts copied verbatim can still drift; when an excerpt cannot be matched to the original, it can only be downgraded to summary |
| Assembled artifacts depend on evidence quality | For fields the evidence does not cover, the related checks can only be skipped |
| Prompts with unexpanded variables | Checks that depend on those variables are permanently missing |
| Not inspectable | Not checked at all; only the reason is recorded |
| No samples for cross-artifact inspection | Each file in the current case set contains only one artifact type; checks such as prompt↔tool still await validating samples |
| Does not replace code-level review | Logic defects in the source as a whole (business errors, concurrency, data flow) are out of scope |

---

## 4. Revision history

| Version | Change |
|---|---|
| 0.1–0.3 | Established the five-step flow, four object classes, three identification tiers, verbatim-copy/assembly extraction, four completeness levels, excerpt evidence, and the single-agent premise |
| 0.4 | After the first full re-run: added JS/TS object literals and pure JSON to tier 1; changed tier 2 to be judged by carrier rather than syntax (adding docstrings, plain quoted strings, function return values); added Python decorators and the Go builder to tier 3; moved Go struct+tag and dynamic registration to not-inspectable; added the "variable unexpanded" and "incomplete" labels |
| 0.5 | Terminology fully localized to Chinese (assembled artifact, completeness level, etc.), removing invented labels; this document narrowed to design rationale and re-run data only, with rules and terminology carried solely by `SKILL.md` to prevent the two documents from diverging; assembled artifacts judged field by field; `.refine()` reclassified as a genuine defect candidate |
| 0.5.1 | Multi-agent handling changed from "return for confirmation" to "await user confirmation" (same agent, continue / declare collection mode to inspect artifacts one by one); collection mode became a user-declarable run mode of the inspect skill (Mode 3) |
| 0.5.2 | Removed internal re-run validation data (the former §3 in full, plus specific counts and test-case IDs throughout), keeping only universal design rationale |
| 0.5.3 | Translated the document to English; no change to any decision or wording in substance |
