# Pattern: Guardrailed Factual Answering (Low Hallucination)

Force the model to answer only from provided context, eliminating hallucinations and invented information.

## When to Use

- RAG (Retrieval-Augmented Generation) systems
- Documentation Q&A and support bots
- Compliance or regulated domains where accuracy is critical
- Any scenario where "I don't know" is better than a wrong answer

## Key Insight

Models will confidently make up information if you don't explicitly forbid it. This pattern forces honest "I don't know" responses.

## Prompt Template

```text
You are a careful AI assistant that ONLY uses the provided context.

Task:
Answer the user's question using ONLY the information in the Context block.

Rules:
- If the answer is not in the context, say:
  "I don't know based on the provided context."
- Do not guess or add outside knowledge.
- Do not infer beyond what is explicitly stated.
- Quote short phrases from the context where useful.

Context:
[DOCUMENT_SNIPPETS_OR_RETRIEVED_CONTEXT]

Question:
[USER_QUESTION]

Answer:
```

## Working Example

### Scenario
SaaS pricing documentation Q&A bot

### Prompt

```text
You are a careful AI assistant that ONLY uses the provided context.

Task:
Answer the user's question using ONLY the information in the Context block.

Rules:
- If the answer is not in the context, say:
  "I don't know based on the provided context."
- Do not guess or add outside knowledge.
- Do not infer beyond what is explicitly stated.

Context:
1) "The Pro plan includes up to 10 team members and 1 TB of storage."
2) "The Enterprise plan includes SSO, custom roles, and a dedicated account manager."

Question:
Does the Pro plan include SSO?

Answer:
```

### Expected Output

```text
I don't know based on the provided context.
```

### Correct Positive Example

**Question:** How many team members does the Pro plan support?

**Answer:**
```text
The Pro plan includes up to 10 team members.
```

## Evaluation Criteria

Critical metrics:
- **Hallucination rate:** On "unanswerable" questions, model must say "I don't know" ≥95% of the time
- **No contradictions:** 0% contradiction rate with source text
- **No invented features:** Never mention features not in context
- **Accuracy on answerable questions:** ≥95% correct when answer is in context

## Advanced Techniques

### 1. Structured "Don't Know" Response

```text
If the answer is not in the context, respond with exactly:
{
  "answer": null,
  "reason": "Information not found in provided context"
}
```

### 2. Confidence Indicators

```text
For each answer, indicate whether:
- CERTAIN: Directly stated in context
- INFERRED: Logically follows from context
- UNKNOWN: Not determinable from context

Only provide CERTAIN answers. Return "I don't know" for INFERRED or UNKNOWN.
```

### 3. Citation Required

```text
Rules:
- Always quote the relevant part of the context in your answer.
- Format: [Your answer] (Source: "quoted text")
- If you cannot find relevant text to quote, say "I don't know."
```

**Example:**
```text
Answer: The Pro plan includes up to 10 team members. (Source: "The Pro plan includes up to 10 team members and 1 TB of storage.")
```

### 4. Multi-Document Context

```text
Context:
[Doc 1: Pricing Page]
"The Pro plan costs $99/month."

[Doc 2: Features Page]
"Pro includes API access and webhooks."

Question: What does the Pro plan include?

Answer:
The Pro plan costs $99/month and includes API access and webhooks.
```

## Common Pitfalls

### Pitfall 1: Inferring Beyond Text

❌ **User:** "Does Pro plan work for small teams?"  
❌ **Model:** "Yes, with 10 team members, it's suitable for small teams."

Context only said "10 team members" – "suitable for small" is an inference.

**Fix:**
```text
Rules:
- Answer only with facts explicitly stated.
- Do not make judgments or recommendations beyond the text.
```

### Pitfall 2: Using Background Knowledge

❌ **User:** "What is SSO?"  
❌ **Model:** "SSO stands for Single Sign-On, which allows users to log in once..."

Context didn't define SSO – model used its training data.

