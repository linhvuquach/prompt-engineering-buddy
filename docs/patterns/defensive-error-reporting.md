# Pattern: Defensive Error Reporting

Force the model to return structured errors instead of hallucinating data when inputs are invalid or ambiguous.

## When to Use

- Any integration with code, workflows, or APIs
- Data validation and normalization tasks
- Want **predictable failures** instead of "pretty lies"
- Downstream code can handle errors gracefully

## Key Insight

Models will try to be "helpful" by guessing or inventing data when they shouldn't. This pattern makes failures explicit and actionable.

## Prompt Template

```text
You are a strict validation engine.

Task:
Validate the input data and either:
- return a normalized result, or
- return a structured error.

Rules:
- Never guess missing values.
- If required data is missing or inconsistent, return an error object.
- Use one of these error codes: ["MISSING_FIELD", "INVALID_FORMAT", "AMBIGUOUS"].

Output (JSON ONLY):

Success:
{
  "ok": true,
  "data": { ...normalized fields... }
}

Error:
{
  "ok": false,
  "error_code": "MISSING_FIELD|INVALID_FORMAT|AMBIGUOUS",
  "message": "short explanation",
  "field": "field_name_or_null"
}

Input:
[RAW_INPUT]
```

## Working Example

### Scenario
Extract due date from invoice emails for payment system integration

### Prompt

```text
You are a strict validation engine.

Task:
Extract the due_date from the invoice email.

Rules:
- due_date must be in YYYY-MM-DD format.
- If no explicit date is found, return error code "MISSING_FIELD".
- If the date is relative (e.g., "next Friday"), return error code "AMBIGUOUS".
- Do not calculate or infer dates.

Output (JSON ONLY):

Success:
{
  "ok": true,
  "data": {
    "due_date": "YYYY-MM-DD"
  }
}

Error:
{
  "ok": false,
  "error_code": "MISSING_FIELD|AMBIGUOUS|INVALID_FORMAT",
  "message": "explanation",
  "field": "due_date"
}

Input:
[EMAIL_TEXT]
```

### Test Case 1: Ambiguous Input

**Input:**
```text
Payment is due next Friday.
```

**Expected Output:**
```json
{
  "ok": false,
  "error_code": "AMBIGUOUS",
  "message": "Due date specified as relative time, cannot map to exact date.",
  "field": "due_date"
}
```

### Test Case 2: Success

**Input:**
```text
Invoice due date: 2025-01-15.
```

**Expected Output:**
```json
{
  "ok": true,
  "data": {
    "due_date": "2025-01-15"
  }
}
```

### Test Case 3: Missing

**Input:**
```text
Thank you for your business.
```

**Expected Output:**
```json
{
  "ok": false,
  "error_code": "MISSING_FIELD",
  "message": "No due date found in the input.",
  "field": "due_date"
}
```

## Evaluation Criteria

- **Zero hallucinations:** Ambiguous/missing data → errors, not guessed values
- **Correct error codes:** Each failure type mapped to right code
- **Parseable:** 100% of outputs are valid JSON
- **Actionable messages:** Error messages help developers debug
- **Consistency:** Same bad input produces same error across runs

## Error Code Design

### Standard Error Codes

| Code | Meaning | When to Use |
|------|---------|-------------|
| `MISSING_FIELD` | Required data not found | Email has no due date |
| `INVALID_FORMAT` | Data present but wrong format | Date is "Jan 5th" not "YYYY-MM-DD" |
| `AMBIGUOUS` | Multiple valid interpretations | "next Friday" - which Friday? |
| `INCONSISTENT` | Contradictory information | Two different due dates in text |
| `OUT_OF_RANGE` | Value outside acceptable bounds | Date is in the past |

### Custom Error Codes per Domain

**E-commerce:**
- `INVALID_PRICE` - Price is negative or zero
- `UNKNOWN_SKU` - Product code not in catalog
- `INVALID_QUANTITY` - Quantity not a positive integer

**Support Tickets:**
- `UNCLEAR_INTENT` - Can't determine what user wants
- `MISSING_CONTEXT` - Not enough info to classify
- `MULTIPLE_ISSUES` - User reported 3 unrelated problems

## Advanced Techniques

### 1. Validation with Confidence

```json
{
  "ok": true,
  "data": {
    "due_date": "2025-01-15"
  },
  "confidence": "high|medium|low",
  "warnings": ["Date format was ambiguous, inferred as ISO format"]
}
```

Use `warnings` for successful extractions with potential issues.

### 2. Multiple Errors

```json
{
  "ok": false,
  "errors": [
    {
      "error_code": "MISSING_FIELD",
      "field": "email",
      "message": "Customer email not provided"
    },
    {
      "error_code": "INVALID_FORMAT",
      "field": "phone",
      "message": "Phone number must be in E.164 format"
    }
  ]
}
```

### 3. Partial Success

```json
{
  "ok": "partial",
  "data": {
    "name": "Acme Corp",
    "revenue": 12400000
  },
  "errors": [
    {
      "error_code": "MISSING_FIELD",
      "field": "year",
      "message": "Year not found in text"
    }
  ]
}
```

Allow system to proceed with partial data.

### 4. Suggested Fixes

```json
{
  "ok": false,
  "error_code": "INVALID_FORMAT",
  "message": "Date must be YYYY-MM-DD",
  "field": "due_date",
  "found_value": "Jan 15, 2025",
  "suggested_value": "2025-01-15",
  "suggestion_confidence": "high"
}
```

