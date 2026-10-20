---
title: Code Contract Inspection Checklist (Lite)
description: "Condensed QC (Code Contract) inspection checklist for the sanityops-inspect lite inspector: syntax validation, resource management, code pattern defects, and embedding consistency in example code snippets within Logic Artifacts. Extracted from quality/code-contract.md v1.0; defines no new rules."
---

# Code Contract Inspection Checklist (QC, Lite)

> **Provenance**: Condensed from `quality/code-contract.md` v1.0 (SanityOps Framework, CC BY-SA 4.0). This file is a compressed extraction for the sanityops-inspect lite inspector and defines NO new rules. On any conflict, `quality/code-contract.md` governs.

## 1. How to use

- **Scope**: Inspect code snippets embedded in Logic Artifacts (System Prompts, Skill descriptions/steps, Tool Schema description/default/example fields). Does NOT inspect full source code files — use standard linters for those.
- **Entry gate**: QC-0 Baseline MUST pass before QC-1 through QC-4 inspection.
- **One verdict per rule**: `QC-x.y | PASS/FAIL | evidence: file, line, excerpt`.
- **Defect levels**: P0 (blocking) / P1 (warning) / P2 (advisory).
- **Method**: Automated for structural checks; Semi-automated for pattern matching.
- **Dependency-context rule**: QC-1.2 (Invalid Import) and QC-1.3 (Deprecated API Usage) require external inputs — a declared dependency list and an API version database — that are not part of the Logic Artifacts. When those inputs are absent (the lite default), record the rule as `Not Executed`, never as FAIL; flag FAIL only for an internally evident defect (e.g. importing an obviously nonexistent module) or a self-contradiction inside the snippet.
- **Illustrative-snippet rule**: a snippet whose enclosing text marks it as illustrative or pseudocode (e.g. "example", "pseudo", elided with `...`) is exempt from the QC-0.1 / QC-0.3 parse-completeness requirements; elision or omitted imports do not constitute a FAIL for such snippets.

## 2. QC-0 Baseline Compliance (entry gate) — 5 items

| ID | Name | Definition (condensed) | Typical manifestation | Level |
| --- | --- | --- | --- | --- |
| QC-0.1 | Syntax Parsability | Code snippet parses as valid syntax for its declared language | `def foo(` missing closing `)`; YAML indentation error | P0 |
| QC-0.2 | Language Declaration | Code block has explicit language identifier | Markdown code block without ` ```python ` or similar | P1 |
| QC-0.3 | Structural Completeness | Code snippet is complete enough for standalone parsing (has required imports, no mid-function truncation) | Function body truncated with `...`; missing `import` statements | P1 |
| QC-0.4 | Fence Integrity | Code fence opens and closes correctly; the closing fence is not swallowed by code content | Markdown code fence never closed, so trailing prose is absorbed into the block; mismatched backtick count | P1 |
| QC-0.5 | Language-Content Mismatch | Declared language identifier matches the actual snippet syntax | Fence declares `python` but the content is bash or JSON | P2 |

## 3. QC-1 Syntax Validation — 4 items

| ID | Name | Definition (condensed) | Typical manifestation | Level | Method |
| --- | --- | --- | --- | --- | --- |
| QC-1.1 | Undefined Name Reference | Uses class/function/variable not defined in snippet or standard library | `Sacer(1, 0.2*inch)` — `Sacer` not defined (should be `Spacer`) | P1 | Automated |
| QC-1.2 | Invalid Import | Imports module not available in standard library or declared dependencies | `import custom_module` not in dependency list | P2 | Automated |
| QC-1.3 | Deprecated API Usage | Uses deprecated or removed API versions | `pdfplumber.load()` instead of `pdfplumber.open()` | P2 | Semi-automated |
| QC-1.4 | Type Mismatch | Function argument types clearly incompatible | Passing string to integer parameter | P2 | Automated |

## 4. QC-2 Resource Management — 3 items

| ID | Name | Definition (condensed) | Typical manifestation | Level | Method |
| --- | --- | --- | --- | --- | --- |
| QC-2.1 | Unclosed File Handle | File opened without corresponding close or `with` statement | `pdf = pdfplumber.open("file.pdf")` without `pdf.close()` | P1 | Pattern match |
| QC-2.2 | Unreleased Network Connection | HTTP/socket connection opened without explicit close or context manager | `conn = http.client.HTTPSConnection()` without `conn.close()` | P2 | Pattern match |
| QC-2.3 | Missing Exception Handling | High-risk operation (file/network/DB) without try-catch | `open(user_input)` without `try/except FileNotFoundError` | P2 | Pattern match |

## 5. QC-3 Code Pattern — 3 items

| ID | Name | Definition (condensed) | Typical manifestation | Level | Method |
| --- | --- | --- | --- | --- | --- |
| QC-3.1 | Unused Import | Module imported but never referenced | `import numpy as np` with no `np.` usage | P2 | Automated |
| QC-3.2 | Missing Import | Module referenced but never imported | Uses `datetime.now()` without `import datetime` | P1 | Automated |
| QC-3.3 | Hardcoded Demo Value | Uses placeholder/demo/machine-specific values that may be mistaken for production | `password = "123456"`, `api_key = "xxx"`, local paths (`/Users/alice/...`, `C:\Users\...`), internal IP/hostname literals | P2 | Pattern match |

> **Note**: QC-3.3 overlaps with NR-S-9 (Hardcoded Credential). NR-S-9 takes precedence for actual credentials; QC-3.3 covers weak/demo patterns that may not match credential detectors.

## 6. QC-4 Embedding Consistency — 2 items

Checks consistency between an embedded snippet and the artifact that embeds it. Unlike Stage B cross-checks (Prompt↔Skill↔Tool), these are intra-artifact: snippet vs. its owning artifact's own declaration.

| ID | Name | Definition (condensed) | Typical manifestation | Level | Method |
| --- | --- | --- | --- | --- | --- |
| QC-4.1 | Snippet-Artifact Declaration Mismatch | Snippet's calls/parameters contradict the owning artifact's own declaration (Tool Schema `properties`/`required`, or Skill/Prompt inline parameter/field names) | Tool `example` invokes a parameter absent from `properties`, or omits a `required` parameter; Skill step example uses a parameter name that differs from the Skill body | P1 | Semi-automated |
| QC-4.2 | Snippet-Body Version/API Drift | Snippet contradicts the owning artifact's own stated version/dependency/API (stale example) | Body states "use `pdfplumber.open()`", snippet still calls `pdfplumber.load()`; body declares a dependency the snippet does not import | P2 | Semi-automated |

## Appendix — Detection rules index

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
