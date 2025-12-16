# Pattern: Controlled Chain-of-Thought (Internal Trace)

Force the model to show its reasoning before producing an answer, making decisions debuggable and auditable.

## When to Use

- Complex reasoning tasks (pricing, diagnosis, planning, prioritization)
- Need **debuggable reasoning traces** for failed outputs
- Agents/tools where wrong intermediate steps must be visible
- Compliance/regulated scenarios requiring decision explanation

## Key Insight

By separating reasoning from the final answer, you can:
- Debug which reasoning step failed
- Validate logical consistency
- Build trust with stakeholders (show your work)
- Improve accuracy on multi-step problems

## Prompt Template

```text
You are a [ROLE].

Task:
[TASK_DESCRIPTION]

Follow this reasoning process:
1) Restate the problem in your own words.
2) List the key facts from the input.
3) Consider at least 2 options and compare them.
4) Decide and justify your choice.

First, think through the problem step by step inside a
<reasoning>...</reasoning> block.
Then, provide the final user-facing result inside an
<answer>...</answer> block.

The <answer> must be concise and not include the reasoning.

Input:
[INPUT_BLOCK]
```

## Working Example

### Scenario
Product feature prioritization from user feedback

### Prompt

```text
You are a senior product manager.

Task:
Suggest the most impactful next feature based on recent user feedback.

Follow this reasoning process:
1) Restate the problem.
2) Extract main pain points and counts.
3) Map pain points to possible features.
4) Choose 1 feature and justify briefly.

First, think step by step in:
<reasoning>...</reasoning>
Then answer for the product team in:
<answer>...</answer>

Input:
- 12 users: "search is slow and incomplete"
- 5 users: "dark mode would be nice"
- 9 users: "mobile navigation is confusing"
```

### Expected Output

```xml
<reasoning>
1) Problem: Prioritize the most impactful feature from three areas of feedback.

2) Key facts:
   - Search issues: 12 users (46% of feedback)
   - Dark mode: 5 users (19% of feedback)
   - Mobile navigation: 9 users (35% of feedback)

3) Options:
   - Option A: Fix search (12 users, core functionality)
   - Option B: Add dark mode (5 users, nice-to-have)
   - Option C: Improve mobile nav (9 users, affects mobile users)

   Search affects the most users and is a core product function. Mobile nav affects a significant portion but is platform-specific. Dark mode has the least demand.

4) Decision: Prioritize search improvements. It has the highest user count, and search is mission-critical for product discovery. Mobile nav should be second priority.
</reasoning>

<answer>
Build search improvements. 12 users reported issues, making it the most frequently mentioned pain point and a core product capability that affects discoverability.
</answer>
```

## Evaluation Criteria

- **Reasoning completeness:** All steps from template present
- **Fact grounding:** Reasoning references actual input data
- **Logical consistency:** Answer follows from reasoning
- **Debuggability:** Can pinpoint which reasoning step failed
- **Accuracy:** Final decision matches expert judgment on test set

## Variations

### 1. JSON Reasoning (for programmatic parsing)

```text
Output format:

{
  "reasoning": {
    "problem": "string",
    "facts": ["string", ...],
    "options": [{"name": "string", "pros": "string", "cons": "string"}],
    "decision": "string"
  },
  "answer": "string"
}
```

### 2. Confidence Scoring

```text
In your reasoning, include:
5) Assign a confidence score (low/medium/high) and explain why.
```

### 3. Multi-Stage Reasoning

```text
Follow this process:
1) Analyze: [specific analysis task]
2) Synthesize: [combine findings]
3) Recommend: [make decision]
4) Risk-check: [identify potential issues]
```

## Advanced Techniques

### 1. Constrained Reasoning Format

Force specific reasoning structure:

```text
<reasoning>
**Problem:** [one sentence]
**Key Facts:** [bullet list]
**Options:** [bullet list with pros/cons]
**Decision:** [one sentence with justification]
</reasoning>
```

