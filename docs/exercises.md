# Hands-On Exercises

Practice converting flaky prompts into production-grade implementations using the patterns from this playbook.

## How to Use These Exercises

1. **Read the bad prompt** and understand why it's problematic
2. **Rewrite using a pattern** from the [Patterns](./patterns/) section
3. **Test your prompt** against the provided inputs
4. **Check against evaluation criteria** to see if your solution is production-ready

For each exercise, try to solve it yourself before looking at the expected output.

---

## Exercise 1: Support Ticket Classification

### Scenario
You have a flaky support triage prompt that needs to become production-grade.

### Current (Bad) Prompt

```text
Can you summarize this support ticket and tell me what to do?
```

**Problems:**
- No role definition
- Task is vague ("summarize and tell me")
- No output format specified
- No failure behavior defined
- Conversational tone ("Can you")

### Your Task

Rewrite this using the **[Role + Task + Rules + Format](./patterns/role-task-rules-format.md)** pattern:

1. Define a specific role
2. Make the task explicit and actionable
3. Add JSON output schema
4. Define failure behavior for missing information
5. Add constraints to prevent hallucination

### Test Input

```text
"The billing page charges us twice when we change our card. It's been happening for 3 months. Please refund and fix this."
```

### Expected Output Shape

```json
{
  "summary": "Customer is charged twice when changing card on billing page.",
  "intent": "bug",
  "recommended_action": "Escalate to billing engineering and issue refund for 3 months of duplicate charges.",
  "priority": "high",
  "needs_refund": true
}
```

### Evaluation Criteria

✅ Your prompt should produce output that:
- Parses as valid JSON 100% of the time
- Correctly identifies this as a "bug" with "high" priority
- Captures the key details (billing, double charge, 3 months)
- Sets `needs_refund: true`
- Has no extra keys beyond the schema
- No narrative text outside JSON

### Hints

- Start with "You are a [specific role]..."
- List each field with type and rules
- Add: "Use only information explicitly in the ticket"
- Add: "If a field is missing, set it to null"
- End with: "Output JSON only, no extra text"

---

## Exercise 2: Release Notes Style Alignment

### Scenario
Convert technical changelog entries into user-friendly release notes with consistent style.

### Current (Bad) Approach

Just asking "Make this user-friendly" leads to inconsistent tone and structure.

### Your Task

Create a **[Few-Shot Style](./patterns/few-shot-style.md)** prompt that:

1. Defines the target style with rules
2. Includes 2-3 examples of input → output transformations
3. Handles technical jargon appropriately
4. Maintains consistent structure

### Target Style Characteristics

- Non-technical wording
- Bulleted list format
- Each item starts with a verb (Fixed, Added, Improved)
- User-facing benefits, not implementation details
- No stack traces, error codes, or internal jargon

### Test Input

```text
- Fixed NPE in invoice generator when tax_id is null
- Added CSV export for invoices
- Improved query performance on /invoices endpoint
```

### Expected Output Example

```text
- Fixed an issue that could cause invoice creation to fail for some customers.
- Added CSV export so you can download your invoices in bulk.
- Improved invoice loading speed for large accounts.
```

### Evaluation Criteria

✅ Your prompt should produce output that:
- Removes technical terms (NPE, endpoint, null)
- Converts each item to user-facing benefit
- Maintains bulleted list structure
- Starts each item with active verb
- Stays consistent across multiple different inputs
- Word count similar to examples (±20%)

### Hints

