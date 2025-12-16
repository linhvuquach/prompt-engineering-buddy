# Practical Prompt Engineering Playbook

## 1. Core Mental Models (10%)

- **Prompt as config, not chat**  
  Treat prompts like code/config, not conversation. Every word is a constraint. Prefer explicit structure over “clever wording”.

- **Output-first design**  
  Design the **output format + schema first**, then write instructions that produce exactly that. If post-processing is painful, your prompt is wrong.

- **Examples > instructions**  
  Models pattern-match. A few high-quality examples usually beat long instruction blocks for style, formatting, and task nuances.

- **Decompose tasks**  
  For production: split big tasks into smaller, checkable steps (classify → extract → generate). Use multiple cheap calls over one giant “do everything” prompt.

- **Defensive prompting**  
  Always define: what to do on missing/ambiguous data, how to report uncertainty, and what is forbidden. Force **“don’t know”** paths instead of guessing.

---

## 2. High-Impact Prompt Patterns (50%)

### Pattern: Role + Task + Rules + Format

- **When to use**

  - Any production prompt
  - Need predictable behavior and easy debugging
  - Model used by multiple features/teams

- **Prompt template**

```text
You are a [ROLE] with expertise in [DOMAIN].

Your task:
[1–2 sentences, imperative verb: classify / extract / generate / rewrite / compare]

Input:
[Describe exact input the model receives]

Rules:
- [Hard constraint 1]
- [Hard constraint 2]
- [Failure behavior: when to say “unknown” or return null]
- Do not [forbidden behavior].

Output:
- Format: [JSON | markdown | plain text]
- No extra commentary unless specified.

Now process this input:
[INPUT_BLOCK]
```

- **Example**

```text
You are a senior support triage assistant for a B2B SaaS.

Your task:
Classify the ticket’s intent and extract key fields.

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
"Our invoices page times out every time I try to download the PDF. We’re closing our books today, please help ASAP."
```

- **How to evaluate**
  - JSON parses 100% of the time
  - Intent matches human label on eval set (e.g. 50 tickets) ≥ 90%
  - Priority not “low” on any clearly urgent sample
  - No hallucinated product_area values

---

### Pattern: Structured Output (Strict JSON)

- **When to use**

  - Downstream code parses responses
  - ETL / enrichment / extraction
  - Agents/tools calling other systems

- **Prompt template**

```text
You are a data extraction engine.

Task:
Extract the following fields from the text:

- [FieldName1] ([type], [rules])
- [FieldName2] ([type], [rules])
- ...

Rules:
- Output JSON only.
- Do not include any explanation.
- Use null if a value is missing or cannot be determined.
- Do not add extra keys.
- Strings must not contain line breaks.

JSON schema:
{
  "field1": <type/description>,
  "field2": <type/description>,
  "field3": <type/description>
}

Text:
[RAW_TEXT]
```

- **Example**

```text
You are a data extraction engine.

Task:
Extract the following fields from the text:
- company_name (string)
- revenue_usd (number, in USD)
- year (integer, YYYY)

Rules:
- Output JSON only.
- Use null if missing.
- No extra text.

JSON schema:
{
  "company_name": "string or null",
  "revenue_usd": "number or null (no currency symbols)",
  "year": "integer or null"
}

Text:
"Acme Corp reported revenue of $12.4M in 2023."
```

- **How to evaluate**
  - Parsing: JSON.parse succeeds on 100% of responses
  - Schema: keys exactly match schema; no extras
  - Numeric normalization correct (12.4M → 12400000)
  - On a labeled subset: ≥95% correct extraction

---

### Pattern: Few-Shot Style & Quality Alignment

- **When to use**

  - You care about **tone/voice** (marketing, docs, emails)
  - Want to avoid “LLM generic essay” style
  - Need consistency across many calls

- **Prompt template**

```text
You are a [ROLE].

Follow the writing style shown in the examples below.

Style rules:
- [Rule 1: e.g., “short sentences, no fluff”]
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

- **Example**

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

The server does not store client state between requests. That’s why REST is called stateless. Each request contains everything needed: authentication, parameters, and body.

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

- **How to evaluate**
  - Compare word count vs examples (±20%)
  - Check structure (same # of paragraphs, example presence)
  - Manual rating of style-match on 10 samples (aim ≥4/5)

---

### Pattern: Controlled Chain-of-Thought (Internal Trace)

- **When to use**

  - Complex reasoning: pricing, diagnosis, planning
  - Need debuggable **reasoning traces**
  - Agents/tools where wrong steps must be visible

- **Prompt template (trace visible to you, not end-user)**

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

- **Example**

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
- 12 users: “search is slow and incomplete”
- 5 users: “dark mode would be nice”
- 9 users: “mobile navigation is confusing”
```