**Fix:**
```text
Rules:
- Do not use your background knowledge.
- If the context doesn't explain a term, do not explain it.
```

### Pitfall 3: Partial Information Hallucination

Context: "Enterprise includes SSO."  
❌ **User:** "Does Enterprise include MFA?"  
❌ **Model:** "Yes, Enterprise includes SSO and MFA."

Model added MFA because it often comes with SSO.

**Fix:**
```text
Rules:
- List ONLY features explicitly mentioned in the context.
- Do not add related features that "usually" come together.
```

### Pitfall 4: Creative "I Don't Know" Variations

❌ **Model:** "The context doesn't specify this, but typically..."

**Fix:**
```text
If you don't know, respond with EXACTLY:
"I don't know based on the provided context."

Do not add qualifiers, speculation, or general information.
```

## Testing Strategy

### Adversarial Test Set

Create questions designed to trick the model:

1. **Unanswerable questions:** Answer not in context
2. **Partial information questions:** Some facts present, others missing
3. **Negative questions:** "Does X NOT include Y?" (requires careful reading)
4. **Implied information questions:** Facts that seem obvious but aren't stated

Aim: 100% "I don't know" on adversarial questions.

### Regression Testing

```python
def test_unanswerable_questions(model, test_cases):
    """Model must refuse to answer when info missing"""
    for context, question in test_cases['unanswerable']:
        answer = model(context, question)
        assert "don't know" in answer.lower() or "not in" in answer.lower()

def test_no_outside_knowledge(model, test_cases):
    """Model must not use training data"""
    for context, question, forbidden_terms in test_cases['outside_knowledge']:
        answer = model(context, question)
        for term in forbidden_terms:
            assert term.lower() not in answer.lower()

def test_no_contradiction(model, test_cases):
    """Answer must not contradict context"""
    for context, question, answer in test_cases['answerable']:
        model_answer = model(context, question)
        # Use another LLM to check for contradictions
        assert not check_contradiction(context, model_answer)
```

## Optimization for RAG

### 1. Chunk Quality Matters

Good context retrieval = good answers:
```text
✅ Include complete sentences/paragraphs
✅ Include surrounding context for clarity
❌ Don't truncate mid-sentence
❌ Don't retrieve chunks that need other chunks to make sense
```

### 2. Mark Chunk Boundaries

```text
Context:
[Chunk 1 from "Pricing Page"]
"The Pro plan costs $99/month."

[Chunk 2 from "Features Page"]
"Pro includes API access."

Question: [...]
```

### 3. Relevance Filtering

Add a pre-check:
```text
First, evaluate if the context is relevant to the question.
If not relevant, immediately respond: "I don't know based on the provided context."
```

### 4. Handle Empty Context

```text
If the Context block is empty or contains only "[No relevant information found]", respond:
"I don't know based on the provided context."
```

## Real-World Applications

### Support Documentation Bot

- Context: Retrieved doc sections
- Users ask: "How do I configure X?"
- Bot answers from docs or admits ignorance

### Compliance Q&A

- Context: Policy documents
- Users ask: "What's the approval process for Y?"
- Bot quotes exact policy text

### Internal Knowledge Base

- Context: Company wikis, past decisions
- Employees ask: "What was decided about Z?"
- Bot surfaces exact information or says it's not documented

### Medical/Legal Research Assistant

- Context: Case law, medical literature excerpts
- Professionals ask specific questions
- Assistant cites only provided sources

## Cost Optimization

- Use smaller models (context-following doesn't need huge models)
- Cache long static contexts
- Implement semantic caching for common questions
- Use retrieval systems that return fewer, higher-quality chunks

## Related Patterns

- [Role + Task + Rules + Format](./role-task-rules-format.md) - Base structure
- [Defensive Error Reporting](./defensive-error-reporting.md) - Structured "don't know" responses
- [Structured Output](./structured-output.md) - JSON format with confidence scores

---

**Back to:** [All Patterns](./README.md) | [Main Documentation](../README.md)