- Provide 2-3 complete examples of internal → external transformation
- Add explicit rules: "No technical jargon", "Focus on user benefit"
- Show what to avoid in examples (don't just show good examples)
- Test on diverse technical inputs to ensure consistency

---

## Exercise 3: Context-Only Q&A

### Scenario
Build a docs Q&A bot that never hallucinates by only answering from provided context.

### Context (Product Docs)

```text
- The Free plan includes up to 3 projects.
- The Pro plan includes unlimited projects and priority support.
- Custom integrations are only available on Enterprise.
```

### Your Task

Create a **[Guardrailed Q&A](./patterns/guardrailed-qa.md)** prompt that:

1. Forces answers to come ONLY from the context
2. Returns "I don't know" when info isn't available
3. Never uses the model's background knowledge
4. Quotes from context when possible

### Test Questions

**Question 1:**  
"Does the Pro plan include custom integrations?"

**Question 2:**  
"How many projects can I create on the Free plan?"

**Question 3:**  
"What is the price of the Pro plan?"

### Expected Outputs

**Q1 Expected:**
```text
I don't know based on the provided context.
```
Or:
```text
The context states that custom integrations are only available on Enterprise. Pro plan features listed are unlimited projects and priority support.
```

**Q2 Expected:**
```text
The Free plan includes up to 3 projects.
```

**Q3 Expected:**
```text
I don't know based on the provided context.
```

### Evaluation Criteria

✅ Your prompt should:
- Return "I don't know" for Q1 (Pro plan custom integrations not explicitly stated)
- Return "I don't know" for Q3 (price not in context)
- Answer Q2 correctly and quote from context
- **Never claim Pro includes custom integrations**
- Work consistently across 10+ similar questions
- Not use the model's knowledge about what "usually" comes with plans

### Common Pitfalls to Avoid

❌ Model infers: "Pro probably has integrations since Enterprise does"  
❌ Model uses training data: "Pro plans typically cost $99"  
❌ Model adds helpful context: "Custom integrations let you..."  

### Hints

- Start with: "You are a careful AI assistant that ONLY uses the provided context"
- Add: "Do not guess or add outside knowledge"
- Add: "Do not infer beyond what is explicitly stated"
- Add exact phrase to use for unknowns
- Test adversarial questions designed to trick the model

---

## Exercise 4: Defensive Date Extraction

### Scenario
Extract due dates from invoice emails, but handle ambiguous and missing dates gracefully.

### Current (Bad) Prompt

```text
Extract the due_date from the invoice email and return it as YYYY-MM-DD.
```

**Problem:** Model will guess dates from relative time expressions like "next Friday."

### Your Task

Convert this into a **[Defensive Error Reporting](./patterns/defensive-error-reporting.md)** prompt that:

1. Returns structured success/error objects
2. Uses error codes: `MISSING_FIELD`, `AMBIGUOUS`, `INVALID_FORMAT`
3. Never guesses or calculates dates
4. Returns normalized date format on success

### Test Cases

**Case 1: Ambiguous**
```text
Input: "Payment is due next Friday."
```

**Case 2: Success**
```text
Input: "Invoice due date: 2025-01-15."
```

**Case 3: Missing**
```text
Input: "Thank you for your business."
```

**Case 4: Invalid Format**
```text
Input: "Due on Jan 5th, 2025."
```

### Expected Outputs

**Case 1:**
```json
{
  "ok": false,
  "error_code": "AMBIGUOUS",
  "message": "Due date specified as relative time, cannot map to exact date.",
  "field": "due_date"
}
```

**Case 2:**
```json
{
  "ok": true,
  "data": {
    "due_date": "2025-01-15"
  }
}
```

**Case 3:**
```json
{
  "ok": false,
  "error_code": "MISSING_FIELD",
  "message": "No due date found in the input.",
  "field": "due_date"
}
```

**Case 4:**
```json
{
  "ok": false,
  "error_code": "INVALID_FORMAT",
  "message": "Date found but not in YYYY-MM-DD format.",
  "field": "due_date"
}
```

### Evaluation Criteria

✅ Your prompt should:
- Never guess specific dates from "next Friday"
- Always return valid JSON
- Use correct error codes for each failure type
- Return exact YYYY-MM-DD format on success
- Handle all 4 test cases correctly
- Be consistent across multiple runs

### Hints

- Define success and error JSON schemas explicitly
- Add: "Do not calculate or infer dates"
- Add: "If date is relative, return error_code AMBIGUOUS"
- Add: "If no date found, return error_code MISSING_FIELD"
- List exact error codes to use

---

## Advanced Exercises

### Exercise 5: Multi-Step Content Generation

Apply **[Task Decomposition](./patterns/task-decomposition.md)** to break a blog post generation task into:

1. Research phase (extract key facts from sources)
2. Outline phase (structure from facts)
3. Draft phase (write sections)
4. Edit phase (refine tone)

Define the prompt for each stage and the JSON interface between stages.

### Exercise 6: Reasoning Trace for Pricing

Use **[Chain-of-Thought](./patterns/chain-of-thought.md)** to build a pricing recommendation system:

- Input: Product details, market data, costs
- Process: Reason through cost factors, comparables, value proposition
- Output: Recommended price with reasoning trace

### Exercise 7: Hybrid Pattern

Combine **[Structured Output](./patterns/structured-output.md)** + **[Defensive Error Reporting](./patterns/defensive-error-reporting.md)**:

Extract company information from text:
- Success: Return structured JSON with company name, revenue, employees
- Errors: Return error codes for missing/invalid data
- Validation: Check revenue is positive, employees is integer

---

## Evaluation Tips

### Testing Your Prompts

1. **Format Tests:** Does output always parse?
2. **Accuracy Tests:** Correct on 20-50 labeled examples?
3. **Edge Case Tests:** Handles missing/ambiguous/invalid inputs?
4. **Consistency Tests:** Same input → same output across 5 runs?
5. **Anti-Goal Tests:** Doesn't do forbidden things (hallucinate, guess, etc.)?

### Iteration Process

1. Write initial prompt using pattern template
2. Test on provided examples
3. Identify failures (format, accuracy, edge cases)
4. Add specific rules to address each failure
5. Re-test until all criteria pass

### Common Mistakes

- **Too vague:** "Be professional" instead of showing examples
- **Missing failure paths:** No rules for missing/ambiguous data
- **No schema:** Hoping for JSON without defining exact structure
- **Undertesting:** Only testing happy path, not edge cases

---

## Next Steps

After completing these exercises:

1. **Review:** [Debugging & Optimization](./debugging.md) for fixing common issues
2. **Apply:** Adapt these patterns to your real use cases
3. **Measure:** Use the [Production Checklist](./checklist.md) before deploying
4. **Learn More:** Explore [Real-World Use Cases](./use-cases.md)

---

**Back to:** [Main Documentation](./README.md)