- **How to evaluate**
  - Reasoning steps match the input facts
  - Final answer consistent with reasoning
  - For debugging: use logs to inspect <reasoning> on failed cases

---

### Pattern: Guardrailed Factual Answering (Low Hallucination)

- **When to use**

  - RAG, documentation Q&A, support bots
  - You must not answer beyond provided context
  - Compliance / regulated domains

- **Prompt template**

```text
You are a careful AI assistant that ONLY uses the provided context.

Task:
Answer the user’s question using ONLY the information in the Context block.

Rules:
- If the answer is not in the context, say:
  "I don’t know based on the provided context."
- Do not guess or add outside knowledge.
- Do not infer beyond what is explicitly stated.
- Quote short phrases from the context where useful.

Context:
[DOCUMENT_SNIPPETS_OR_RETRIEVED_CONTEXT]

Question:
[USER_QUESTION]

Answer:
```

- **Example**

```text
You are a careful AI assistant that ONLY uses the provided context.

Task:
Answer the user’s question using ONLY the information in the Context block.

Rules:
- If the answer is not in the context, say:
  "I don’t know based on the provided context."
- Do not guess or add outside knowledge.
- Do not infer beyond what is explicitly stated.

Context:
1) "The Pro plan includes up to 10 team members and 1 TB of storage."
2) "The Enterprise plan includes SSO, custom roles, and a dedicated account manager."

Question:
Does the Pro plan include SSO?

Answer:
```

- **How to evaluate**
  - On a test set with “unanswerable” questions, the model must say “I don’t know…” ≥ 95% of the time
  - No contradictions with source text on answerable ones
  - Check that answers never mention features not in context

---

### Pattern: Task Decomposition (Multi-Call)

- **When to use**

  - Complex flows that are brittle in one call
  - Need better control, caching, and cost reduction
  - Orchestrating agents/tools

- **High-level pattern**

1. Call 1: classify / understand / plan
2. Call 2: execute per step
3. Optional call: review / refine

- **Prompt template (Step 1: Planner)**

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

Then you feed each step into a simpler prompt specialized for that step (classification / rewriting / summarization, etc.).

- **How to evaluate**
  - Steps are executable without extra info
  - Pipeline success rate vs one-shot prompt (measure on test set)
  - Cost: sum of tokens across steps vs monolithic version

---

### Pattern: Defensive Error Reporting

- **When to use**

  - Any integration with code, workflows, or APIs
  - You want predictable failures, not “pretty lies”

- **Prompt template**

```text
You are a strict validation engine.

Task:
Validate the input data and either:
- return a normalized result, or
- return a structured error.

Rules:
- Never guess missing values.
- If required data is missing or inconsistent, return an error object.
- Use one of these error codes: ["MISSING_FIELD", "INVALID_FORMAT", "AMBIGUOUS"].

Output (JSON ONLY):

Success:
{
  "ok": true,
  "data": { ...normalized fields... }
}

Error:
{
  "ok": false,
  "error_code": "MISSING_FIELD|INVALID_FORMAT|AMBIGUOUS",
  "message": "short explanation",
  "field": "field_name_or_null"
}

Input:
[RAW_INPUT]
```

- **How to evaluate**
  - Downstream code never sees malformed JSON
  - Random fuzzed inputs → errors instead of hallucinated “valid” data
  - Manual review: correct error_code on diverse bad inputs

---

## 3. Hands-On Exercises (20%)

### Exercise 1: Rewrite a Flaky Prompt into Production-Grade

- **Exercise**  
  Start from this bad prompt:

  ```text
  Can you summarize this support ticket and tell me what to do?
  ```

  1. Rewrite it using **Role + Task + Rules + Format**
  2. Add JSON output schema
  3. Add explicit failure behavior

- **Input**

  ```text
  "The billing page charges us twice when we change our card. It’s been happening for 3 months. Please refund and fix this."
  ```

