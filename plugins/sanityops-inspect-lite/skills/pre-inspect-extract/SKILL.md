---
name: pre-inspect-extract
version: 1.1.0
owner: Sanity AI Labs
description: Trigger words: sanityops-extract / artifact extraction / 工件提取 / 只提取. Pre-inspection extraction layer that runs before sanityops-inspect-lite. Finds logic artifacts (System Prompt / Skill / Tool Schema) in any user-submitted document, segments them one by one, strips the code/markup shell that carries them, reconstructs the schema field-by-field from source when necessary, labels each artifact's completeness level with verbatim evidence, and determines each artifact's tier (AC-L / OR-L). Extract-only, never judges defects. Outputs a two-layer IR (objects + references) for the inspection stage. Use when you need to first extract logic artifacts from a document before inspecting them.
license: CC BY-SA 4.0
changelog: https://github.com/sanityops-org/sanityops-inspect-lite/releases
---

# Pre-Inspection Extraction Layer (pre-inspect-extract)

## 0. Glossary (read this first)

The words in this file mean exactly the following. Nothing else.

| Term | Meaning |
|---|---|
| **artifact** | One of the three things an Agent runtime can load: System Prompt, Skill, Tool Schema |
| **carrier** | Code or markup that wraps an artifact, e.g. `f"""..."""`, backtick template strings, ``` fences |
| **scaffolding** | Non-artifact content in a document: Metadata, Evaluation Dimensions, Notes, etc. |
| **verbatim copy** | Copy the original text word-for-word, dropping only the outer code shell |
| **assembly** | There is no ready-made schema; copy fields out of the source one by one and assemble a JSON Schema |
| **full original** | Obtained by verbatim copy, complete, directly inspectable |
| **original · variable unexpanded** | Obtained by verbatim copy, but the prompt has unexpanded variables that expand into rules |
| **assembled** | A schema assembled from code, not the verbatim original |
| **incomplete** | The artifact itself is incomplete, e.g. only name/description, no schema |
| **summary** | Summarized / transcribed / truncated; cannot be matched to the original in full |
| **not inspectable** | Cannot be extracted; skip and record the reason |
| **completeness level** | The umbrella term for the six labels above |
| **skip a check** | Because the data is unreliable, a specific check is not judged this time (this is not a FAIL) |
| **AC-L** | Agent Complexity Level — how complex the agent is (L1 single-turn / L2 multi-turn / L3 complex; L4 multi-agent reserved). Feeds the Prompt-domain severity (P). |
| **OR-L** | Operation Risk Level — how risky the operations are (L1 read-only / L2 reversible write / L3 irreversible). Feeds the Skill/Tool-domain severity (P). |
| **tier determination (L)** | The per-artifact classification into AC-L / OR-L, done once here and handed down. It feeds severity (P) only, never "whether to check". |
| **IR** | Intermediate Representation — this layer's structured output: two layers (object + reference) plus L tagging, materialized to a file and also kept in-context. |
| **object layer** | One of the two IR layers — artifacts actually extracted with a full signature (how many P / S / T). |
| **reference layer** | One of the two IR layers — things mentioned / depended on but not formed as an extracted object. |

## 1. What this is

A pure-prompt, zero-dependency **pre-inspection extraction** skill that runs before `sanityops-inspect-lite`. It turns any user-submitted document into "inspectable artifacts + verbatim evidence".

It only extracts, only classifies, only labels completeness. **It does not judge defects** — PASS/FAIL is the job of the seven-stage inspection that follows.

Three inviolable principles:

1. **The source document is read-only.** Never rewrite the input.
2. **Rather not inspect than force a verdict.** If something is unrecognizable, label it "not inspectable" and record the reason; never fabricate fields, never guess missing content, never mistake a carrier for an artifact.
3. **Reliability comes from verbatim copy and self-evidence against the original**, not from a parser (this skill has no scripts).

## 2. Input and output

**Input**: one user-submitted document; document size is bounded by "must be fully readable within the host context" — if over the limit, extract only the fully-readable part and label it "summary".

**Output**: a two-layer Intermediate Representation (IR), sized in proportion to the number of input artifacts (one block per artifact), with no extra inflation. The IR is **materialized to a file** (for audit and reuse by CLI / platform) and also **kept in-context** (for the chained inspection to read immediately).

- **Extraction summary** (top): agent membership declaration (single agent / pending confirmation), total artifact count, count by type, not-inspectable count and per-item reasons, completeness-level distribution, and L distribution.
- **Object layer**: one block per extracted artifact, each carrying type, name, completeness level, **tier (L)**, source location, extraction method, and verbatim evidence.
- **Reference layer**: the things mentioned / depended on but not formed as extracted objects.
- **`ir-core` block**: the machine-readable core of the object + reference layers (type / name / L / completeness), carried in a fixed JSON block so CLI / platform can parse it directly.

## 3. Things in a document fall into four categories

| Category | What it is | What to do |
|---|---|---|
| **artifact** | Itself a Prompt / Skill / Tool Schema | Send to inspection |
| **carrier** | Code or markup that wraps an artifact | Strip the shell, copy the content verbatim |
| **scaffolding** | Metadata, Evaluation Dimensions, Scoring Criteria, Test Suggestions, Notes, etc. | Discard |
| **reference file** | Pure code or pure data, contains no extractable artifact | Skip |

> A carrier is not an artifact. `def f(...): return f"""..."""` is a carrier; the string inside is the artifact.

## 4. Six-step flow

```
Segment → classify → extract → determine completeness → determine level (L) → record evidence
```

### 4.1 Segmentation

Cut into candidate objects, discard scaffolding.

- Scaffolding headings: `## Metadata`, `## Evaluation Dimensions`, `## Scoring Criteria`, `## Test Suggestions`, `## Notes`, `## Source Profile`.
- Each `### N. xxx`, each top-level code fence block, each text passage with a structural signature, is one candidate object.

