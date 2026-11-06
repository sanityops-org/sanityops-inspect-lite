# pre-inspect-extract golden fixtures

Regression fixtures for the extraction layer. Each case is `input → expected ir-core`. Six cases, one per completeness label, plus one cross-object reference example. These pin the determinism principle (SKILL.md §1 principle 4); a change that alters an expected `ir-core` is a breaking change to the ir-core v1 contract.

## Case 1 — full original (E1, fenced JSON Schema)

Input:

````markdown
```json
{
  "name": "search_docs",
  "description": "Search the knowledge base",
  "inputSchema": {
    "type": "object",
    "properties": { "query": { "type": "string", "description": "The query" } },
    "required": ["query"]
  }
}
```
````

Expected `ir-core`:

```json
{
  "ir_version": "1",
  "objects": [
    { "type": "tool", "name": "search_docs", "completeness": "full original", "L": "OR-L1", "source": "case 1 fence", "mismatch": null }
  ],
  "not_inspectable": [],
  "references": []
}
```

## Case 2 — original · variable unexpanded (E2, rule variable not locatable)

Input (definition of `reference_prompt` is not in the input):

```python
system = f"You are a support agent. {reference_prompt} Answer the question."
```

Expected `ir-core` (the unexpanded variable list lives in the object layer prose, not in `ir-core`):

```json
{
  "ir_version": "1",
  "objects": [
    { "type": "prompt", "name": "(unnamed)", "completeness": "original · variable unexpanded", "L": "undetermined", "source": "case 2", "mismatch": null }
  ],
  "not_inspectable": [],
  "references": []
}
```

## Case 3 — assembled (E3, Python decorator)

Input:

```python
@mcp.tool()
def delete_customer(customer_id: str) -> str:
    """Delete a customer permanently.

    Args:
        customer_id: The customer to delete.
    """
    ...
```

Expected `ir-core`:

```json
{
  "ir_version": "1",
  "objects": [
    { "type": "tool", "name": "delete_customer", "completeness": "assembled", "L": "OR-L3", "source": "case 3", "mismatch": null }
  ],
  "not_inspectable": [],
  "references": []
}
```

## Case 4 — incomplete (name + description, no schema)

Input:

```json
{ "name": "update_record", "description": "Update a record" }
```

Expected `ir-core`:

```json
{
  "ir_version": "1",
  "objects": [
    { "type": "tool", "name": "update_record", "completeness": "incomplete", "L": "OR-L2", "source": "case 4", "mismatch": null }
  ],
  "not_inspectable": [],
  "references": []
}
```

## Case 5 — summary (truncated text)

Input (truncated; cannot be restored verbatim):

```text
You are an agent that ... (content elided)
```

Expected `ir-core`:

```json
{
  "ir_version": "1",
  "objects": [
    { "type": "prompt", "name": "(unnamed)", "completeness": "summary", "L": "undetermined", "source": "case 5", "mismatch": null }
  ],
  "not_inspectable": [],
  "references": []
}
```

## Case 6 — not inspectable (EX, dynamic registration)

Input:

```python
server.tool(get_data, name="get_data", output_schema=SomeModel)
```

Expected `ir-core`:

```json
{
  "ir_version": "1",
  "objects": [],
  "not_inspectable": [
    { "name": "get_data", "reason": "dynamic registration, no literal schema" }
  ],
  "references": []
}
```

## Case 7 — cross-object reference (tool call, no schema present)

Input:

```markdown
You are an agent. Call `search_tool` to look up records.
```

(No `search_tool` schema is provided.)

Expected `ir-core`:

```json
{
  "ir_version": "1",
  "objects": [
    { "type": "prompt", "name": "(unnamed)", "completeness": "full original", "L": "undetermined", "source": "case 7", "mismatch": null }
  ],
  "not_inspectable": [],
  "references": [
    { "name": "search_tool", "referenced_by": "(unnamed)", "relationship": "tool call" }
  ]
}
```