- **Expected Output (shape, not exact text)**

```json
{
  "summary": "Customer is charged twice when changing card on billing page.",
  "intent": "bug",
  "recommended_action": "Escalate to billing engineering and issue refund for 3 months of duplicate charges.",
  "priority": "high",
  "needs_refund": true
}
```

- **Evaluation Criteria**
  - JSON parses correctly
  - Fields populated logically
  - “priority” is “high” here
  - No extra keys, no narrative text outside JSON

---

### Exercise 2: Few-Shot Style Alignment for Release Notes

- **Exercise**  
  Craft a few-shot prompt that converts internal changelog entries into friendly release notes.

- **Input examples**

  ```text
  Input:
  - Fixed NPE in invoice generator when tax_id is null
  - Added CSV export for invoices
  - Improved query performance on /invoices endpoint

  Output:
  [Your styled release note]
  ```

- **Target Style (expected characteristics)**

  - Non-technical wording
  - Bulleted list
  - Each item starts with a verb
  - No stack traces or internal jargon

- **Expected Output Example**

```text
- Fixed an issue that could cause invoice creation to fail for some customers.
- Added CSV export so you can download your invoices in bulk.
- Improved invoice loading speed for large accounts.
```

- **Evaluation Criteria**
  - No raw technical terms (e.g., “NPE”, “endpoint”)
  - All bullets user-facing benefits, not implementation details
  - Consistent tone across multiple runs

---

### Exercise 3: Guardrailed Q&A on Docs

- **Exercise**  
  Build a prompt that answers only using this context:

  ```text
  - The Free plan includes up to 3 projects.
  - The Pro plan includes unlimited projects and priority support.
  - Custom integrations are only available on Enterprise.
  ```

- **Input**

  Question 1:  
  “Does the Pro plan include custom integrations?”

  Question 2:  
  “How many projects can I create on the Free plan?”

- **Expected Outputs**

  - For Q1:  
    `I don’t know based on the provided context.` (or explicit “not mentioned”)
  - For Q2:  
    `The Free plan includes up to 3 projects.`

- **Evaluation Criteria**
  - Never claims that Pro includes custom integrations
  - Answers Q2 exactly from context
  - Behavior stays correct across 10 similar questions

---

### Exercise 4: Add Defensive Error Handling to an Extractor

- **Exercise**  
  You have this prompt:

  ```text
  Extract the due_date from the invoice email and return it as YYYY-MM-DD.
  ```

  1. Turn it into a strict JSON extractor with error codes.
  2. Test on:
     - “Payment is due next Friday.”
     - “Invoice due date: 2025-01-15.”

- **Expected Outputs**

  - Case 1 (ambiguous): error
  - Case 2: success

```json
// Case 1
{
  "ok": false,
  "error_code": "AMBIGUOUS",
  "message": "Due date specified as relative time, cannot map to exact date.",
  "field": "due_date"
}

// Case 2
{
  "ok": true,
  "data": {
    "due_date": "2025-01-15"
  }
}
```

- **Evaluation Criteria**
  - No guessing of specific dates from “next Friday”
  - Correct classification as error vs success
  - JSON shape exactly follows your schema

---

## 4. Debugging & Optimization (10%)

### Common Failures

- **Hallucinations / “confident nonsense”**

  - Model answers beyond context
  - Invented fields in JSON
  - Names or IDs not in source

- **Format drift**

  - Extra prose around JSON
  - Missing keys / extra keys
  - Inconsistent types (string vs number)

- **Style drift**

  - Gets more verbose over time
  - Ignores tone constraints
  - Switches from bullets to paragraphs

- **Hidden ambiguity**

  - Prompt doesn’t say what to do when info is missing
  - Multiple valid interpretations of task
  - Model flips behavior run-to-run

- **Cost bloat**
  - Copy-pasted giant instructions into every call
  - Too many examples
  - Overuse of large models where small would work

### Fix Strategies

- **For hallucinations**

  - Add explicit rules:  
    “If information is not present, say ‘I don’t know’.”  
    “Use only facts from the Context block.”
  - Lower temperature for factual tasks
  - Use RAG + guardrail prompt instead of open-web knowledge