**Must report the artifact count**: after segmentation, state explicitly "this file has N candidate objects" and name each one. A file often contains multiple artifacts (two `Tool(...)` in one `.py`, two prompt constants in one `.ts`, are two artifacts). **A missed segment is a missed inspection.** Do not accept "several" or "multiple".

**Detect cross-object references (reference layer)** while segmenting: record every named thing a prompt / skill mentions as a dependency but that is not itself present as a formed object — e.g. a prompt says `call search_tool` but the document provides no `search_tool` schema. These go to the **reference layer** (output in §11), so the downstream cross-object checks can catch "referenced but missing". Only a **concrete, named call / dependency** counts (a verbatim tool/skill name); generic prose like "query the database" is not a reference-layer entry.

### 4.2 Classification

For each candidate object, determine which tier it falls into, and state the basis (the structural feature you saw).

### 4.3 Extraction

Two methods: **verbatim copy** and **assembly**, see §6.

### 4.4 Determine completeness

Decide whether it can be sent to inspection and which label to apply, see §7.

### 4.5 Determine level (L)

For each extracted artifact, determine its tier (AC-L for Prompt, OR-L for Skill/Tool), see §8. Only artifacts with a usable extraction get an L; "not inspectable" objects get none.

### 4.6 Record evidence

See §9.

## 5. Classification rules

### Tier 1 — directly readable

The artifact exists in a loadable form:

