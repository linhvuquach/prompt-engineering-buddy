# Debugging & Optimization

Common failure modes in production prompts and proven strategies to fix them.

## Table of Contents

- [Common Failures](#common-failures)
- [Fix Strategies](#fix-strategies)
- [Debugging Workflow](#debugging-workflow)
- [Performance Optimization](#performance-optimization)

---

## Common Failures

### 1. Hallucinations / "Confident Nonsense"

**Symptoms:**
- Model answers beyond provided context
- Invented fields in JSON output (e.g., `user_id: 12345` when no ID was mentioned)
- Names, IDs, or facts not present in source material
- "Helpful" additions of related information not requested

**Example:**

Input:
```text
Context: "Enterprise plan includes SSO."
Question: "What authentication methods does Enterprise support?"
```

❌ **Bad Output:**
```text
Enterprise supports SSO, SAML, OAuth2, and Active Directory integration.
```
(Only SSO was mentioned; rest is hallucinated)

**Root Causes:**
- No explicit constraint against using background knowledge
- Missing "I don't know" path in prompt
- Temperature too high for factual tasks
- Prompt encourages "helpfulness" over accuracy

---

### 2. Format Drift

**Symptoms:**
- Extra prose around JSON (e.g., "Here is the result: {...}")
- Missing required keys or unexpected extra keys
- Inconsistent types (string vs number, array vs single value)
- Valid JSON but doesn't match your schema

**Example:**

❌ **Bad Output:**
```text
Based on the ticket, here's what I found:
{
  "intent": "bug",
  "priority": "high",
  "confidence": 0.95,
  "notes": "Customer seems frustrated"
}
```

Expected only `intent` and `priority` keys, got extra keys and prose.

**Root Causes:**
- Schema not explicitly defined in prompt
- Missing "JSON only, no other text" instruction
- No examples showing exact format
- Prompt uses conversational tone

---

### 3. Style Drift

**Symptoms:**
- Output gets more verbose over time
- Ignores tone constraints (e.g., becomes formal when you want casual)
- Switches format (bullets → paragraphs)
- Inconsistent voice across multiple generations

**Example:**

First run (good):
```text
- Fixed invoice bug
- Added CSV export
```

Later run (drifted):
```text
We are pleased to announce the following improvements:
- We have successfully resolved an issue affecting invoice generation
- Users can now export data in CSV format for their convenience
```

**Root Causes:**
- Style rules too vague ("be professional")
- Not enough examples demonstrating target style
- No length constraints
- Model defaults to "helpful assistant" voice

---

### 4. Hidden Ambiguity

**Symptoms:**
- Model behavior flips between runs on same input
- Different valid interpretations of task produce different outputs
- Inconsistent handling of missing information
- Edge cases handled differently each time

**Example:**

Input: "Ticket from John about billing"

Run 1: `{"customer": "John"}`  
Run 2: `{"customer": null}` (since no last name)  
Run 3: `{"customer": "John", "customer_id": null}`

All three interpretations are "valid" given ambiguous prompt.

**Root Causes:**
- Prompt doesn't specify what to do when info is partial
- Multiple valid interpretations not addressed
- No decision rule for edge cases
- Missing examples of ambiguous cases

---

### 5. Prompt Drift

**Symptoms:**
- Same prompt produces different results over time
- "It worked yesterday" but fails today
- Inconsistent outputs across team members
- Can't reproduce issues

**Example:**

```python
# Original prompt (v1.0)
prompt = "Classify sentiment as positive/negative/neutral"

# Someone tweaks it
prompt = "Classify sentiment (positive, negative, or neutral)"

# Another person modifies
prompt = "You are a sentiment classifier. Classify as positive, negative, or neutral."

# Now outputs are inconsistent across versions
```

**Root Causes:**
- No version control for prompts
- Ad-hoc modifications without testing
- Multiple people editing same prompt
- No diff/change tracking
- Prompt stored as magic strings in code

**Impact:** Inconsistent outputs, degraded performance, impossible debugging

---

### 6. Cost Bloat

**Symptoms:**
- Prompt token count keeps growing
- Same instructions repeated across multiple prompts
- Too many examples for simple tasks
- Using large models for simple classification/extraction

**Example:**

❌ **Inefficient:**
```
Every prompt includes:
- 500 tokens of role description
- 10 examples (2000 tokens)
- Detailed formatting rules (300 tokens)
- Uses GPT-4 for simple "bug vs feature" classification
```

**Root Causes:**
- Copy-pasted boilerplate across prompts
- Haven't tested minimum examples needed
- Not using system prompts for shared instructions
- Model size not matched to task complexity

---

## Fix Strategies

### For Hallucinations

#### Strategy 1: Explicit Constraints

Add clear rules forbidding invention:

```text
Rules:
- If information is not present in the input, say "I don't know" or set the field to null.
- Use ONLY facts from the Context block.
- Do not add information from your training data.
- Do not infer beyond what is explicitly stated.
```

#### Strategy 2: Lower Temperature

For factual extraction and classification:
```
temperature = 0.0 to 0.3
```

For creative generation:
```
temperature = 0.7 to 1.0
```

#### Strategy 3: Guardrail Prompts

Use the **[Guardrailed Q&A](./patterns/guardrailed-qa.md)** pattern:
```text
You are a careful AI assistant that ONLY uses the provided context.
Task: Answer using ONLY the information in the Context block.
If the answer is not in the context, say: "I don't know based on the provided context."
```

#### Strategy 4: Two-Step Validation

```python
# Step 1: Generate answer
answer = model.generate(prompt, input)

# Step 2: Validate against source
validation_prompt = f"""
Does this answer contain ONLY information from the source?
Answer: {answer}
Source: {source}
Respond: YES or NO
"""
is_valid = model.generate(validation_prompt)
```

---

### For Format Drift

#### Strategy 1: Explicit Format Rules

```text
Output:
- Format: JSON only
- Do not include any explanation, introduction, or commentary
- Start your response with { and end with }
- No text before or after the JSON object
```

#### Strategy 2: Show Format Examples

Include 1-2 examples of **exactly** the format you want:

```text
Example output:
{
  "intent": "bug",
  "priority": "high"
}

Now process this input:
[...]
```

#### Strategy 3: Schema Enforcement

Define schema in prompt:

```text
Output JSON that matches this schema exactly:
{
  "intent": "bug|feature_request|question",
  "priority": "low|medium|high"
}

Do not add extra keys.
```

#### Strategy 4: Use Structured Output APIs

If available (OpenAI, Anthropic structured outputs):
```python
response = client.completions.create(
    model="gpt-4",
    messages=[...],
    response_format={
        "type": "json_schema",
        "json_schema": {
            "name": "ticket_classification",
            "schema": {
                "type": "object",
                "properties": {
                    "intent": {"type": "string", "enum": ["bug", "feature", "question"]},
                    "priority": {"type": "string", "enum": ["low", "medium", "high"]}
                },
                "required": ["intent", "priority"]
            }
        }
    }
)
```

---

### For Style Drift

#### Strategy 1: More Examples, Fewer Rules

Replace verbose style instructions with 3-5 solid examples:

❌ **Verbose instructions:**
```text
Write in a professional but approachable tone. Keep sentences short. 
Avoid jargon. Be direct. Use active voice. No marketing speak...
```

✅ **Show examples:**
```text
Examples of correct style:
[3 examples demonstrating exact style you want]
```

#### Strategy 2: Add Hard Length Constraints

```text
Rules:
- Maximum 150 words
- 2-3 paragraphs only
- Each bullet point: max 15 words
```

#### Strategy 3: Structure Templates

```text
Output must follow this structure:

**Problem:** [1 sentence]
**Solution:** [2-3 sentences]
**Next Steps:** [bulleted list]
```

#### Strategy 4: Negative Examples

Show what NOT to do:

```text
❌ INCORRECT style:
"We're thrilled to announce that our amazing team has been working tirelessly..."

✅ CORRECT style:
"We fixed the login bug affecting mobile users."
```

---

### For Ambiguity

#### Strategy 1: Enumerate Edge Cases

```text
Rules:
- If both shipping and billing addresses are present, use shipping for delivery.
- If only one address is present, use it for both.
- If no address is present, return error_code "MISSING_FIELD".
- If date is in the past, return error_code "INVALID_DATE".
```

#### Strategy 2: Decision Rules

```text
When multiple items match:
1. Prefer exact matches over partial matches
2. If multiple exact matches, choose the first one
3. If no matches, return null
```

#### Strategy 3: Two-Stage Processing

```python
# Stage 1: Classify ambiguity
classification = model.generate("""
Is this input ambiguous or unclear?
Respond: CLEAR or AMBIGUOUS
""", input)

if classification == "AMBIGUOUS":
    # Request clarification or use fallback logic
else:
    # Proceed with main task
```

#### Strategy 4: Example-Based Edge Cases

Include examples showing edge case handling:

```text
Example 1 (clear input):
Input: "Ship to: 123 Main St"
Output: {"address": "123 Main St"}

Example 2 (ambiguous input):
Input: "Ship to the office"
Output: {"ok": false, "error": "AMBIGUOUS_ADDRESS"}
```

---

### For Prompt Drift

#### Strategy 1: Version Control

Treat prompts as code:

```python
# prompts/ticket_classifier.py
PROMPT_VERSION = "2.1.0"

TICKET_CLASSIFIER_PROMPT = """
You are a support triage assistant.

Task: Classify ticket intent and priority.

Output JSON:
{
  "intent": "bug|feature|billing|question",
  "priority": "low|medium|high"
}

Ticket: {ticket_text}
"""

# Git commit with version bump
# Changelog: v2.1.0 - Added "billing" to intent types
```

#### Strategy 2: Centralized Prompt Registry

```python
# prompt_registry.py
class PromptRegistry:
    """Centralized prompt management"""
    
    prompts = {
        "ticket_classifier_v2": {
            "template": TICKET_CLASSIFIER_PROMPT,
            "version": "2.1.0",
            "created_by": "user@example.com",
            "last_updated": "2024-01-15",
            "test_accuracy": 0.92
        }
    }
    
    @classmethod
    def get_prompt(cls, name: str, version: str = "latest") -> str:
        """Retrieve specific prompt version"""
        if version == "latest":
            return cls.prompts[name]["template"]
        return cls.prompts[f"{name}_v{version}"]["template"]
```

#### Strategy 3: Automated Testing

Prevent regressions with tests:

```python
def test_prompt_v2_accuracy():
    """Ensure new prompt version doesn't regress"""
    
    test_cases = load_test_cases("ticket_classifier_eval.json")
    prompt = PromptRegistry.get_prompt("ticket_classifier_v2")
    
    accuracy = evaluate_prompt(prompt, test_cases)
    
    # Require 90% accuracy
    assert accuracy >= 0.90, f"Prompt accuracy {accuracy} below threshold"

def test_prompt_consistency():
    """Same input should produce same output"""
    
    prompt = PromptRegistry.get_prompt("ticket_classifier_v2")
    test_input = "Bug: Login button not working"
    
    results = [classify(prompt, test_input) for _ in range(5)]
    
    # All results should be identical (at temperature=0)
    assert all(r == results[0] for r in results)
```

#### Strategy 4: Change Documentation

```markdown
# Prompt Changelog

## v2.1.0 (2024-01-15)
### Changed
- Added "billing" to intent enum
- Improved priority detection for urgent keywords

### Performance
- Test accuracy: 92% (up from 89%)
- Cost per call: $0.002 (unchanged)

### Migration
- No breaking changes
- Backward compatible with v2.0.0
```

---

### For Cost

#### Strategy 1: Move to System Prompt

If using OpenAI/Anthropic, put shared instructions in system prompt:

```python
# ❌ Inefficient: repeat in every user message
messages = [
    {"role": "user", "content": f"{LONG_INSTRUCTIONS}\n\nInput: {input}"}
]

# ✅ Efficient: instructions once in system prompt
messages = [
    {"role": "system", "content": LONG_INSTRUCTIONS},
    {"role": "user", "content": input}
]
```

System prompts are often cached automatically.

#### Strategy 2: Minimize Examples

Test: Can you get 90% quality with 2 examples instead of 5?

```python
# A/B test different example counts
for num_examples in [1, 2, 3, 5, 10]:
    accuracy = test_prompt_with_n_examples(num_examples)
    cost = calculate_cost(num_examples)
    print(f"{num_examples} examples: {accuracy}% accuracy, ${cost}")
```

Pick minimum examples that meet quality bar.

#### Strategy 3: Model Right-Sizing

Task complexity determines model size:

| Task Type | Model Suggestion |
|-----------|------------------|
| Simple classification (2-5 classes) | GPT-3.5 Turbo, Claude Haiku |
| JSON extraction (fixed schema) | GPT-3.5 Turbo, Claude Haiku |
| Complex reasoning | GPT-4, Claude Opus |
| Long-form generation | GPT-4, Claude Opus |
| Style-sensitive writing | GPT-4, Claude Sonnet |

#### Strategy 4: Prompt Caching

Cache long static context:

```python
# Cacheable: product docs, examples, long instructions
system_context = load_documentation()  # 10K tokens

# Dynamic: user question
user_query = get_user_input()  # 50 tokens

# Provider-specific caching (e.g., Anthropic prompt caching)
response = client.messages.create(
    model="claude-3-5-sonnet-20241022",
    system=[
        {"type": "text", "text": system_context, "cache_control": {"type": "ephemeral"}}
    ],
    messages=[{"role": "user", "content": user_query}]
)
```

---

## Debugging Workflow

### Step 1: Reproduce the Failure

- Collect the exact input that failed
- Run it 5-10 times to see if failure is consistent
- Log both prompt and output

### Step 2: Identify Failure Type

Match to one of the categories:
- **Hallucination:** Wrong facts, invented data
- **Format:** Structure/parsing issues
- **Style:** Tone/length/voice issues
- **Ambiguity:** Inconsistent behavior
- **Other:** Logic errors, incomplete output

### Step 3: Apply Targeted Fix

Use strategies from the relevant section above.

### Step 4: Test Fix on Edge Cases

Don't just test on the one failure:
- Create 5-10 similar edge cases
- Test prompt on all of them
- Ensure fix doesn't break existing working cases

### Step 5: Measure Impact

```python
# Before fix
accuracy_before = test_on_eval_set(old_prompt)

# After fix
accuracy_after = test_on_eval_set(new_prompt)

# Compare
improvement = accuracy_after - accuracy_before
print(f"Accuracy improved by {improvement}%")
```

---

## Performance Optimization

### Latency Optimization

**1. Use Streaming**
```python
for chunk in client.chat.completions.create(stream=True, ...):
    print(chunk.choices[0].delta.content)
```

**2. Parallel Calls**
```python
import asyncio

async def process_batch(inputs):
    tasks = [model.generate_async(prompt, inp) for inp in inputs]
    return await asyncio.gather(*tasks)
```

**3. Reduce Output Tokens**
```text
Rules:
- Keep response under 100 words
- Use abbreviations where clear
- Omit unnecessary explanations
```

### Accuracy Optimization

**1. Build Evaluation Set**
```python
eval_set = [
    {"input": "...", "expected_output": "..."},
    # 50-100 examples
]
```

**2. Measure Baseline**
```python
def accuracy(prompt, eval_set):
    correct = 0
    for example in eval_set:
        output = model.generate(prompt, example['input'])
        if output == example['expected_output']:
            correct += 1
    return correct / len(eval_set)
```

**3. Iterate on Failures**
```python
failures = [ex for ex in eval_set if model_output != ex['expected_output']]
# Analyze failure patterns
# Add rules/examples targeting those patterns
# Re-test
```

### Cost Optimization Checklist

- [ ] Use smallest model that meets accuracy bar
- [ ] Cache long static context
- [ ] Remove redundant examples (A/B test)
- [ ] Use system prompts for shared instructions
- [ ] Batch requests when possible
- [ ] Set max_tokens to prevent runaway generation
- [ ] Use prompt compression tools (e.g., LLMLingua)

---

## Parameter Tuning Guide

Beyond the prompt itself, API parameters significantly affect output quality and cost.

### Temperature

Controls randomness/creativity.

| Value | Best For | Example Use Cases |
|-------|----------|-------------------|
| 0.0 - 0.3 | Deterministic, factual tasks | Data extraction, classification, code generation |
| 0.4 - 0.7 | Balanced creativity | Q&A, summarization, translation |
| 0.8 - 1.0 | Creative generation | Marketing copy, brainstorming, creative writing |
| 1.0 - 2.0 | Maximum creativity | Experimental, artistic generation |

**Debug tip:** If outputs are inconsistent, try temperature=0 to eliminate randomness.

### Top-P (Nucleus Sampling)

Alternative to temperature. Controls diversity by probability mass.

- **top_p=1.0:** Consider all tokens (default)
- **top_p=0.9:** Consider top 90% probability mass (recommended for most tasks)
- **top_p=0.1:** Very focused, deterministic

**General rule:** Use temperature OR top_p, not both. If using top_p, set temperature=1.

### Max Tokens

Limits output length.

```python
# For classification/extraction (save cost)
max_tokens=100

# For summaries
max_tokens=500

# For long-form generation
max_tokens=2000

# Safety: prevent runaway generation
max_tokens=4000  # Hard cap even for long tasks
```

### Frequency Penalty & Presence Penalty

Reduce repetition:

- **frequency_penalty** (0.0 to 2.0): Penalizes tokens based on frequency
  - 0.0: No penalty (default)
  - 0.5 to 1.0: Moderate reduction of repetition
  - 1.0+: Strong penalty (use sparingly)

- **presence_penalty** (0.0 to 2.0): Penalizes tokens that already appeared
  - Use when you want diverse topics/vocabulary

**Debug tip:** If output is repetitive, add `frequency_penalty=0.7`

### Stop Sequences

Force generation to stop at specific tokens:

```python
# Stop at end of JSON object
stop=["}"]

# Stop at specific markers
stop=["</answer>", "\n\n---"]

# Multiple stop sequences
stop=["END", "STOP", "\n\n\n"]
```

### Recommended Settings by Task

```python
# Data extraction
{
    "temperature": 0.0,
    "max_tokens": 200,
    "top_p": 1.0
}

# Customer support Q&A
{
    "temperature": 0.3,
    "max_tokens": 500,
    "frequency_penalty": 0.3  # Reduce repetitive phrasing
}

# Creative content
{
    "temperature": 0.9,
    "max_tokens": 2000,
    "presence_penalty": 0.6  # Encourage topic diversity
}

# Code generation
{
    "temperature": 0.2,
    "max_tokens": 1500,
    "stop": ["```", "# End"]  # Stop at code fence
}
```

---

## Debugging Tools

### Logging Template

```python
import logging

logger = logging.getLogger(__name__)

def log_llm_call(prompt, input, output, metadata):
    logger.info({
        "prompt_version": metadata.get("version"),
        "model": metadata.get("model"),
        "input_tokens": count_tokens(prompt + input),
        "output_tokens": count_tokens(output),
        "latency_ms": metadata.get("latency"),
        "input_hash": hash(input),
        "output": output,
        "timestamp": datetime.now().isoformat()
    })
```

### Eval Harness Example

```python
def run_eval(prompt_template, test_cases, model):
    results = {
        "total": len(test_cases),
        "passed": 0,
        "failed": 0,
        "failures": []
    }
    
    for case in test_cases:
        output = model.generate(prompt_template, case["input"])
        
        if validate(output, case["expected"]):
            results["passed"] += 1
        else:
            results["failed"] += 1
            results["failures"].append({
                "input": case["input"],
                "expected": case["expected"],
                "actual": output
            })
    
    results["accuracy"] = results["passed"] / results["total"]
    return results
```

---

## Related Resources

- **[Patterns](./patterns/):** Use correct pattern to prevent failures
- **[Exercises](./exercises.md):** Practice debugging flaky prompts
- **[Checklist](./checklist.md):** Pre-production quality gate
- **[Core Mental Models](./core-mental-models.md):** Defensive prompting principles

---

**Back to:** [Main Documentation](./README.md)

