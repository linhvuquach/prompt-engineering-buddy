# Prompt Patterns

Production-ready prompt patterns for common use cases. Each pattern includes templates, examples, and evaluation criteria.

## Quick Reference

| Pattern | Best For | Model Size | Complexity |
|---------|----------|------------|------------|
| [Role + Task + Rules + Format](./role-task-rules-format.md) | Any production prompt (start here) | Any | ⭐ |
| [Structured Output](./structured-output.md) | JSON extraction, APIs, ETL | Small-Medium | ⭐⭐ |
| [Few-Shot Style](./few-shot-style.md) | Tone/voice control, marketing | Medium | ⭐⭐ |
| [Chain-of-Thought](./chain-of-thought.md) | Complex reasoning, debugging | Large | ⭐⭐⭐ |
| [Guardrailed Q&A](./guardrailed-qa.md) | RAG, docs, compliance | Small-Medium | ⭐⭐ |
| [Task Decomposition](./task-decomposition.md) | Complex workflows, agents | Mixed | ⭐⭐⭐⭐ |
| [Defensive Error Reporting](./defensive-error-reporting.md) | Data validation, APIs | Small | ⭐⭐ |

## By Use Case

### Data Extraction & ETL
- [Structured Output](./structured-output.md) - Extract fields into JSON
- [Defensive Error Reporting](./defensive-error-reporting.md) - Validate and normalize

### Customer Support
- [Role + Task + Rules + Format](./role-task-rules-format.md) - Ticket classification
- [Task Decomposition](./task-decomposition.md) - Triage → Extract → Respond

### Documentation & Q&A
- [Guardrailed Q&A](./guardrailed-qa.md) - Context-only answers
- [Few-Shot Style](./few-shot-style.md) - Consistent documentation voice

### Content Generation
- [Few-Shot Style](./few-shot-style.md) - Marketing copy, emails
- [Chain-of-Thought](./chain-of-thought.md) - Complex writing with reasoning

### Agents & Workflows
- [Task Decomposition](./task-decomposition.md) - Multi-step orchestration
- [Chain-of-Thought](./chain-of-thought.md) - Debuggable agent decisions
- [Defensive Error Reporting](./defensive-error-reporting.md) - Tool call validation

## Pattern Selection Guide

### Start Simple

1. **First time?** Use [Role + Task + Rules + Format](./role-task-rules-format.md)
2. **Need JSON?** Add [Structured Output](./structured-output.md)
3. **Still failing?** Try [Task Decomposition](./task-decomposition.md)

### Common Combinations

**RAG System:**
```
Retrieval → Guardrailed Q&A → Structured Output (with citations)
```

**Support Automation:**
```
Role+Task+Rules (classify) → Structured Output → Few-Shot (response)
```

**Content Pipeline:**
```
Task Decomposition (research → outline → draft → edit)
Each stage uses Few-Shot for style
```

**Agent with Tools:**
```
Chain-of-Thought (planning) → Defensive Error Reporting (validation) → Task Decomposition (execution)
```

## Pattern Relationships

```
Core: Role + Task + Rules + Format
├── + Strict Schema → Structured Output
├── + Examples → Few-Shot Style
├── + Reasoning → Chain-of-Thought
├── + Context-Only → Guardrailed Q&A
├── + Validation → Defensive Error Reporting
└── + Multiple Calls → Task Decomposition
```

## Anti-Patterns to Avoid

❌ **"Do everything" prompts**
- Trying to classify, extract, generate, and format in one call
- → Use [Task Decomposition](./task-decomposition.md)

❌ **Hoping for JSON**
- "Please return JSON" without schema or rules
- → Use [Structured Output](./structured-output.md)

❌ **Vague style instructions**
- "Be professional and friendly"
- → Use [Few-Shot Style](./few-shot-style.md) with examples

❌ **Allowing hallucinations**
- Not defining failure behavior
- → Use [Guardrailed Q&A](./guardrailed-qa.md) + [Defensive Error Reporting](./defensive-error-reporting.md)

❌ **Debugging black boxes**
- Can't see why model made a decision
- → Use [Chain-of-Thought](./chain-of-thought.md)

## Next Steps

1. **Read:** Pick a pattern relevant to your use case
2. **Practice:** Try the [Hands-On Exercises](../exercises.md)
3. **Debug:** Check [Debugging & Optimization](../debugging.md) if issues arise
4. **Ship:** Use the [Production Checklist](../checklist.md) before deploying

---

**Back to:** [Main Documentation](../README.md)