- **For format drift**

  - Add: “Output JSON only. No other text.”
  - Show 1–2 **valid output examples**
  - Wrap response expectations in clear markers:  
    “Respond with a single JSON object that matches this schema: …”

- **For style drift**

  - Add more/better examples instead of more prose
  - Add max length constraints: “<= 150 words”
  - Add hard rules: “No introductions or conclusions.”

- **For ambiguity**

  - Enumerate edge cases explicitly: missing fields, conflicting info, multiple items
  - Add defensive branches: “If X and Y both appear, prefer X”
  - Use a first call to classify/clarify, second to answer

- **For cost**
  - Remove repeated global instructions; move them to system prompt
  - Cut examples to the minimum that still preserves quality
  - Use smaller models for simple classification/rewrites
  - Cache long static context and reuse

---

## 5. Real-World Use Cases (10%)

### SaaS

- **Support triage**

  - Pattern: Structured extraction (intent, product, priority)
  - Add defensive errors for missing product, ambiguous intent
  - Connect to ticket routing rules

- **Product analytics enrichment**

  - Pattern: Batch classification of feedback into themes
  - Use few-shot examples for each theme
  - Export JSON and join with event data

- **Docs copilots**
  - Pattern: Guardrailed Q&A over product docs
  - RAG + “context-only” answer prompt
  - Log failures to improve retrieval or docs

### Automation

- **Email / Slack routing**

  - Extract owner, due date, action type
  - Defensive prompt: errors for ambiguous or missing deadline
  - Drive workflow engines (e.g., create tasks, labels)

- **Data cleaning & normalization**

  - Address / company / category normalization
  - Strict JSON outputs plus error codes
  - Use smaller models at high volume

- **Report summarization**
  - Pattern: Decompose into “extract metrics” → “generate summary”
  - First call: structured metrics JSON
  - Second call: narrative explanation from metrics only

### Agents

- **Tool-calling agents**

  - Pattern: output JSON with `tool_name`, `arguments`, `thought`
  - Enforce that arguments are serializable and schema-valid
  - Add explicit instructions: “If no tool is appropriate, respond with tool_name=null.”

- **Debuggable reasoning**

  - Use `<reasoning>` / `<answer>` separation
  - Store reasoning in logs for failed runs
  - Iterate prompts focusing on the **step where reasoning goes wrong**

- **Guardrails as separate prompts**
  - Pre-check user input (safety, scope)
  - Post-check agent output (policy, hallucination risk)
  - Use small models + strict, few-shot guardrail prompts

---

## 6. Curated Resources

### Must-read

- **Learn Prompting** – Practical guides on roles, few-shot, and techniques.
- **Prompting Guide (CoT, RAG, etc.)** – Concise technique recipes.
- **Claude / OpenAI / AWS prompt best-practices docs** – Up-to-date, model-specific tips.

### Tools

- **Prompt management / eval**

  - Langfuse, LangSmith, Humanloop, Maxim AI, PromptLayer
  - Use them for: versioning prompts, A/B tests, logging, evaluation metrics.

- **Guardrails / safety**

  - Dedicated guardrail libraries (moderation, PII, policy checks)
  - Combine prompt-based guardrails + external classifiers.

- **RAG / orchestration**
  - LlamaIndex, LangChain, semantic-kernel style frameworks
  - Use to separate retrieval from generation & centralize prompts.

### Repos / Patterns

- Open-source repos that show:
  - JSON-schema-based structured output with automatic parsing
  - Evaluation harnesses (accuracy, consistency, cost)
  - Real RAG templates with injection-safe prompts and guardrails

---

## Practical Checklist (for Every Production Prompt)

- **Clarity**

  - [ ] Role is explicit and relevant
  - [ ] Task is a single, clear action
  - [ ] No vague verbs (“help”, “discuss”) without specifics

- **Constraints**

  - [ ] Output format defined (JSON/markdown/plain)
  - [ ] Length constraints where needed
  - [ ] Failure behavior defined (null, error object, or “don’t know”)

- **Structure**

  - [ ] For structured use: schema included in prompt
  - [ ] No extraneous keys expected by downstream code
  - [ ] Examples (if used) mirror exact desired output

- **Safety & Reliability**

  - [ ] Factual tasks include “do not guess” rule
  - [ ] Context-only tasks explicitly forbid outside knowledge
  - [ ] Error conditions are explicit