### 2. Reasoning Templates per Domain

**For diagnosis:**
```text
1) Symptoms observed
2) Possible causes (at least 3)
3) Most likely cause and why
4) Recommended action
```

**For pricing:**
```text
1) Cost factors identified
2) Market comparables
3) Value justification
4) Recommended price
```

### 3. Self-Critique

```text
Follow this process:
1) [Generate initial answer]
2) Critique: What could be wrong with this answer?
3) Revise: Provide improved answer
4) Final answer
```

## Common Pitfalls

### Pitfall 1: Reasoning in Answer

❌ **Model mixes reasoning into final answer:**
```xml
<answer>
Based on the 12 users who complained about search, and considering that search is core functionality, we should prioritize search improvements.
</answer>
```

**Fix:**
```text
The <answer> must be a direct response suitable for end users.
Do not include your reasoning process in the answer.
```

### Pitfall 2: Skipped Steps

❌ **Model skips directly to conclusion**

**Fix:**
```text
You MUST complete all 4 reasoning steps before providing an answer.
Do not skip steps.
```

### Pitfall 3: Reasoning Doesn't Match Answer

❌ **Reasoning says Option A, answer recommends Option B**

**Fix:**
```text
Your <answer> must be consistent with your <reasoning>.
If your reasoning leads to Option A, your answer must recommend Option A.
```

### Pitfall 4: Too Verbose

❌ **Reasoning is 1000 words for a simple decision**

**Fix:**
```text
Keep reasoning concise:
- Problem: 1 sentence
- Facts: Bullet points only
- Options: 2-3 options max
- Decision: 1-2 sentences
```

## Debugging with Reasoning Traces

When outputs are wrong:

1. **Read the reasoning block** to find where logic breaks
2. **Check if facts are correctly extracted** from input
3. **Verify options considered** cover the actual possibilities
4. **Identify the bad reasoning step** and add specific instruction

**Example debug:**
```
Problem: Model keeps recommending Option B
Debug: Reasoning shows it's overweighting recency of feedback
Fix: Add rule "Weight by user count, not recency"
```

## Cost Optimization

Chain-of-thought adds tokens to output:
- Use only when debugging/explainability needed
- For production, run with reasoning during development
- Once stable, optionally remove reasoning requirement
- Consider: reasoning during eval, direct answers in production

## Testing Strategy

```python
import re

def test_reasoning_present(output: str):
    """Reasoning block must exist"""
    assert '<reasoning>' in output and '</reasoning>' in output

def test_answer_present(output: str):
    """Answer block must exist"""
    assert '<answer>' in output and '</answer>' in output

def test_reasoning_completeness(output: str, required_sections: list[str]):
    """All reasoning sections present"""
    reasoning = re.search(r'<reasoning>(.*?)</reasoning>', output, re.DOTALL).group(1)
    for section in required_sections:
        assert section in reasoning.lower()

def test_answer_conciseness(output: str, max_words: int):
    """Answer should be concise"""
    answer = re.search(r'<answer>(.*?)</answer>', output, re.DOTALL).group(1)
    word_count = len(answer.split())
    assert word_count <= max_words
```

## Real-World Applications

### Medical/Legal Reasoning
- Document diagnosis process
- Show precedent consideration
- Explain risk factors

### Financial Analysis
- Document calculation steps
- Show comparable analysis
- Justify recommendations

### Agent Decision-Making
- Log why tool was chosen
- Show parameter selection logic
- Enable failure diagnosis

### Content Moderation
- Document policy matching
- Show severity assessment
- Justify action taken

## Related Patterns

- [Role + Task + Rules + Format](./role-task-rules-format.md) - Base structure
- [Task Decomposition](./task-decomposition.md) - Break reasoning into separate calls
- [Defensive Error Reporting](./defensive-error-reporting.md) - Handle reasoning failures

---

**Back to:** [All Patterns](./README.md) | [Main Documentation](../README.md)

