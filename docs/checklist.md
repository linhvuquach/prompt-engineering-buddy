# Production Checklist

Quality gate checklist for every production prompt. Use this before deploying any prompt to ensure it meets production standards.

## How to Use This Checklist

1. **Review each section** for your prompt
2. **Check off items** that are complete
3. **Fix gaps** before deploying
4. **Keep this checklist** with your prompt documentation

Not every item applies to every prompt. Use judgment, but err on the side of completeness.

---

## 1. Clarity

### Role & Task

- [ ] **Role is explicit and relevant**
  - ✅ "You are a senior data analyst..."
  - ❌ "You are a helpful assistant..."

- [ ] **Task is a single, clear action**
  - ✅ "Extract company name and revenue from the text"
  - ❌ "Help me understand this document"

- [ ] **No vague verbs without specifics**
  - ✅ "Classify intent as bug|feature|question"
  - ❌ "Help with", "discuss", "think about"

### Input Specification

- [ ] **Input format is defined**
  - What does the model receive? (raw text, JSON, email, etc.)

- [ ] **Input boundaries are clear**
  - Where does input start/end?
  - Are there multiple inputs?

---

## 2. Constraints

### Output Format

- [ ] **Output format is explicitly defined**
  - JSON, markdown, plain text, XML, etc.

- [ ] **Format requirements are specific**
  - ✅ "Output JSON only, no other text"
  - ❌ "Return as JSON"

- [ ] **No extra commentary rule (if applicable)**
  - "Do not include explanations or introductions"

### Length Constraints

- [ ] **Length limits defined (if needed)**
  - Max words/sentences/paragraphs
  - Token limits if relevant

- [ ] **Structure constraints (if needed)**
  - Number of bullet points
  - Paragraph count
  - Section headings

### Failure Behavior

- [ ] **"Don't know" path is explicit**
  - What should model do with missing data?
  - How to handle ambiguous inputs?

- [ ] **Error codes defined (if applicable)**
  - `MISSING_FIELD`, `INVALID_FORMAT`, `AMBIGUOUS`, etc.

- [ ] **Null vs error distinction clear**
  - When to use `null` vs error object?

---

## 3. Structure

### For Structured Outputs

- [ ] **Schema included in prompt**
  - Exact keys and types defined
  - Enum values listed if applicable

- [ ] **No extraneous keys allowed**
  - "Do not add extra keys"
  - "Output ONLY the keys defined in schema"

- [ ] **Type constraints specified**
  - String, number, boolean, array
  - Format constraints (dates, currencies, etc.)

### Examples (if used)

- [ ] **Examples mirror exact desired output**
  - Same format, structure, length
  - Not "hand-wavy" approximations

- [ ] **Examples cover edge cases**
  - Missing data handling
  - Ambiguous inputs
  - Multiple items

- [ ] **Example count is justified**
  - Have you tested with fewer examples?
  - Are you using minimum needed for quality?

---

## 4. Safety & Reliability

### Factual Tasks

- [ ] **"Do not guess" rule is explicit**
  - "If information is not present, return null"
  - "Do not infer beyond what is stated"

### Context-Only Tasks

- [ ] **Outside knowledge explicitly forbidden**
  - "Do not use your background knowledge"
  - "Answer ONLY from provided context"

- [ ] **"I don't know" phrase is exact**
  - Specify exact response when uncertain

### Error Conditions

- [ ] **Error conditions are explicit**
  - What counts as invalid input?
  - What validation rules must pass?

- [ ] **Error response format matches schema**
  - Errors return same JSON structure (or defined error schema)

---

## 5. Cost

### Token Efficiency

- [ ] **No duplicated global instructions**
  - Shared instructions moved to system prompt
  - Reusable context cached if possible

- [ ] **Example count minimized**
  - A/B tested: does 3 examples work as well as 6?

- [ ] **Model size appropriate for task**
  - Simple classification → small model (GPT-3.5, Haiku)
  - Complex reasoning → large model (GPT-4, Opus)
  - Content generation → medium-large model

### Optimization Opportunities

- [ ] **Prompt caching enabled (if available)**
  - Long static context marked for caching
  - Dynamic parts separated from static

- [ ] **Batch processing considered (if applicable)**
  - Can multiple inputs be processed together?

- [ ] **Task decomposition evaluated**
  - Would breaking into smaller steps reduce cost?

---

## 6. Evaluation

### Test Data

- [ ] **Eval set created (20-100+ samples)**
  - Representative of production inputs
  - Includes edge cases and failures

- [ ] **Ground truth labels exist**
  - What is the correct output for each?

### Metrics

- [ ] **Format validity tested**
  - 100% of outputs parse correctly
  - No exceptions on valid inputs

- [ ] **Accuracy measured**
  - Correctness on labeled eval set (aim ≥90%)
  - Per-field accuracy for structured outputs

- [ ] **Cost measured**
  - $/request calculated
  - Compared to alternatives (smaller models, decomposition)

### Failure Analysis

- [ ] **Failure modes logged**
  - Specific examples where prompt fails
  - Categorized by type (format, accuracy, edge case)

- [ ] **Fix strategy for each failure type**
  - Not just "it sometimes fails"

---

## 7. Anti-Goals (What NOT to Do)

- [ ] **Does not hallucinate** on your eval set
  - 0% invented facts/IDs/names on factual tasks

- [ ] **Does not add extra keys** to JSON
  - Schema violations: 0%

- [ ] **Does not use forbidden behavior**
  - e.g., no outside knowledge when context-only required

- [ ] **Does not ignore length constraints**
  - All outputs within specified limits

---

## 8. Documentation

### Prompt Versioning

- [ ] **Prompt has version identifier**
  - v1.0, v2.3, etc.

