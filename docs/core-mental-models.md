# Core Mental Models

These five principles form the foundation of production-grade prompt engineering. Internalize them before diving into specific patterns.

## 1. Prompt as Config, Not Chat

**Principle:** Treat prompts like code or configuration files, not conversation.

- Every word is a constraint
- Structure matters more than "clever wording"
- Prompts should be versioned, reviewed, and tested like code
- Avoid conversational filler ("Can you...", "Please...")

**Example:**

❌ **Chat-style (bad)**
```text
Hey, can you help me summarize this support ticket? Try to figure out what the user wants and maybe suggest what we should do about it.
```

✅ **Config-style (good)**
```text
You are a support triage assistant.

Task: Extract intent and priority from the ticket.

Output format: JSON with keys {intent, priority, summary}
```

---

## 2. Output-First Design

**Principle:** Design the output format and schema first, then write instructions to produce it.

- Start by defining what your code needs to consume
- If post-processing is painful, your prompt is wrong
- Schema-driven prompts catch errors early

**Workflow:**

1. Define exact output shape (JSON schema, markdown structure, etc.)
2. Write evaluation criteria (parsing, validation, business logic)
3. Write prompt to produce that output
4. Test and iterate

**Key insight:** If you're writing complex parsing logic after the LLM call, you failed at prompt design.

---

## 3. Examples > Instructions

**Principle:** Models learn better from examples than from long instruction blocks.

- Pattern-matching is the core strength of LLMs
- A few high-quality examples beat paragraphs of rules
- Use examples for: style, tone, formatting, edge cases

**When to use examples:**
- Style/voice alignment (marketing copy, docs)
- Complex output structures
- Handling ambiguous inputs
- Demonstrating edge case behavior

**When instructions work better:**
- Hard constraints (security, compliance)
- Binary rules (never/always behaviors)
- Mathematical/logical operations

---

## 4. Decompose Tasks

**Principle:** Split complex tasks into smaller, checkable steps.

For production systems, prefer:
- Multiple cheap LLM calls with clear responsibilities
- Pipeline: classify → extract → generate
- Each step produces structured output the next step consumes

Over:
- One giant "do everything" prompt
- Hoping the model figures out the right sequence
- Debugging failures in a monolithic call

**Benefits:**
- Easier debugging (know which step failed)
- Better caching (reuse classification results)
- Lower cost (use small models for simple steps)
- Testability (unit test each stage)

**Example pipeline:**
```
User input
  → Call 1: Classify intent (small model)
  → Call 2: Extract entities (medium model)
  → Call 3: Generate response (large model, only if needed)
```

---

## 5. Balance Clarity with Brevity

**Principle:** Include all essential details and constraints, but remove unnecessary words to save tokens and reduce confusion.

- **Be precise, not verbose:** "Extract company name (string)" beats "Please try to extract what you think might be the company name if there is one"
- **Use structure over prose:** Bullet points and schemas are clearer than paragraphs
- **Cut filler words:** Remove "please", "try to", "you should", "it would be great if"
- **Each token has purpose:** If removing a word doesn't change behavior, remove it

**Example comparison:**

**Verbose (wasteful):**
```text
Hello! I need you to please help me by trying to extract some information from the following text. If you can, please try to find the company name and also the revenue if it's mentioned anywhere. It would be really helpful if you could provide this in a JSON format, but only if that's possible for you to do. Thanks!
```

**Concise (effective):**
```text
Extract from the text:
- company_name (string)
- revenue (number or null)

Output: JSON only
```

**When to be verbose:** Safety-critical rules, complex edge cases, and disambiguation need explicit detail. Don't sacrifice clarity for brevity on critical constraints.

---

## 6. Defensive Prompting

**Principle:** Always define failure behavior explicitly. Force "don't know" paths instead of guessing.

Every production prompt must answer:
1. **What to do with missing data?**  
   → Return null, error code, or "I don't know"

2. **What to do with ambiguous input?**  
   → Ask for clarification or flag as `AMBIGUOUS`

3. **What is explicitly forbidden?**  
   → "Do not invent", "Do not use outside knowledge"

**Anti-pattern:**
```text
Extract the due date from this text.
```
→ Model will guess/hallucinate dates

**Defensive version:**
```text
Extract the due date from this text.

Rules:
- Return null if no explicit date is found
- Return error_code "AMBIGUOUS" if the date is relative (e.g., "next Friday")
- Do not infer or calculate dates
```

---

## Applying These Models

These mental models work together:

- **Output-first design** gives you the schema
- **Prompt as config** keeps it maintainable
- **Examples** teach the model the patterns
- **Decomposition** makes complex tasks tractable
- **Brevity** reduces cost and confusion
- **Defensive prompting** prevents production incidents

Every pattern in this playbook builds on these foundations.

## Quick Reference

| Situation | Apply This Model |
|-----------|------------------|
| Starting new prompt | Output-first design |
| Prompt too long | Balance clarity with brevity |
| Style inconsistency | Examples > instructions |
| Complex multi-step task | Decompose tasks |
| Hallucinations/errors | Defensive prompting |
| Team maintenance | Prompt as config |

---

**Next:** Explore specific [Prompt Patterns](./patterns/) that implement these principles.

