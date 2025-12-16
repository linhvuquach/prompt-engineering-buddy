# Pattern: Role + Task + Rules + Format

The universal template for production prompts. Use this as your baseline for any new prompt.

## When to Use

- **Any production prompt** (default starting point)
- Need predictable behavior and easy debugging
- Model used by multiple features or teams
- Want prompt to be maintainable by others

## Prompt Template

```text
You are a [ROLE] with expertise in [DOMAIN].

Your task:
[1–2 sentences, imperative verb: classify / extract / generate / rewrite / compare]

Input:
[Describe exact input the model receives]

Rules:
- [Hard constraint 1]
- [Hard constraint 2]
- [Failure behavior: when to say "unknown" or return null]
- Do not [forbidden behavior].

Output:
- Format: [JSON | markdown | plain text]
- No extra commentary unless specified.

Now process this input:
[INPUT_BLOCK]
```

## Working Example

### Scenario
Support ticket triage system for B2B SaaS

### Prompt

```text
You are a senior support triage assistant for a B2B SaaS.

Your task:
Classify the ticket's intent and extract key fields.

Input:
A single customer ticket message as free text.

Rules:
- Use only information explicitly in the ticket.
- If a field is missing, set it to null.
- If multiple intents are present, pick the primary one.
- Do not invent product names or features.

Output:
- Format: JSON only, no extra text.
- Schema:
  {
    "intent": "bug|feature_request|billing|question|other",
    "summary": "short string (<=20 words)",
    "priority": "low|medium|high",
    "product_area": "string or null",
    "needs_human": "boolean"
  }

Now process this ticket:
"Our invoices page times out every time I try to download the PDF. We're closing our books today, please help ASAP."
```

### Expected Output

```json
{
  "intent": "bug",
  "summary": "Invoice PDF download timing out",
  "priority": "high",
  "product_area": "invoices",
  "needs_human": true
}
```

## Evaluation Criteria

Test your prompt against these metrics:

- **Format validity:** JSON parses 100% of the time
- **Accuracy:** Intent matches human label on eval set (50+ tickets) ≥ 90%
- **Business logic:** Priority not "low" on any clearly urgent sample
- **Constraint adherence:** No hallucinated `product_area` values
- **Consistency:** Same input produces same output across 5 runs

## Variations

### Minimal Version (for simple tasks)

```text
You are a [ROLE].

Task: [ONE SENTENCE]

Rules:
- [CRITICAL CONSTRAINT]

Output: [FORMAT]

Input: [INPUT_BLOCK]
```

### Extended Version (for complex tasks)

Add these sections as needed:
- **Context:** Background information the model should know
- **Examples:** 1-2 input/output pairs demonstrating edge cases
- **Constraints:** Additional formatting or content rules

## Common Mistakes

❌ **Vague role:** "You are a helpful assistant"  
✅ **Specific role:** "You are a senior data analyst specializing in e-commerce metrics"

❌ **Unclear task:** "Help me with this text"  
✅ **Clear task:** "Extract product names and prices from this receipt"

❌ **No failure path:** (model guesses when uncertain)  
✅ **Explicit failure:** "Return null if the information is not explicitly stated"

❌ **Format implied:** (hoping for JSON)  
✅ **Format demanded:** "Output: JSON only, no other text"

## Related Patterns

- [Structured Output](./structured-output.md) - Extends this with strict JSON schemas
- [Defensive Error Reporting](./defensive-error-reporting.md) - Adds error codes and validation
- [Few-Shot Style](./few-shot-style.md) - Adds examples for tone/style control

---

**Back to:** [All Patterns](./README.md) | [Main Documentation](../README.md)