- [ ] **Changes logged**
  - What changed from previous version?
  - Why was it changed?

- [ ] **Performance tracked per version**
  - Can compare accuracy/cost across versions

### Integration Documentation

- [ ] **Input preprocessing documented**
  - What happens to input before prompt?

- [ ] **Output postprocessing documented**
  - What happens to output after generation?

- [ ] **Error handling flow documented**
  - How does code handle prompt failures?

---

## 9. Monitoring (Post-Deployment)

### Metrics to Track

- [ ] **Response time (p50, p95, p99)**
  - Is latency acceptable?

- [ ] **Format parsing success rate**
  - % of outputs that parse correctly

- [ ] **User feedback (if applicable)**
  - Thumbs up/down, satisfaction scores

- [ ] **Cost per request**
  - Track over time, alert on spikes

- [ ] **Error rate by type**
  - Track defensive errors (AMBIGUOUS, MISSING_FIELD)

### Alerting

- [ ] **Alerts configured for anomalies**
  - Parsing failure rate > 1%
  - Latency > threshold
  - Cost > budget

- [ ] **On-call process defined**
  - Who gets alerted?
  - What is escalation path?

---

## 10. Iteration Process

### Continuous Improvement

- [ ] **Failed cases collected**
  - Log production failures for analysis

- [ ] **Eval set updated with failures**
  - Add production failures to test set
  - Ensure fixes don't regress

- [ ] **Regular prompt reviews scheduled**
  - Monthly/quarterly prompt audits
  - Compare new models on existing prompts

---

## Checklist by Prompt Type

### Simple Classification

Must-haves:
- ✅ Clear role and task
- ✅ Explicit output format (JSON with enum values)
- ✅ Failure behavior for unclear inputs
- ✅ Eval set with ≥90% accuracy
- ✅ Small model (cost optimization)

### Data Extraction

Must-haves:
- ✅ Structured output with schema
- ✅ Defensive error reporting
- ✅ No hallucination rules
- ✅ Type validation
- ✅ Null vs error distinction

### RAG Q&A

Must-haves:
- ✅ Context-only constraint
- ✅ "I don't know" path
- ✅ No outside knowledge rule
- ✅ Citation mechanism
- ✅ Unanswerable question handling (95%+ correct)

### Content Generation

Must-haves:
- ✅ Style examples (few-shot)
- ✅ Length constraints
- ✅ Structure template
- ✅ Tone rules
- ✅ Consistency testing (same input, consistent style)

### Agent / Tool Calling

Must-haves:
- ✅ Tool list with schemas
- ✅ Reasoning trace (chain-of-thought)
- ✅ Argument validation
- ✅ "No tool appropriate" path
- ✅ Error handling for tool failures

---

## Final Gate

Before deploying, answer these questions:

1. **Can I explain this prompt's behavior to a non-technical stakeholder?**
   - If no, simplify or decompose.

2. **Do I have a test set and know my accuracy?**
   - If no, create one.

3. **What happens when the prompt fails?**
   - If unclear, add defensive error handling.

4. **Can I debug a failure in production?**
   - If no, add logging and reasoning traces.

5. **Is this the cheapest approach that meets quality bar?**
   - If unsure, test smaller models or decomposition.

---

## Example Checklist in Practice

### Prompt: Support Ticket Classifier

**Clarity**
- [x] Role: "Senior support triage assistant"
- [x] Task: "Classify ticket intent and priority"
- [x] Input: "Single customer ticket message"

**Constraints**
- [x] Output: JSON only
- [x] Schema: `{intent, priority, needs_human}`
- [x] Failure: Set to null if missing

**Structure**
- [x] Schema in prompt with enum values
- [x] No extra keys rule added
- [x] 2 examples included (tested: 2 = 5 examples in accuracy)

**Safety**
- [x] "Use only information explicitly in ticket"
- [x] "Do not invent product names"

**Cost**
- [x] Using GPT-3.5 Turbo (tested: 92% accuracy, GPT-4 = 94% for 10x cost)
- [x] Prompt cached via system message

**Evaluation**
- [x] 50-ticket eval set created
- [x] 92% accuracy on intent classification
- [x] 100% parsing success rate
- [x] No hallucinated product names in 50 samples

**Monitoring**
- [x] Logging all classifications
- [x] Alert if parsing failure rate > 2%
- [x] Track accuracy on random sample weekly

**Result: APPROVED FOR PRODUCTION** ✅

---

## Checklist Template

Copy this template for your prompts:

```markdown
## Prompt: [NAME] v[VERSION]

### Clarity
- [ ] Role explicit
- [ ] Task clear
- [ ] Input defined

### Constraints
- [ ] Output format specified
- [ ] Failure behavior defined
- [ ] Length limits (if needed)

### Structure
- [ ] Schema included (if JSON)
- [ ] Examples provided (if needed)
- [ ] No extra keys rule

### Safety
- [ ] No hallucination rules
- [ ] Error codes defined
- [ ] Context-only (if RAG)

### Cost
- [ ] Model size justified
- [ ] Examples minimized
- [ ] Caching enabled

### Evaluation
- [ ] Eval set created (size: ___)
- [ ] Accuracy measured: ___%
- [ ] Parsing success: ___%
- [ ] Cost/request: $___

### Approved: [ ] YES / [ ] NO
### Approved by: ___________
### Date: ___________
```

---

If you've checked all applicable items, your prompt is production-ready. Ship it! 🚀

---

**Related Documentation:**

- **[Patterns](./patterns/):** Reference patterns while filling checklist
- **[Debugging](./debugging.md):** Use when checklist items fail
- **[Exercises](./exercises.md):** Practice applying checklist
- **[Core Mental Models](./core-mental-models.md):** Foundation principles

---

**Back to:** [Main Documentation](./README.md)

