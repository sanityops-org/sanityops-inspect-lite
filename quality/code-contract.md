---
title: Code Contract Inspection (QC)
version: 1.0
status: DRAFT
license: CC BY-SA 4.0
---

# Code Contract Inspection (QC)

> **DRAFT NOTICE**: This document is a **draft**, reverse-derived from `sanityops-inspect-lite`'s condensed checklist (`references/code-contract-checklist.md`). It is NOT an authoritative SanityOps Framework source. It defines no new rules relative to that checklist; it only restates, in source-spec form, the rules the checklist already contains. The checklist remains the operational source until this draft is reviewed and promoted.

> **Reverse-derivation provenance**: the `code-contract-checklist.md` file was authored directly for the lite inspector (no prior `quality/code-contract.md` existed). This document reconstructs the source-spec shape of that checklist: scope, terminology, numbering, defect levels, detection method, entry gate, disposition rules, the QC-0..QC-4 rule families, cross-spec relationships, exclusions, and the detection-rules index. No rule ID, name, level, or threshold is invented here.

## 1. Purpose and positioning

Code Contract inspection (the `QC` rule family) is the **static code contract** subset of the Quality dimension. It inspects **code snippets embedded in Logic Artifacts** — System Prompt, Skill, and Tool Schema — for four defect classes:

1. Syntax validation (`QC-1`)
2. Resource management (`QC-2`)
3. Code pattern (`QC-3`)
4. Embedding consistency (`QC-4`)

A `QC-0` baseline gates the rest. This is a **detect-only** check: it reports defects, never rewrites the inspected snippet or artifact.

## 2. Scope

- **In scope**: code snippets embedded in Logic Artifacts — System Prompt text, Skill descriptions/steps, and Tool Schema `description` / `default` / `example` fields.
- **Out of scope**: full source-code files (use standard linters for those). A script referenced by an artifact is a referenced file, not an embedded snippet.

## 3. Terms

| Term | Definition |
| --- | --- |
| **Logic Artifact** | A System Prompt, a Skill definition, or a Tool Schema. |
| **Embedded snippet** | A code block appearing inside a Logic Artifact's text or structured fields. |
| **Owning artifact** | The single Logic Artifact that contains the snippet under inspection. |

## 4. Numbering system

Rules use the form `QC-x.y`:

- `x` — the family: `0` = Baseline, `1` = Syntax, `2` = Resource, `3` = Code Pattern, `4` = Embedding Consistency.
- `y` — the item number within the family.

Only rule IDs defined in this document may be emitted. Never invent `QC` IDs, names, or levels.

## 5. Defect levels

| Level | Meaning |
| --- | --- |
| P0 | blocking |
| P1 | warning |
| P2 | advisory |

## 6. Detection method

Each rule is labeled with one method:

| Method | Meaning |
| --- | --- |
| Automated | Structural check, decidable directly from the snippet. |
| Semi-automated | Pattern matching combined with reasoning. |
| Pattern match | Pure pattern matching against the snippet text. |

> **Wording note (restated, not adjudicated)**: the checklist's "How to use" section summarizes the method as *"Automated for structural checks; Semi-automated for pattern matching"*, while the rule tables use three distinct labels (`Automated` / `Semi-automated` / `Pattern match`). This document reproduces the three table labels as written.

## 7. Entry gate

`QC-0` (Baseline Compliance) MUST pass before `QC-1` through `QC-4` inspection runs. A snippet that fails Baseline is not assessed against the later families.

## 8. Disposition rules

### 8.1 One verdict per rule

Each rule yields one verdict in the form:

```
QC-x.y | PASS/FAIL | evidence: file, line, excerpt
```

Every FAIL quotes the offending snippet text (short excerpt).

### 8.2 Dependency-context rule

`QC-1.2` (Invalid Import) and `QC-1.3` (Deprecated API Usage) require external inputs — a declared dependency list and an API version database — that are not part of the Logic Artifacts. When those inputs are absent, record the rule as **Not Executed**, never as FAIL. Flag FAIL only for an internally evident defect (for example, importing an obviously nonexistent module) or a self-contradiction inside the snippet.

### 8.3 Illustrative-snippet rule

A snippet whose enclosing text marks it as illustrative or pseudocode (for example, "example", "pseudo", elided with `...`) is exempt from the `QC-0.1` / `QC-0.3` parse-completeness requirements. Elision or omitted imports do not constitute a FAIL for such snippets.

## 9. QC-0 Baseline Compliance — 5 items

