# Pattern: Few-Shot Style & Quality Alignment

Use examples to teach the model your exact tone, voice, and quality standards.

## When to Use

- You care about **tone/voice** consistency (marketing, docs, customer emails)
- Want to avoid generic "LLM essay" style
- Need consistency across many generations
- Instructions alone don't capture the style you want

## Key Insight

Models are pattern-matching engines. A few high-quality examples often beat pages of style instructions.

## Prompt Template

```text
You are a [ROLE].

Follow the writing style shown in the examples below.

Style rules:
- [Rule 1: e.g., "short sentences, no fluff"]
- [Rule 2]
- [Rule 3]

Examples:

Input:
[EXAMPLE_INPUT_1]
Output:
[EXAMPLE_OUTPUT_1]

Input:
[EXAMPLE_INPUT_2]
Output:
[EXAMPLE_OUTPUT_2]

Now respond in the same style.

Input:
[NEW_INPUT]
Output:
```

## Working Example

### Scenario
Technical documentation for junior developers - consistent voice and structure

### Prompt

```text
You are a senior backend engineer explaining concepts to junior devs.

Style rules:
- Use plain language.
- Max 3 short paragraphs.
- Include 1 concrete example.
- Avoid metaphors and jokes.

Examples:

Input:
Explain REST APIs
Output:
REST APIs let two systems talk over HTTP using standard methods like GET, POST, PUT, and DELETE. Each URL represents a resource, such as a user or an order.

The server does not store client state between requests. That's why REST is called stateless. Each request contains everything needed: authentication, parameters, and body.

For example, GET /users/123 returns user 123. PUT /users/123 with a JSON body updates that user.

Input:
Explain Webhooks
Output:
Webhooks are HTTP callbacks that let one system notify another when something happens. Instead of polling an API, you receive an HTTP POST when an event occurs.

You configure a URL on your server. When an event fires, the other system sends an HTTP POST with a JSON payload describing the event.

For example, a payment provider can send a POST /webhooks/payment with the transaction ID and status when a charge succeeds.

Now respond in the same style.

Input:
Explain idempotent APIs
Output:
```

### Expected Output

```text
Idempotent APIs produce the same result no matter how many times you call them with the same input. This prevents duplicate actions when requests are retried due to network issues.

GET and PUT are typically idempotent. GET always returns the same resource. PUT sets a resource to a specific state, so calling it twice has the same effect as calling it once.

For example, PUT /users/123 {"name": "Alice"} sets the name to Alice. If you call it again, the name is still Alice. POST is usually not idempotent because it creates new resources each time.
```

## Evaluation Criteria

- **Word count:** ±20% of example outputs
- **Structure:** Same number of paragraphs, example present
- **Tone match:** Manual rating on 10 samples (aim ≥4/5)
- **Consistency:** Style doesn't drift across multiple generations
- **Content quality:** Factually correct while maintaining style

## How Many Examples?

- **Simple style changes:** 2-3 examples
- **Complex tone/structure:** 4-6 examples
- **Very specific formats:** 6-10 examples

More examples = better alignment, but diminishing returns after ~8.

## Advanced Techniques

### 1. Negative Examples

Show what NOT to do:

```text
Examples of INCORRECT style:

Input:
Explain REST APIs
Output:
❌ REST APIs are like a restaurant menu where you order food...
(Avoid: metaphors, casual tone)

Examples of CORRECT style:

Input:
Explain REST APIs
Output:
✅ REST APIs let two systems talk over HTTP...
```

### 2. Edge Case Examples

Include examples handling tricky inputs:

```text
Input:
Explain a concept that's too advanced
Output:
This topic requires understanding of [prerequisite]. I recommend learning [simpler concept] first.
```

### 3. Length Variations

Show how to handle different input sizes:

```text
Input (short):
[Brief question]
Output:
[2 paragraphs]

Input (detailed):
[Complex multi-part question]
Output:
[3 paragraphs, addressing each part]
```

## Common Mistakes

### Mistake 1: Examples Don't Match Target

❌ Examples are 500 words, but you want 100-word outputs  
✅ Examples are exactly the length you want

### Mistake 2: Inconsistent Example Quality

❌ Example 1 is great, Example 2 is sloppy  
✅ All examples represent your gold standard

### Mistake 3: Examples Without Style Rules

❌ Only showing examples, no explicit rules  
✅ Rules + Examples (rules catch what examples miss)

### Mistake 4: Too Many Examples

❌ 20 examples taking up 2000 tokens  
✅ 3-5 carefully chosen examples covering key variations

## Testing Strategy

### Automated Checks

```python
def test_output_length(output: str, examples: list[str]):
    """Output length should match examples ±20%"""
    avg_example_length = sum(len(e) for e in examples) / len(examples)
    assert 0.8 * avg_example_length <= len(output) <= 1.2 * avg_example_length

def test_structure(output: str, expected_paragraphs: int):
    """Check structural consistency"""
    paragraphs = output.split('\n\n')
    assert len(paragraphs) == expected_paragraphs
```

### Manual Review

Rate 10 outputs on:
1. Style match (1-5)
2. Tone appropriateness (1-5)
3. Structure consistency (1-5)

Target: Average ≥4/5 on all dimensions

## Use Cases by Domain

### Marketing Copy

```text
Style: Punchy, benefit-driven, under 50 words per item
Examples: Headlines, CTAs, feature descriptions
```

### Customer Support

```text
Style: Empathetic, clear, actionable
Examples: Ticket responses with apologies, solutions, next steps
```

### Release Notes

```text
Style: User-facing benefits, no jargon
Examples: Internal changelog → customer-friendly updates
```

### Code Comments

```text
Style: Concise, explains "why" not "what"
Examples: Good vs bad comments for same code
```

## Cost Optimization

Few-shot prompts are longer, so:
- Cache the prompt prefix (role + rules + examples)
- Use smaller models if style is simple
- A/B test: Can you get 90% quality with 2 examples instead of 5?

## Related Patterns

- [Role + Task + Rules + Format](./role-task-rules-format.md) - Base structure
- [Controlled Chain-of-Thought](./chain-of-thought.md) - Examples for reasoning steps
- [Core Mental Models](../core-mental-models.md) - "Examples > Instructions" principle

---

**Back to:** [All Patterns](./README.md) | [Main Documentation](../README.md)

