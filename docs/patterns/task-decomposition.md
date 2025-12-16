# Pattern: Task Decomposition (Multi-Call)

Break complex tasks into a pipeline of simpler, specialized prompts instead of one monolithic call.

## When to Use

- Complex workflows that are brittle in a single call
- Need better control, caching, and cost reduction
- Orchestrating agents or tools with multiple steps
- Want to test/debug individual stages independently

## Key Insight

Prefer:
- **3 simple prompts in sequence** → Each doing one thing well
- **Different model sizes** → Small for classification, large for generation
- **Cacheable intermediate results** → Reuse common classifications

Over:
- **1 giant prompt** → "Understand this, extract that, then generate..."

## High-Level Pattern

```
┌─────────────┐
│   Call 1    │  Classify / Understand / Plan
└──────┬──────┘
       │ (structured output)
       ▼
┌─────────────┐
│   Call 2    │  Execute / Extract / Transform
└──────┬──────┘
       │ (structured output)
       ▼
┌─────────────┐
│   Call 3    │  Review / Refine / Format (optional)
└─────────────┘
```

Each call:
- Has a single, clear responsibility
- Produces structured output (JSON)
- Can be tested independently
- Can use different models

## Prompt Template (Step 1: Planner)

```text
You are a task planner for a [DOMAIN] assistant.

Task:
Break the user request into 1–5 atomic steps that can be executed independently.

Rules:
- Steps must be ordered.
- Each step must be executable by a single LLM call.
- If the request is unclear, include a "clarify" step first.

Output (JSON):
{
  "steps": [
    {
      "id": "step-1",
      "description": "string",
      "type": "plan|analyze|generate|search|call_api"
    }
  ]
}

User request:
[USER_REQUEST]
```

Then feed each step into specialized prompts.

## Working Example

### Scenario
Customer support email → Ticket creation + auto-response

### Monolithic Approach (Fragile)

```text
You are a support assistant.

Read this email, figure out what the issue is, extract relevant details, create a ticket summary, assign priority, and write a response to the customer.

Email: [...]
```

Problems:
- Hard to debug (which part failed?)
- Can't cache classification
- Can't use different models per stage
- Inconsistent outputs

### Decomposed Approach (Robust)

**Call 1: Classify Intent & Priority**

```text
You are a support triage classifier.

Task: Classify the email's intent and priority.

Rules:
- Output JSON only.
- Use context from email to determine priority.

Output:
{
  "intent": "bug|feature_request|billing|question",
  "priority": "low|medium|high",
  "sentiment": "frustrated|neutral|happy"
}

Email: [...]
```

Model: Small/cheap (GPT-3.5, Claude Haiku)

---

**Call 2: Extract Key Details**

```text
You are a data extraction engine.

Task: Extract ticket fields from the email.

Output:
{
  "summary": "string (<=20 words)",
  "product_area": "string or null",
  "affected_users": "number or null",
  "due_date": "ISO date or null"
}

Email: [...]
```

Model: Small/medium

---

**Call 3: Generate Customer Response**

```text
You are a customer support agent.

Task: Write a response based on the ticket classification.

Input:
- Intent: {intent}
- Priority: {priority}
- Summary: {summary}

Style rules:
- Empathetic and professional
- Acknowledge the issue
- Set expectations for response time

Output: Plain text email response

Response:
```

Model: Large (GPT-4, Claude Opus) – only for final generation

## Benefits

| Aspect | Monolithic | Decomposed |
|--------|-----------|------------|
| **Debugging** | Hard - can't isolate failures | Easy - test each step |
| **Cost** | High - uses large model for everything | Low - small models for simple steps |
| **Caching** | None - reprocess everything | High - cache classification results |
| **Flexibility** | Rigid - one prompt does all | Flexible - swap out stages |
| **Testing** | E2E only | Unit test per stage |

## Evaluation Criteria

- **Pipeline success rate:** % of requests that complete all steps
- **Stage accuracy:** Test each stage independently on labeled data
- **Cost comparison:** Total tokens (decomposed) vs monolithic
- **Latency:** Measure if parallel calls or caching offsets sequential overhead

Aim: Higher success rate + lower cost than monolithic, even if slightly slower.

## Advanced Techniques

### 1. Conditional Branching

```python
# Pseudo-code
classification = call_1(input)

if classification['intent'] == 'billing':
    details = call_2_billing(input)
elif classification['intent'] == 'bug':
    details = call_2_bug(input)
else:
    details = call_2_general(input)

response = call_3(classification, details)
```

