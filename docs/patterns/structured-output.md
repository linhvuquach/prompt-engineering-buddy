# Pattern: Structured Output (Strict JSON)

Force the model to produce perfectly parseable JSON for downstream code integration.

## When to Use

- Downstream code parses LLM responses
- ETL pipelines, data enrichment, or extraction tasks
- Agents/tools calling other systems or APIs
- Any scenario where JSON.parse failure breaks your workflow

## Prompt Template

```text
You are a data extraction engine.

Task:
Extract the following fields from the text:

- [FieldName1] ([type], [rules])
- [FieldName2] ([type], [rules])
- ...

Rules:
- Output JSON only.
- Do not include any explanation.
- Use null if a value is missing or cannot be determined.
- Do not add extra keys.
- Strings must not contain line breaks.

JSON schema:
{
  "field1": <type/description>,
  "field2": <type/description>,
  "field3": <type/description>
}

Text:
[RAW_TEXT]
```

## Working Example

### Scenario
Extract financial data from news articles for database ingestion

### Prompt

```text
You are a data extraction engine.

Task:
Extract the following fields from the text:
- company_name (string)
- revenue_usd (number, in USD)
- year (integer, YYYY)

Rules:
- Output JSON only.
- Use null if missing.
- No extra text.

JSON schema:
{
  "company_name": "string or null",
  "revenue_usd": "number or null (no currency symbols)",
  "year": "integer or null"
}

Text:
"Acme Corp reported revenue of $12.4M in 2023."
```

### Expected Output

```json
{
  "company_name": "Acme Corp",
  "revenue_usd": 12400000,
  "year": 2023
}
```

## Evaluation Criteria

- **Parsing:** `JSON.parse()` succeeds on 100% of responses
- **Schema conformance:** Keys exactly match schema; no extra keys
- **Type correctness:** Numbers are numbers, strings are strings
- **Normalization:** Units converted correctly (12.4M → 12400000)
- **Accuracy:** On labeled test set, ≥95% correct extraction

## Advanced Techniques

### 1. Nested Objects

```json
{
  "company": {
    "name": "string",
    "headquarters": "string or null"
  },
  "financials": {
    "revenue_usd": "number or null",
    "year": "integer"
  }
}
```

### 2. Arrays

```json
{
  "products": [
    {
      "name": "string",
      "price": "number"
    }
  ]
}
```

**Prompt addition:**
```text
- If multiple items are present, return an array.
- If no items are found, return an empty array [].
```

### 3. Enums (Controlled Vocabularies)

```json
{
  "priority": "low|medium|high",
  "status": "open|in_progress|closed"
}
```

**Prompt addition:**
```text
- priority must be exactly one of: "low", "medium", "high"
- If uncertain, use "medium"
```

## Common Pitfalls

### Pitfall 1: Extra Commentary

❌ **Model output:**
```text
Here is the extracted data:
{"company_name": "Acme Corp", "revenue_usd": 12400000, "year": 2023}
```

**Fix:** Add to rules:
```text
- Output JSON only, starting with { and ending with }
- No introductory text or explanations
```

### Pitfall 2: Extra Keys

❌ **Model output:**
```json
{
  "company_name": "Acme Corp",
  "revenue_usd": 12400000,
  "year": 2023,
  "confidence": "high"
}
```

**Fix:** Add to rules:
```text
- Output ONLY the keys defined in the schema
- Do not add extra fields
```

### Pitfall 3: Malformed Numbers

❌ **Model output:**
```json
{
  "revenue_usd": "$12.4M"
}
```

**Fix:** Be explicit about normalization:
```text
- revenue_usd must be a number (e.g., 12400000)
- Remove currency symbols, convert M/K suffixes to actual numbers
```

### Pitfall 4: Line Breaks in Strings

❌ **Model output:**
```json
{
  "summary": "First line
  Second line"
}
```

**Fix:** Add to rules:
```text
- Strings must not contain line breaks
- Use spaces instead of newlines
```

## Testing Strategy

### Unit Tests

```python
import json

def test_json_parsing(response: str):
    """All responses must parse"""
    data = json.loads(response)  # Should not raise
    assert isinstance(data, dict)

def test_schema_keys(response: str, expected_keys: set):
    """Only expected keys present"""
    data = json.loads(response)
    assert set(data.keys()) == expected_keys

def test_null_handling(response: str):
    """Missing values are null, not empty strings"""
    data = json.loads(response)
    for value in data.values():
        assert value is not None or value is None  # Not ""
```

### Integration Tests

Run prompt on 50-100 labeled examples:
- Measure extraction accuracy per field
- Check for format violations
- Validate business logic (e.g., prices are positive)

## Cost Optimization

- Use **smaller models** for simple extraction (GPT-3.5, Claude Haiku)
- Cache prompts with static schema definitions
- Batch multiple extractions when possible
- Use structured output native APIs when available (e.g., OpenAI structured outputs)

## Related Patterns

- [Role + Task + Rules + Format](./role-task-rules-format.md) - Base template
- [Defensive Error Reporting](./defensive-error-reporting.md) - Add error handling to structured output
- [Task Decomposition](./task-decomposition.md) - Break complex extractions into steps

---

**Back to:** [All Patterns](./README.md) | [Main Documentation](../README.md)