- A complete artifact inside ` ```markdown ` / ` ```text ` / ` ```json ` fences.
- No fence but structurally complete: `name` + `description` front matter; or `type: object` + `properties` + `required`.
- Python `Tool(name=..., description=..., inputSchema={...})` — `inputSchema` itself is already a JSON Schema dict.
- JS/TS object literal `{ name: "...", description: "...", inputSchema: {...} }` or `parameters: {...}`.
- A pure JSON Schema.

→ **verbatim copy**.

### Tier 2 — wrapped in code

The artifact is complete text, but carried inside code. **Do not enumerate syntax; only judge whether the text is carried by code**:

- Python: `f"""..."""`, `"""..."""`, plain `"..."`, **docstring** (e.g. a `dspy.Signature` class docstring), **a string returned by a function**.
- TypeScript / JavaScript: backtick template strings, plain quoted strings, a string returned by an arrow function, a string in an object property.
- Jinja templates (`{{ }}` is runtime interpolation; the template text itself is the artifact).

→ **verbatim copy**: take the string content, drop the assignment statement, function signature, `return`, backticks; unescape; keep variables as-is.

> Do not enumerate syntax case by case. `f"""`, backticks, `"..."`, docstring, `return` differ only in which shell to strip, not in whether this is Tier 2. Enumerating syntax once caused docstring and plain-string cases to be missed.

**Examine each variable**:

| Variable content | How to handle |
|---|---|
| Runtime data (`{local_tz}`, `{date.today()}`, `{context}`, `{question}`) | Keep as-is, label "full original" |
| **A variable that expands into a rule sentence** (`{reference_prompt}`, `{tone_prompt}`) | Label "original · variable unexpanded", state which variable and how many instructions it expands into |

The latter must not be ignored: rules written inside a variable are absent from the literal string. If labeled "full original", checks like "reference specification not declared" would wrongly fail even though the source did declare it.

### Tier 3 — only code, no ready-made schema

Needs **assembly** to become a loadable artifact:

| Source form | Characteristic |
|---|---|
| Zod `z.object({...})` | Constraints written in chained calls; `.refine()` cannot be converted to JSON Schema |
| Python decorator `@mcp.tool()` + type annotations + docstring | Types from annotations, defaults from the signature, descriptions from the `Args:` section |
| Go builder `mcp.NewTool(...)` + `WithString/Required/DefaultString/Description` | Parameters are all literals |

→ **assembly**, see §6.2.

### Tier X — unrecognizable

- No structural signature, no fence, cannot tell whether it contains an artifact.
- Dynamic registration, no literal schema (`server.tool(func, name=..., output_schema=...)`).
- Only a registration list (`{"fn": "...", "tags": [...]}`, `manifest.json` entries), no `inputSchema` / `parameters`.
- Go `type Xxx struct` + tags (types are inferred, constraints rely on validate tags, information is incomplete).

→ label "not inspectable", record the reason.

**Exception**: if `name` and `description` are present and only `inputSchema` is missing, this is not "unrecognizable" — it is an "incomplete artifact". Label "incomplete", extract the available fields, and state "schema not provided with the document". Here "missing inputSchema" is recorded only as a document-level fact, not judged as a product defect.

## 6. Extraction rules

### 6.1 Verbatim copy

Applies to Tier 1 and Tier 2.

1. Copy word-for-word; do not rewrite, polish, or complete.
2. Drop only the shell (assignment statement, function definition line, `return`, fence markers, `export const X =`, backticks); the content inside the shell is untouched.
3. Unescape (`\n` → newline); variables (`{var}`, `{{ var }}`) are kept as-is, not substituted.
4. The result must match the original in full. If it does not → label "summary".
5. Check every placeholder per the "examine each variable" rule in §5 Tier 2.

### 6.2 Assembly

Applies to Tier 3. **Reorganize only, do not translate.**

1. Copy, field by field, everything already literal in the source: `name`, `description` / `.describe("...")`, each parameter's key and description, `enum` values, the `required` list, `default` values.
2. Convert constraints by rule:

| Source form | JSON Schema |
|---|---|
| Zod `.max(n)` / `.min(n)` on a string | `maxLength` / `minLength` |
| Zod `.max(n)` / `.min(n)` on a number | `maximum` / `minimum` |
| Zod `.max(n)` / `.min(n)` on an array | `maxItems` / `minItems` |
| Zod `.int()` | `type: integer` |
| Zod `.enum([...])` | `enum` |
| Zod `.optional()` | not in `required` |
| Zod `.default(v)` | `default` |
| Zod `.regex(re)` | `pattern` |
| Zod `.email()` / `.url()` / `.uuid()` | `format: email` / `uri` / `uuid` |
| Python `str/int/float/bool` annotation | `string/integer/number/boolean` |
| Python `List[X]` annotation | `type: array` + `items` |
| Python `Optional[X]` / `None` default | not in `required` |
| Python `Literal[a, b]` | `enum` |
| Python no default / has default | in `required` / record `default` |
| Python docstring `Args:` section | the corresponding parameter's `description` |
| Go `WithString/WithNumber/WithBoolean` | `type` |
| Go `WithArray(...)` / `WithEnum(...)` / `WithObject(...)` | `type: array` / `enum` / `type: object` |
| Go `Required()` | in `required` |
| Go `DefaultString/DefaultNumber` | `default` |
| Go `Description(...)` | `description` |

3. Wrap into `{ "type": "object", "properties": {...}, "required": [...] }`.
4. **Add no field-level constraint that is absent from the source.** If the source does not write a `type` for a property, do not add a `type` keyword for that property. (The root `"type": "object"` in rule 3 is the assembly wrapper, not a source field.)
5. `.refine()` / `.superRefine()` / custom validation cannot be converted to JSON Schema; **do not fabricate an equivalent constraint** — write "implemented in source via refine, not entered into the schema".
6. Python MCP-injected parameters such as `ctx: Context` are not user parameters and **must be removed**.
7. Attach the source fragment for each field.

### 6.3 The description field must not be lost

`description` / `.describe()` / docstring is the core basis of Tool Schema inspection, and is the easiest thing to lose during assembly.

- Source has a description, result does not → label the whole thing "assembled" and write "description field at risk of loss".
- Source has no description → the result must not have one either; **do not write one**.

## 7. Completeness levels

Six labels, applied to the whole artifact, not subdivided by field.

| Label | Meaning | Which checks can run |
|---|---|---|
| **full original** | Verbatim copy, complete | All |
| **original · variable unexpanded** | Verbatim copy, but has unexpanded variables that expand into rules | All except checks that depend on those variables |
| **assembled** | Schema assembled from code | Structure and contradiction checks all run; "a field is missing some constraint" is judged per field (see below) |
| **incomplete** | Artifact incomplete, missing base fields | Only checks on the fields actually provided |
| **summary** | Summarized / transcribed / does not match the original | Only contradiction checks, not "what is missing" |
| **not inspectable** | Cannot be extracted | No checks; only record the reason |

**How "assembled" judges a missing constraint**: look at the evidence.

| Evidence situation | Conclusion |
|---|---|
| The source fragment covers the field's complete writing (whole chained call / full signature / full builder call) | Judgeable. The source really did not write it → truly missing |
| The source fragment covers only part, or the boundary is unclear | Skip that check, mark "uncertain" |

Whether a check runs depends on whether that field was copied fully, not on whether it is an assembled artifact.

**`.refine()` is looked at separately**: `.refine()` generally does not enter the runtime-generated JSON Schema. So when the description promises a constraint (e.g. "at most 50 words") but the schema has no corresponding keyword, this is a **candidate QD-T-4.5 check to hand off** (recorded as evidence for the inspection stage — this layer never emits PASS/FAIL) and must not be erased as "cannot be assembled". Wording: "description declares X, schema has no corresponding constraint; in source X is implemented via refine, which usually does not enter the JSON Schema."

## 8. Tier determination (L)

This layer determines **L** — the complexity / operation-risk tier of each artifact — once, and hands it down. L is a classification, not a defect grade. It feeds only the severity (P) of the downstream checks; it does **not** decide whether a check runs (that is the downstream structural-fact gate's job, not L's).

### 8.1 Two dimensions, kept separate

| Dimension | Judges | Levels | Applies to |
|---|---|---|---|
| **AC-L** | how complex the agent is | L1 single-turn / L2 multi-turn / L3 complex (L4 multi-agent reserved) | System Prompt |
| **OR-L** | how risky the operations are | L1 read-only / L2 reversible write / L3 irreversible | Skill, Tool Schema |

The two dimensions are **kept separate at the engine level** — they are never merged into one number, because merging would break severity lookup (e.g. "AC-L1 + OR-L3" cannot be expressed as a single level).

### 8.2 AC-L determination (Prompt)

Read the prompt content and judge its complexity:

- **AC-L1 (single-turn)**: one question, one answer; no tools, no multi-turn state.
- **AC-L2 (multi-turn)**: a multi-turn dialogue; each turn is self-contained.
- **AC-L3 (complex)**: planning / reflection / tool orchestration across steps.
- **L4 (multi-agent)** is reserved; do not emit L4 until the multi-agent scale is finalized.

### 8.3 OR-L determination (Skill / Tool)

Read the skill/tool content and judge its operation risk by the most dangerous operation it can perform:

- **OR-L1 (read-only)**: query / search / read; no write.
- **OR-L2 (reversible write)**: create / update / send that can be corrected or undone.
- **OR-L3 (irreversible)**: delete / transfer / irreversible external notification.

### 8.4 Product layer: one unified label

For the user-facing output, collapse the two dimensions into **one unified label per artifact** (the merged view happens in how it is shown, not in how it is computed):

- Prompt → show its AC-L.
- Skill / Tool → show its OR-L.
- (Permission, a cross-cutting concern, later takes `max` across the relevant artifacts — handled downstream, not here.)

### 8.5 When uncertain

- If L cannot be judged from the available content (e.g. "summary" fidelity), emit `L = undetermined` and let the downstream structural-fact gate decide "whether to check"; severity then falls back to the checklist's default.
- A wrong L is safe by design: because L no longer decides "whether to check", the worst case is the severity (P) being off by one tier — it can never drop a check.

## 9. Evidence

Do not use line numbers (models cite them unreliably) or checksums (no scripts). Use **excerpt self-evidence**.

Each output carries:

- **Location**: source file + section heading (e.g. `## Original Artifact Content` / `### 2. Input schema`).
- **Method**: verbatim copy / assembly / summary.
- **Verbatim excerpt**: verbatim-copy cases give the head and tail fragments; assembly cases give the source fragment per field (`name` → source, `description` → source, each parameter → source).

Self-evidence standard: a person who does not know the context can find every point in the original using only the evidence. If not, drop the completeness level by one.

## 10. Agent membership within a document

**By default, one document = one agent's artifact set.**

- One document may contain multiple artifacts of the same agent (one prompt + several tools).
- If multiple agent forms are detected (e.g. a suite of 6 roles), **do not auto-split and do not auto-bounce**: label "agent membership pending confirmation", state the detection basis, and output two options for the user to confirm — (a) confirm they are one agent → continue; (b) declare them an unrelated artifact collection → hand to inspect in collection mode for per-artifact single checks. pre-inspect does not decide for the user.

### How to handle the remaining cases

| Case | Handling |
|---|---|
| Fence not closed | Do not guess the ending; label "summary" |
| Variable is runtime data (`{local_tz}`) | Keep the variable, label "full original" |
| Variable expands into rules (`{reference_prompt}`) | Label "original · variable unexpanded", list the variable |
| Multiple fragments assembled by code into a final prompt | Label "incomplete", write "assembly logic not provided" |
| A field does not match the source during assembly | Label the whole thing "assembled" and note it |
| Description field suspected lost | The related checks do not run, evidence noted |
| Suspected multiple agents | Label "agent membership pending confirmation", output confirmation options |
| Summarized / transcribed / cannot cut cleanly | Label "summary" or "not inspectable" |

## 11. Output format

One Markdown inventory, structured as a two-layer IR:

1. **Extraction summary** (top): agent membership declaration (single agent / pending confirmation), total artifact count, count by type, not-inspectable count and per-item reasons, completeness-level distribution, and L distribution.
2. **Object layer** — one block per extracted artifact: artifact name, type, completeness level, **tier (L)**, source location, extraction method, verbatim evidence. "original · variable unexpanded" must list the unexpanded variables and a content summary.
3. **Reference layer** — every cross-object reference detected in §4.1: `referenced name | referenced by | relationship`; generic prose does not enter here.
4. **Not-inspectable list** — listed separately, each with a reason, producing no defect conclusion.
5. **`ir-core` JSON block** — the machine-readable core of layers 2 and 3 (see below), for CLI / platform to parse.

### ir-core block (machine-readable core)

Materialize the IR to a file and, inside it, embed one `<!-- ir-core -->` JSON block carrying only the machine-readable fields (type / name / L / completeness), so a program engine can parse the IR without reading the human evidence:

<!-- ir-core -->
```json
{
  "objects": [
    { "type": "prompt", "name": "main_prompt", "completeness": "full original", "L": "AC-L2" },
    { "type": "tool", "name": "delete_customer", "completeness": "full original", "L": "OR-L3" }
  ],
  "references": [
    { "name": "search_tool", "referenced_by": "main_prompt", "relationship": "tool call" }
  ]
}
```
<!-- /ir-core -->

**Materialization**: write the IR as a Markdown file (default name `{input_basename}.ir.md`, next to the input or in the user-specified output directory) **and** keep a copy in-context for the chained inspection. The file is the durable audit checkpoint; the in-context copy is the working handoff.

## 12. What this does NOT do

- Do not rewrite, repair, or normalize artifact text.
- Do not judge PASS / FAIL / severity (P0/P1/P2). Determining L (AC-L / OR-L) is a tier classification, not a defect grade.
- Do not split or mix across agents.
- Do not fabricate fields, invent constraints, or write missing descriptions.
- Do not treat a carrier as an artifact, or scaffolding as an artifact.
- Do not write injected parameters like `ctx: Context` as tool parameters.

## 13. Handoff to sanityops-inspect-lite

```
Original document → [pre-inspection extraction layer] → two-layer IR (objects + references, each object tagged with L) → [seven-stage inspection]
```

The pre-inspection layer does not change the seven-stage logic; it only changes what is fed into it — the two-layer IR (with L tagging) replaces the flat artifact list. The IR is handed down both in-context and as a materialized file. "Not inspectable" objects are listed in the final report as input notes, producing no defect conclusion.