- **Cost**

  - [ ] No duplicated global instructions across prompts
  - [ ] Examples count minimized while preserving quality
  - [ ] Model size appropriate for task (use small where possible)

- **Evaluation**
  - [ ] You have a small labeled eval set (20–100 samples)
  - [ ] You measure: format validity, accuracy, and cost
  - [ ] You log failures and update prompts based on real traces

If you apply these patterns, templates, and the checklist, you can iteratively tune prompts like code: controlled, testable, and production-grade.

[1](https://orq.ai/blog/what-is-the-best-way-to-think-of-prompt-engineering)
[2](https://latitude-blog.ghost.io/blog/5-ways-to-optimize-llm-prompts-for-production-environments/)
[3](https://docs.promptlayer.com/features/prompt-registry/structured-outputs)
[4](https://www.claude.com/blog/best-practices-for-prompt-engineering)
[5](https://arxiv.org/html/2404.01077v1)
[6](https://docs.aws.amazon.com/nova/latest/userguide/prompting-structured-output.html)
[7](https://www.promptmixer.dev/blog/7-best-practices-for-ai-prompt-engineering-in-2025)
[8](https://orq.ai/blog/prompt-optimization)
[9](https://devblogs.microsoft.com/semantic-kernel/using-json-schema-for-structured-output-in-net-for-openai-models/)
[10](https://claude.com/blog/best-practices-for-prompt-engineering)
[11](https://www.redhat.com/en/blog/when-llms-day-dream-hallucinations-how-prevent-them)
[12](https://blog.promptlayer.com/how-to-reduce-llm-costs/)
[13](https://www.augmentcode.com/guides/debugging-ai-generated-code-8-failure-patterns-and-fixes)
[14](https://huggingface.co/blog/Imama/pr)
[15](https://aws.amazon.com/blogs/machine-learning/effective-cost-optimization-strategies-for-amazon-bedrock/)
[16](https://www.datagrid.com/blog/11-tips-ai-agent-prompt-engineering)
[17](https://milvus.io/ai-quick-reference/how-can-prompt-engineering-help-mitigate-hallucinations-eg-telling-the-llm-if-the-information-is-not-in-the-provided-text-say-you-dont-know)
[18](https://www.reddit.com/r/PromptEngineering/comments/1i3b2qr/managing_prompt_costs_at_enterprise_scale/)
[19](https://docs.lovable.dev/prompting/prompting-debugging)
[20](https://www.prompthub.us/blog/three-prompt-engineering-methods-to-reduce-hallucinations)
[21](https://futureagi.com/blogs/top-prompt-management-platforms-2025)
[22](https://www.qed42.com/insights/building-simple-effective-prompt-based-guardrails)
[23](https://www.datacamp.com/tutorial/few-shot-prompting)
[24](https://mirascope.com/blog/prompt-engineering-tools)
[25](https://docs.aws.amazon.com/prescriptive-guidance/latest/llm-prompt-engineering-best-practices/enhanced-template.html)
[26](https://learnprompting.org/docs/basics/few_shot)
[27](https://www.getmaxim.ai/articles/top-5-prompt-management-platforms-in-2025/)
[28](https://www.reddit.com/r/aws/comments/1lnbfh7/prompt_engineering_vs_guardrails/)
[29](https://www.promptpanda.io/resources/few-shot-prompting-explained-a-guide/)
[30](https://latitude-blog.ghost.io/blog/top-7-open-source-tools-for-prompt-engineering-in-2025/)
[31](https://promptengineering.org/system-prompts-in-large-language-models/)
[32](https://openreview.net/pdf?id=_VjQlMeSB_J)
[33](https://www.linkedin.com/posts/sakarwalparas_ai-aiagents-promptengineering-activity-7385646201666355200-CbF5)
[34](https://www.prompthub.us/blog/role-prompting-does-adding-personas-to-your-prompts-really-make-a-difference)
[35](https://arxiv.org/abs/2201.11903)
[36](https://learnprompting.org/docs/basics/roles)
[37](https://www.promptingguide.ai/techniques/cot)
[38](https://rediminds.com/future-edge/how-meta-prompting-and-role-engineering-are-unlocking-the-next-generation-of-ai-agents/)
[39](https://learnprompting.org/docs/advanced/zero_shot/role_prompting)