Help users fix errors quickly.

## Common Pitfalls

### Pitfall 1: Model Tries to Be Helpful

❌ **Input:** "Payment due next week"  
❌ **Model:** `{"ok": true, "data": {"due_date": "2025-01-23"}}`

Model calculated "next week" from today.

**Fix:**
```text
Rules:
- Do NOT calculate relative dates.
- Do NOT use today's date or external knowledge.
- If date is relative, return error code "AMBIGUOUS".
```

### Pitfall 2: Vague Error Messages

❌ `{"message": "Invalid input"}`  
✅ `{"message": "Phone number must include country code (E.164 format)"}`

**Fix:**
```text
Error messages must:
- Explain what was wrong
- State the expected format
- Be actionable for developers
```

### Pitfall 3: Inconsistent Error Codes

Same error type gets different codes:
- Run 1: `MISSING_FIELD`
- Run 2: `INVALID_FORMAT`

**Fix:**
```text
Error code definitions:
- "MISSING_FIELD": Field is completely absent from input
- "INVALID_FORMAT": Field is present but in wrong format
- "AMBIGUOUS": Field is present but has multiple valid interpretations

Use these definitions consistently.
```

### Pitfall 4: Success with Null Data

❌ `{"ok": true, "data": {"due_date": null}}`

Should be an error, not success with null.

**Fix:**
```text
Rules:
- Return ok: true ONLY if all required fields have non-null values.
- If any required field is null, return ok: false with error code "MISSING_FIELD".
```

## Integration Patterns

### Python Integration

```python
def extract_due_date(email_text: str) -> dict:
    """Extract due date with error handling"""
    response = llm.generate(prompt, input=email_text)
    data = json.loads(response)
    
    if not data['ok']:
        logger.warning(f"Extraction failed: {data['error_code']} - {data['message']}")
        raise ValidationError(data)
    
    return data['data']

# Usage
try:
    result = extract_due_date(email)
    schedule_payment(result['due_date'])
except ValidationError as e:
    if e.error_code == 'AMBIGUOUS':
        request_clarification(email)
    elif e.error_code == 'MISSING_FIELD':
        mark_for_manual_review(email)
```

### Workflow Orchestration

```python
# Pipeline with error handling
result = step1(input)

if not result['ok']:
    if result['error_code'] == 'MISSING_FIELD':
        # Try alternative extraction method
        result = step1_fallback(input)
    else:
        # Abort pipeline
        return result

# Continue to next step
next_result = step2(result['data'])
```

### User-Facing Errors

```python
error_messages = {
    'MISSING_FIELD': "We couldn't find {field} in your input. Please provide it.",
    'INVALID_FORMAT': "{field} should be in {expected_format} format.",
    'AMBIGUOUS': "The {field} you provided is unclear. Please be more specific."
}

if not result['ok']:
    user_message = error_messages[result['error_code']].format(
        field=result['field'],
        expected_format=result.get('expected_format', 'the correct')
    )
    return {"error": user_message}
```

## Testing Strategy

### Error Path Coverage

```python
def test_missing_field():
    """Test MISSING_FIELD error"""
    input = "Thank you for your business."
    result = extract(input)
    assert result['ok'] == False
    assert result['error_code'] == 'MISSING_FIELD'

def test_ambiguous_input():
    """Test AMBIGUOUS error"""
    input = "Payment due next Friday."
    result = extract(input)
    assert result['ok'] == False
    assert result['error_code'] == 'AMBIGUOUS'

def test_invalid_format():
    """Test INVALID_FORMAT error"""
    input = "Due date: Jan 5th."
    result = extract(input)
    assert result['ok'] == False
    assert result['error_code'] == 'INVALID_FORMAT'
```

### Fuzzing

Generate intentionally bad inputs:

```python
bad_inputs = [
    "",  # Empty
    "Lorem ipsum...",  # Gibberish
    "Due date: maybe tomorrow?",  # Vague
    "Due on 01/02/03",  # Ambiguous format
    "Due: 2025-13-45",  # Invalid date
]

for input in bad_inputs:
    result = extract(input)
    assert result['ok'] == False  # Must fail gracefully
    assert 'error_code' in result
```

### Consistency Testing

```python
def test_error_consistency():
    """Same bad input should produce same error"""
    input = "Payment due next Friday."
    results = [extract(input) for _ in range(10)]
    
    # All runs should return same error code
    error_codes = [r['error_code'] for r in results]
    assert len(set(error_codes)) == 1
```

## Real-World Applications

### Data ETL Pipelines
- Extract from unstructured sources
- Return errors for manual review
- Track error rates per source

### Form Validation
- LLM normalizes user input
- Returns errors for invalid entries
- Suggests corrections

### API Request Validation
- Parse natural language API requests
- Validate parameters
- Return structured errors to API clients

### Content Moderation
- Classify content
- Return error if classification confidence is low
- Flag for human review

## Cost Optimization

- Error handling is cheap (small models work well)
- Use GPT-3.5 or Claude Haiku for validation
- Cache validation prompts
- Batch validation when possible

## Related Patterns

- [Structured Output](./structured-output.md) - Base JSON structure
- [Guardrailed Factual Answering](./guardrailed-qa.md) - "Don't know" responses
- [Task Decomposition](./task-decomposition.md) - Validate at each pipeline stage
- [Core Mental Models](../core-mental-models.md) - "Defensive prompting" principle

---

**Back to:** [All Patterns](./README.md) | [Main Documentation](../README.md)