| ID | Name | Definition | Typical manifestation | Level |
| --- | --- | --- | --- | --- |
| QC-0.1 | Syntax Parsability | Code snippet parses as valid syntax for its declared language | `def foo(` missing closing `)`; YAML indentation error | P0 |
| QC-0.2 | Language Declaration | Code block has an explicit language identifier | Markdown code block without a ` ```python ` identifier or similar | P1 |
| QC-0.3 | Structural Completeness | Code snippet is complete enough for standalone parsing (has required imports, no mid-function truncation) | Function body truncated with `...`; missing `import` statements | P1 |
| QC-0.4 | Fence Integrity | Code fence opens and closes correctly; the closing fence is not swallowed by code content | Markdown code fence never closed, so trailing prose is absorbed into the block; mismatched backtick count | P1 |
| QC-0.5 | Language-Content Mismatch | Declared language identifier matches the actual snippet syntax | Fence declares `python` but the content is bash or JSON | P2 |

## 10. QC-1 Syntax Validation — 4 items

| ID | Name | Definition | Typical manifestation | Level | Method |
| --- | --- | --- | --- | --- | --- |
| QC-1.1 | Undefined Name Reference | Uses a class/function/variable not defined in the snippet or standard library | `Sacer(1, 0.2*inch)` — `Sacer` not defined (should be `Spacer`) | P1 | Automated |
| QC-1.2 | Invalid Import | Imports a module not available in the standard library or declared dependencies | `import custom_module` not in the dependency list | P2 | Automated |
| QC-1.3 | Deprecated API Usage | Uses a deprecated or removed API version | `pdfplumber.load()` instead of `pdfplumber.open()` | P2 | Semi-automated |
| QC-1.4 | Type Mismatch | Function argument types clearly incompatible | Passing a string to an integer parameter | P2 | Automated |

## 11. QC-2 Resource Management — 3 items

| ID | Name | Definition | Typical manifestation | Level | Method |
| --- | --- | --- | --- | --- | --- |
| QC-2.1 | Unclosed File Handle | File opened without a corresponding close or `with` statement | `pdf = pdfplumber.open("file.pdf")` without `pdf.close()` | P1 | Pattern match |
| QC-2.2 | Unreleased Network Connection | HTTP/socket connection opened without an explicit close or context manager | `conn = http.client.HTTPSConnection()` without `conn.close()` | P2 | Pattern match |
| QC-2.3 | Missing Exception Handling | High-risk operation (file/network/DB) without try-catch | `open(user_input)` without `try/except FileNotFoundError` | P2 | Pattern match |

## 12. QC-3 Code Pattern — 3 items

| ID | Name | Definition | Typical manifestation | Level | Method |
| --- | --- | --- | --- | --- | --- |
| QC-3.1 | Unused Import | Module imported but never referenced | `import numpy as np` with no `np.` usage | P2 | Automated |
| QC-3.2 | Missing Import | Module referenced but never imported | Uses `datetime.now()` without `import datetime` | P1 | Automated |
| QC-3.3 | Hardcoded Demo Value | Uses placeholder/demo/machine-specific values that may be mistaken for production | `password = "123456"`, `api_key = "xxx"`, local paths (`/Users/alice/...`, `C:\Users\...`), internal IP/hostname literals | P2 | Pattern match |

> `QC-3.3` overlaps with `NR-S-9` (Hardcoded Credential). `NR-S-9` takes precedence for actual credentials; `QC-3.3` covers weak/demo patterns that may not match credential detectors.

## 13. QC-4 Embedding Consistency — 2 items

Checks consistency between an embedded snippet and the artifact that embeds it. Unlike cross-artifact checks (Prompt↔Skill↔Tool), these are **intra-artifact**: snippet vs. its owning artifact's own declaration.

| ID | Name | Definition | Typical manifestation | Level | Method |
| --- | --- | --- | --- | --- | --- |
| QC-4.1 | Snippet-Artifact Declaration Mismatch | Snippet's calls/parameters contradict the owning artifact's own declaration (Tool Schema `properties`/`required`, or Skill/Prompt inline parameter/field names) | Tool `example` invokes a parameter absent from `properties`, or omits a `required` parameter; Skill step example uses a parameter name that differs from the Skill body | P1 | Semi-automated |
| QC-4.2 | Snippet-Body Version/API Drift | Snippet contradicts the owning artifact's own stated version/dependency/API (stale example) | Body states "use `pdfplumber.open()`", snippet still calls `pdfplumber.load()`; body declares a dependency the snippet does not import | P2 | Semi-automated |

## 14. Cross-spec relationships

- `QC-3.3` (Hardcoded Demo Value) overlaps with the Risk Explicit rule `NR-S-9` (Hardcoded Credential). `NR-S-9` governs actual credentials; `QC-3.3` covers weak/demo patterns that fall outside credential detectors.
- `QC` is the static code-contract subset of the Quality dimension. It is distinct from the Inspect rule families (`QD-*`) and the Risk Explicit family (`EX` / `NR-*`).

## 15. Does NOT check

- Full source-code files (use standard linters).
- Cross-artifact relationships between Prompt, Skill, and Tool (that is the cross-artifact check, not `QC-4`; `QC-4` is intra-artifact only).
- Runtime behavior or dynamic exploitability of the code (Risk Implicit / runtime Quality subsets).

## Appendix A — Detection rules index

| Subcategory | Detection strategy | Priority | Difficulty |
| --- | --- | --- | --- |
| QC-0.4 | Fence open/close matching | High | Low |
| QC-0.5 | Fence language vs content keyword match | Medium | Low |
| QC-1.1 | AST parsing + name resolution | High | Medium |
| QC-1.2 | Import statement extraction + dependency validation | Medium | Medium |
| QC-1.3 | API signature matching against version database | Low | High |
| QC-1.4 | Type inference from literals | Low | High |
| QC-2.1 | `open()` / `plumber.open()` pattern without `with` or `close()` | High | Low |
| QC-2.2 | Connection constructor pattern without context manager | Medium | Low |
| QC-2.3 | High-risk call pattern without try/except block | Medium | Medium |
| QC-3.1 | Import extraction vs reference counting | Low | Low |
| QC-3.2 | Reference extraction vs import matching | High | Low |
| QC-3.3 | Weak credential + machine-specific value pattern matching (sequential digits, "xxx", "changeme", local paths, internal IPs) | Medium | Low |
| QC-4.1 | Snippet symbol extraction vs owning artifact field/property match | Medium | Medium |
| QC-4.2 | Snippet API/version reference vs body statement match | Low | Medium |