### 2. Parallel Execution

When steps don't depend on each other:

```python
# Run in parallel
classification = call_classify(input)
sentiment = call_sentiment(input)
entities = call_extract(input)

# Combine results
response = call_generate(classification, sentiment, entities)
```

### 3. Validation Layer

Add a check between steps:

```python
step1_output = call_1(input)

# Validate output
if not is_valid(step1_output):
    return error("Step 1 validation failed")

step2_output = call_2(step1_output)
```

### 4. Human-in-the-Loop

Insert approval gates:

```python
plan = call_planner(user_request)

# Show plan to user
if user.approves(plan):
    for step in plan['steps']:
        execute(step)
```

## Common Pitfalls

### Pitfall 1: Over-Decomposition

❌ Breaking a simple task into 10 micro-steps  
✅ Use 1 call if task is simple; decompose only when necessary

**Rule of thumb:** Decompose when monolithic fails >20% or debugging is hard.

### Pitfall 2: Poor Interface Between Steps

❌ **Call 1 output:** "The issue is about billing stuff"  
❌ **Call 2 input:** Expects structured JSON

**Fix:** Enforce structured output at every step:
```json
{"intent": "billing", "confidence": 0.95}
```

### Pitfall 3: No Error Handling

❌ Step 2 crashes if Step 1 returns unexpected format

**Fix:** Validate and handle errors:
```python
if 'intent' not in step1_output:
    return error("Classification failed")
```

### Pitfall 4: Decomposing Without Testing

❌ Assuming decomposition is better without measuring

**Fix:** A/B test:
- Monolithic prompt on 100 examples
- Decomposed pipeline on same 100 examples
- Compare: accuracy, cost, latency

## Real-World Patterns

### Content Generation Pipeline

```
1. Research: Extract key facts from sources
2. Outline: Generate structure from facts
3. Draft: Write sections from outline
4. Edit: Refine for tone and clarity
```

### Data Enrichment Pipeline

```
1. Parse: Extract fields from raw text
2. Validate: Check field consistency
3. Enrich: Add computed fields
4. Format: Export as JSON
```

### Agent Workflow

```
1. Intent: Determine user goal
2. Plan: Choose tools and parameters
3. Execute: Call tools in sequence
4. Synthesize: Generate final answer from results
```

### Document Q&A Pipeline

```
1. Retrieve: Semantic search for relevant chunks
2. Rerank: Score chunk relevance
3. Answer: Generate response from top chunks
4. Cite: Add source references
```

## Cost Optimization

Decomposition enables:
- **Model right-sizing:** Use GPT-3.5 for steps 1-2, GPT-4 only for step 3
- **Caching:** Cache classification results for similar inputs
- **Batch processing:** Run step 1 on 100 items, then step 2 on results
- **Early exit:** Stop pipeline if classification says "no action needed"

**Example savings:**
```
Monolithic: 100 requests × 2000 tokens × $0.03/1K = $6
Decomposed:
  Step 1: 100 × 500 tokens × $0.005/1K = $0.25
  Step 2: 100 × 800 tokens × $0.005/1K = $0.40
  Step 3: 50 × 1500 tokens × $0.03/1K = $2.25  (only 50 need generation)
Total: $2.90 (52% savings)
```

## Testing Strategy

### Unit Tests per Stage

```python
def test_stage_1_classification():
    """Test classification accuracy"""
    for input, expected in test_cases:
        output = classify(input)
        assert output['intent'] == expected['intent']

def test_stage_2_extraction():
    """Test extraction correctness"""
    for input, expected in test_cases:
        output = extract(input)
        assert output['summary'] == expected['summary']
```

### Integration Tests

```python
def test_full_pipeline():
    """Test end-to-end pipeline"""
    result = pipeline.run(test_input)
    assert result['success']
    assert 'response' in result
```

### Failure Mode Testing

```python
def test_pipeline_handles_bad_classification():
    """Pipeline should gracefully handle errors"""
    # Inject bad output from step 1
    result = pipeline.run_from_step(2, bad_classification)
    assert result['error'] == 'INVALID_INPUT'
```

## Related Patterns

- [Role + Task + Rules + Format](./role-task-rules-format.md) - Template for each stage
- [Structured Output](./structured-output.md) - Interface between stages
- [Defensive Error Reporting](./defensive-error-reporting.md) - Handle stage failures
- [Core Mental Models](../core-mental-models.md) - "Decompose tasks" principle

---

**Back to:** [All Patterns](./README.md) | [Main Documentation](../README.md)

