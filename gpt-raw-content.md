# Practical Prompt Engineering Playbook

## 1\. Core Mental Models (10%)

- _LLMs require explicit context and precision._ Provide detailed background and specifics (e.g. language, frameworks, example data) so the model doesn't have to guess[\[1\]](https://addyo.substack.com/p/the-prompt-engineering-playbook-for#:~:text=,the%20AI%20has%20to%20guess)[\[2\]](https://latitude-blog.ghost.io/blog/10-best-practices-for-production-grade-llm-prompt-engineering/#:~:text=,with%20your%20audience%20and%20purpose). More context yields more accurate output.
- _Treat the prompt like a function or spec._ Think of prompts as reusable templates (with static instructions + dynamic inputs)[\[3\]](https://kinde.com/learn/ai-for-software-engineering/prompting/prompt-patterns-that-scale-reusable-llm-prompts-for-dev-eams/#:~:text=Scalable%20prompt%20patterns%20are%20structured%2C,and%20produce%20consistent%2C%20predictable%20outputs)[\[4\]](https://kinde.com/learn/ai-for-software-engineering/prompting/prompt-patterns-that-scale-reusable-llm-prompts-for-dev-eams/#:~:text=,or%20style%20of%20its%20response). Explicitly define role, task, steps, and format as parameters.
- _Use examples/few-shot to illustrate requirements._ Showing even one concrete example of input→output significantly guides the model[\[5\]](https://addyo.substack.com/p/the-prompt-engineering-playbook-for#:~:text=,guide%20the%20model%E2%80%99s%20response%20significantly). Few-shot demonstrations reduce ambiguity in tone, format, and content.
- _Set a persona or role to align style._ Instruct the model to "act as" an expert or specific persona (e.g. "You are a senior software engineer") to get domain-specific knowledge and tone[\[6\]](https://addyo.substack.com/p/the-prompt-engineering-playbook-for#:~:text=,In%20our%20own%20usage%2C%20we).
- _Decompose complex tasks into steps._ For multi-stage tasks, either break into sequential prompts or enumerate steps. This improves consistency and reasoning[\[7\]](https://addyo.substack.com/p/the-prompt-engineering-playbook-for#:~:text=,would%20incrementally%20build%20a%20solution)[\[8\]](https://launchdarkly.com/blog/prompt-engineering-best-practices/#:~:text=If%20you%20have%20a%20complex,the%20steps%20in%20the%20prompt).
- _Specify output schema and constraints clearly._ Demand a precise format (JSON, bullet points, etc.) and list rules (like length, tone) so the model's output can be reliably parsed[\[9\]](https://launchdarkly.com/blog/prompt-engineering-best-practices/#:~:text=Clearly%20specify%20the%20type%20of,into%20another%20model%20or%20process)[\[10\]](https://launchdarkly.com/blog/prompt-engineering-best-practices/#:~:text=Use%20formatting%20to%20make%20critical,constraints%20that%20cannot%20be%20ignored).
- _Iterate and refine prompts like code._ Test prompts on examples, collect metrics, version-control them, and adjust based on errors. Treat prompt engineering as an ongoing cycle of testing and improvement[\[11\]](https://kinde.com/learn/ai-for-software-engineering/prompting/prompt-patterns-that-scale-reusable-llm-prompts-for-dev-eams/#:~:text=Best%20Practice%20Description%20Centralize%20Your,created%20or%20last%20updated%20it)[\[12\]](https://addyo.substack.com/p/the-prompt-engineering-playbook-for#:~:text=,on%20the%20first%20try).
- _Balance clarity with brevity._ Include essential details and constraints, but remove unnecessary words to save tokens and avoid confusion[\[13\]](https://launchdarkly.com/blog/prompt-engineering-best-practices/#:~:text=).
- _Plan for safety and hallucinations._ Cleanse inputs to prevent injections[\[14\]](https://launchdarkly.com/blog/prompt-engineering-best-practices/#:~:text=input%20validation%20invites%20prompt%20injection,commands%20that%20override%20original%20instructions)[\[15\]](https://launchdarkly.com/blog/prompt-engineering-best-practices/#:~:text=AI%20systems%20are%20targets%20for,model%20to%20ignore%20embedded%20instructions), explicitly instruct the model to "stick to facts" or "only use provided info," and implement guardrails to catch bad outputs.

## 2\. High-Impact Prompt Patterns (50%)

### Pattern: Role/Persona Prompting

- **When to use:** When you need domain expertise, specific knowledge or a consistent tone (technical, formal, humorous, etc.).
- **Prompt template:**
- You are a \[ROLE - e.g. senior software engineer, medical consultant, data analyst\].  
   \[Task or instruction here - be specific about what you want\]
- **Example:**
- You are a senior cybersecurity analyst. Analyze the following network log and identify any security risks.
- **How to evaluate:** Check that the response reflects the persona's expertise and style (uses relevant terminology, shows structured reasoning). For instance, a "senior engineer" response should be detailed and accurate, not generic[\[6\]](https://addyo.substack.com/p/the-prompt-engineering-playbook-for#:~:text=,In%20our%20own%20usage%2C%20we)[\[16\]](https://launchdarkly.com/blog/prompt-engineering-best-practices/#:~:text=Specify%20the%20type%20of%20expert,Example).

### Pattern: Structured Output (Schema Prompting)

- **When to use:** When downstream code needs to parse the output, or you need consistent data (e.g. JSON, CSV).
- **Prompt template:**
- You are an assistant extracting information. Extract the following fields:  
   \- Field1 (type)  
   \- Field2 (type)  
   \[etc.\]  
   Output as JSON only (no extra text).
- **Example:**
- You are a product analyst assistant. Extract \`productId\` (string) and \`price\` (float) from the text below. Output JSON only.  
   <br/>Text: "Item A (ID: A123) costs \$19.99 and is currently in stock."
- **How to evaluate:** Parse the response as JSON. Verify that all required keys exist with correct types/format and no extra fields. For example, values should match expected types (numbers as numbers, dates in YYYY-MM-DD, etc.)[\[17\]](https://kinde.com/learn/ai-for-software-engineering/prompting/prompt-patterns-that-scale-reusable-llm-prompts-for-dev-eams/#:~:text=,reliably%20in%20your%20application%20code)[\[9\]](https://launchdarkly.com/blog/prompt-engineering-best-practices/#:~:text=Clearly%20specify%20the%20type%20of,into%20another%20model%20or%20process).

### Pattern: Few-Shot / Example Prompting

- **When to use:** When you want the model to follow a specific format, style, or reasoning pattern. Few-shot examples show _how_ to answer.
- **Prompt template:**
- Follow the format shown in these examples:  
   <br/>Example 1:  
   Input: \[Example input 1\]  
   Output: \[Example output 1\]  
   <br/>Example 2:  
   Input: \[Example input 2\]  
   Output: \[Example output 2\]  
   <br/>Now apply this to:  
   Input: \[New input\]
- **Example:**
- You are a technical writer. Use the style in the example below.  
   <br/>Example:  
   Input: "Explain REST APIs"  
   Output: "REST APIs are a stateless communication model built on HTTP..."  
   <br/>Now do the same for:  
   Input: "Explain GraphQL"
- **How to evaluate:** Check that the response matches the examples' style and format. For instance, verify similar structure (conciseness, tone, terminology). Ensure factual accuracy and no hallucinations. (One can use similarity metrics or manual checks against the example pattern[\[5\]](https://addyo.substack.com/p/the-prompt-engineering-playbook-for#:~:text=,guide%20the%20model%E2%80%99s%20response%20significantly).)

### Pattern: Chain-of-Thought Prompting

- **When to use:** For tasks requiring reasoning or multi-step logic (math, troubleshooting, planning).
- **Prompt template:**
- Question: \[Complex problem\]  
   Think step by step:
- **Example:**
- Question: "A jar has red, blue, and green marbles. There are 5 times as many blue as red, and 10 fewer green than blue. If there are 8 red marbles, how many marbles total?"  
   Think step by step:
- **How to evaluate:** Verify that the reasoning steps are logical and lead to a correct answer. The final answer must be correct. Ensure each step is coherent. (For hidden reasoning, you may ask the model to "explain your answer" afterward or trust self-consistency.) This often greatly improves correctness on complex questions[\[8\]](https://launchdarkly.com/blog/prompt-engineering-best-practices/#:~:text=If%20you%20have%20a%20complex,the%20steps%20in%20the%20prompt).

### Pattern: Guardrails & Constraints

- **When to use:** To enforce rules or filter undesirable content. Important for safety (no disallowed content) or strict formats.
- **Prompt template:**
- Rules:  
   \- \[Rule 1 - e.g. "Do not reveal personal data."\]  
   \- \[Rule 2 - e.g. "Only output in Spanish."\]  
   \- \[Rule 3 - e.g. "Keep answers under 50 words."\]  
   Task: \[Instruction for the model\]
- **Example:**
- You are a customer support assistant.  
   <br/>Rules:  
   \- Do not share any internal metrics or confidential data.  
   \- Use a friendly, professional tone.  
   \- Keep answers under 50 words.  
   \- If uncertain, say "I don't have that information."  
   <br/>Task: "How secure is our service?"
- **How to evaluate:** Check compliance with all rules. For example, ensure no forbidden info appears and the tone/length matches instructions. Test with edge cases (malicious or nonsensical inputs) to ensure the model does not violate guardrails[\[15\]](https://launchdarkly.com/blog/prompt-engineering-best-practices/#:~:text=AI%20systems%20are%20targets%20for,model%20to%20ignore%20embedded%20instructions).

### Pattern: Step-by-Step Instruction (Task Decomposition)

- **When to use:** For complex tasks that benefit from explicit procedures, or when you want to generate outputs stepwise (often across multiple prompts).
- **Prompt template:**
- Follow these steps:  
   1\. \[First step instruction\]  
   2\. \[Second step\]  
   ...  
   n. \[Final step\]
- **Example:**
- You are a data scientist assistant. Analyze the sales report:  
   <br/>Steps:  
   1\. Summarize overall sales figures.  
   2\. Identify the top 3 performing products.  
   3\. Note any significant trends.  
   Output: A brief report summarizing these points.
- **How to evaluate:** Ensure each step is addressed in the answer. The final output should integrate all steps. For example, verify that the summary lists overall sales, top products, and trends. This approach often yields more complete and accurate responses[\[7\]](https://addyo.substack.com/p/the-prompt-engineering-playbook-for#:~:text=,would%20incrementally%20build%20a%20solution)[\[8\]](https://launchdarkly.com/blog/prompt-engineering-best-practices/#:~:text=If%20you%20have%20a%20complex,the%20steps%20in%20the%20prompt).

## 3\. Hands-On Exercises (20%)

- **Exercise:** _Structured Data Extraction_.  
   **Input:** "Acme Corp had 150 employees in 2023, up from 120 in 2022."  
   **Expected Output:** JSON like {"Company":"Acme Corp","Employees2022":120,"Employees2023":150}.  
   **Evaluation Criteria:** Parse the JSON to ensure keys Company, Employees2022, Employees2023 exist with correct types and values. Check no extra fields; values match the input.
- **Exercise:** _Style/Tone Matching_.  
   **Input:** "Describe the benefits of cloud computing."  
   **Expected Output:** A concise explanation in a formal technical tone (2-3 sentences).  
   **Evaluation Criteria:** The answer should correctly define benefits and use a formal tone. Compare style and length against an example of formal writing; check for clarity and absence of casual language.
- **Exercise:** _Polishing Text_.  
   **Input:** "heLLo, plz write this program in PYTHON."  
   **Expected Output:** A cleaned-up request, e.g. "Hello, please write this program in Python.".  
   **Evaluation Criteria:** The output should correct spelling/capitalization ("heLLo" → "Hello", "plz" → "please") and formatting. It should preserve intent.
- **Exercise:** _Debugging Prompt_.  
   **Input:** A code snippet that sums a list but has an off-by-one error.  
   **Expected Output:** A description identifying the bug (e.g. "The loop stops one iteration early") and a fix suggestion.  
   **Evaluation Criteria:** Ensure the model pinpoints the real bug and the fix is accurate. The answer should refer to the specific error (no generic "check your loop").
- **Exercise:** _Classification with Constraints_.  
   **Input:** "Classify the sentiment of: 'I absolutely love this update!'"  
   **Expected Output:** "Positive."  
   **Evaluation Criteria:** The label must be correct (Positive/Negative/Neutral) with no extra commentary. Verify it matches ground-truth sentiment.

## 4\. Debugging & Optimization (10%)

- **Common failures:**
- **Vague prompts:** Missing details lead to incomplete or incorrect answers. The model may hallucinate facts or be irrelevant[\[18\]](https://launchdarkly.com/blog/prompt-engineering-best-practices/#:~:text=Insufficient%20context%20in%20prompts%20leads,and%20damaging%20the%20brand%27s%20reputation).
- **Format errors:** Without explicit instructions, output may not match the desired schema (e.g. free text instead of JSON)[\[9\]](https://launchdarkly.com/blog/prompt-engineering-best-practices/#:~:text=Clearly%20specify%20the%20type%20of,into%20another%20model%20or%20process).
- **Rule violations:** If constraints are unclear, the model might ignore them (e.g. disallowed content appears, wrong language).
- **Excess verbosity:** Overly long answers waste tokens and may exceed limits.
- **Prompt drift:** Small, uncontrolled changes in prompts cause inconsistent outputs ("it worked yesterday" problem)[\[19\]](https://kinde.com/learn/ai-for-software-engineering/prompting/prompt-patterns-that-scale-reusable-llm-prompts-for-dev-eams/#:~:text=One%20common%20misconception%20is%20that,inconsistent%20outputs%20and%20degraded%20performance).
- **Fix strategies:**
- **Make instructions explicit:** Clarify ambiguous parts, add examples, and restate the task. Explicitly demand formats and rules[\[17\]](https://kinde.com/learn/ai-for-software-engineering/prompting/prompt-patterns-that-scale-reusable-llm-prompts-for-dev-eams/#:~:text=,reliably%20in%20your%20application%20code)[\[9\]](https://launchdarkly.com/blog/prompt-engineering-best-practices/#:~:text=Clearly%20specify%20the%20type%20of,into%20another%20model%20or%20process).
- **Add constraints/roles:** Tighten prompts with roles or guardrails (e.g. "Assistant: A helpful technical writer") and strict rules ("only JSON")[\[6\]](https://addyo.substack.com/p/the-prompt-engineering-playbook-for#:~:text=,In%20our%20own%20usage%2C%20we)[\[9\]](https://launchdarkly.com/blog/prompt-engineering-best-practices/#:~:text=Clearly%20specify%20the%20type%20of,into%20another%20model%20or%20process).
- **Decompose the task:** Split complex prompts into smaller steps or multi-turn dialogs[\[7\]](https://addyo.substack.com/p/the-prompt-engineering-playbook-for#:~:text=,would%20incrementally%20build%20a%20solution). Verify each part before proceeding.
- **Sanitize and verify:** Clean inputs to remove injections, and use follow-up prompts to check outputs (or LLM-based evaluations). Explicitly tell the model to ignore malicious content[\[15\]](https://launchdarkly.com/blog/prompt-engineering-best-practices/#:~:text=AI%20systems%20are%20targets%20for,model%20to%20ignore%20embedded%20instructions).
- **Tune settings:** Adjust temperature (lower for factual tasks, higher for creative) and token limits.
- **Iterative testing:** Run the prompt on varied test cases, use evaluation metrics (similarity scores, correct-data checks) and track versions to identify regressions[\[11\]](https://kinde.com/learn/ai-for-software-engineering/prompting/prompt-patterns-that-scale-reusable-llm-prompts-for-dev-eams/#:~:text=Best%20Practice%20Description%20Centralize%20Your,created%20or%20last%20updated%20it).

## 5\. Real-World Use Cases (10%)

- **SaaS features:** Prompt engineering powers AI features in software products. For example, customer feedback summarization, automated report generation, or smart autocomplete rely on well-crafted prompts. A content management system might use the _Structured Output_ pattern to extract metadata from text, or _Style prompts_ for generating user-facing copy. Consistent prompt patterns ensure scalable features across a product[\[20\]](https://kinde.com/learn/ai-for-software-engineering/prompting/prompt-patterns-that-scale-reusable-llm-prompts-for-dev-eams/#:~:text=For%20product%20features%3A).
- **Automation & Workflows:** Teams automate tasks like generating test data, commit messages, or documentation with prompts[\[21\]](https://kinde.com/learn/ai-for-software-engineering/prompting/prompt-patterns-that-scale-reusable-llm-prompts-for-dev-eams/#:~:text=,for%20new%20functions%20or%20components). In CI/CD pipelines, a prompt can review code changes or write release notes. Automated data pipelines use prompts to classify tickets or extract metrics. For instance, a prompt could parse logs into structured incident reports.
- **Agents & Autonomous Assistants:** In agentic systems (auto-coders, schedulers), prompts define roles, goals, and multi-step actions. An AI agent may use chain-of-thought internally to plan, then execute steps via structured prompts. Guardrail patterns are critical here: agents are explicitly instructed what _not_ to do (e.g. "Do not execute commands unless confirmed")[\[15\]](https://launchdarkly.com/blog/prompt-engineering-best-practices/#:~:text=AI%20systems%20are%20targets%20for,model%20to%20ignore%20embedded%20instructions). Prompt engineering in agents also involves managing memory/context and handling fallbacks when instructions are unclear.

## 6\. Curated Resources

- **Must-read:** Key practical guides and blogs: _"10 Best Practices for Production-Grade LLM Prompt Engineering"_[\[2\]](https://latitude-blog.ghost.io/blog/10-best-practices-for-production-grade-llm-prompt-engineering/#:~:text=,with%20your%20audience%20and%20purpose) (Latitude blog), _"Prompt Engineering Playbook for Programmers"_[\[22\]](https://addyo.substack.com/p/the-prompt-engineering-playbook-for#:~:text=even%20generate%20entire%20modules%20or,at%20how%20to%20systematically%20craft) (Addy Osmani), _"Reusable LLM Prompts for Dev Teams"_[\[3\]](https://kinde.com/learn/ai-for-software-engineering/prompting/prompt-patterns-that-scale-reusable-llm-prompts-for-dev-eams/#:~:text=Scalable%20prompt%20patterns%20are%20structured%2C,and%20produce%20consistent%2C%20predictable%20outputs) (Kinde), and _"Prompt Engineering Best Practices"_[\[9\]](https://launchdarkly.com/blog/prompt-engineering-best-practices/#:~:text=Clearly%20specify%20the%20type%20of,into%20another%20model%20or%20process) (LaunchDarkly). These provide actionable tips and templates.
- **Tools:** Interactive and management tools: **OpenAI Playground** (for live testing), **LangChain** (framework for building multi-step prompt workflows), **PromptLayer** (logging & version control for prompts), and evaluation suites like **DeepEval** or **OpenAI Evals** for automated testing of outputs.
- **Repos:** Collections of templates and examples: \[promptslab/Awesome-Prompt-Engineering\] (curated list of guides and tools)[\[23\]](https://github.com/promptslab/Awesome-Prompt-Engineering#:~:text=Image), \[dair-ai/Prompt-Engineering-Guide\] (tutorials and papers)[\[24\]](https://github.com/dair-ai/Prompt-Engineering-Guide#:~:text=Prompt%20engineering%20is%20a%20relatively,with%20LLMs%20and%20other%20tools), OpenAI's _Prompt Engineering Cookbook_ examples, and community demo repos (e.g. LangChain examples). These contain ready-to-use prompt patterns and exercises.

[\[1\]](https://addyo.substack.com/p/the-prompt-engineering-playbook-for#:~:text=,the%20AI%20has%20to%20guess) [\[5\]](https://addyo.substack.com/p/the-prompt-engineering-playbook-for#:~:text=,guide%20the%20model%E2%80%99s%20response%20significantly) [\[6\]](https://addyo.substack.com/p/the-prompt-engineering-playbook-for#:~:text=,In%20our%20own%20usage%2C%20we) [\[7\]](https://addyo.substack.com/p/the-prompt-engineering-playbook-for#:~:text=,would%20incrementally%20build%20a%20solution) [\[12\]](https://addyo.substack.com/p/the-prompt-engineering-playbook-for#:~:text=,on%20the%20first%20try) [\[22\]](https://addyo.substack.com/p/the-prompt-engineering-playbook-for#:~:text=even%20generate%20entire%20modules%20or,at%20how%20to%20systematically%20craft) The Prompt Engineering Playbook for Programmers

<https://addyo.substack.com/p/the-prompt-engineering-playbook-for>

[\[2\]](https://latitude-blog.ghost.io/blog/10-best-practices-for-production-grade-llm-prompt-engineering/#:~:text=,with%20your%20audience%20and%20purpose) 10 Best Practices for Production-Grade LLM Prompt Engineering

<https://latitude-blog.ghost.io/blog/10-best-practices-for-production-grade-llm-prompt-engineering/>

[\[3\]](https://kinde.com/learn/ai-for-software-engineering/prompting/prompt-patterns-that-scale-reusable-llm-prompts-for-dev-eams/#:~:text=Scalable%20prompt%20patterns%20are%20structured%2C,and%20produce%20consistent%2C%20predictable%20outputs) [\[4\]](https://kinde.com/learn/ai-for-software-engineering/prompting/prompt-patterns-that-scale-reusable-llm-prompts-for-dev-eams/#:~:text=,or%20style%20of%20its%20response) [\[11\]](https://kinde.com/learn/ai-for-software-engineering/prompting/prompt-patterns-that-scale-reusable-llm-prompts-for-dev-eams/#:~:text=Best%20Practice%20Description%20Centralize%20Your,created%20or%20last%20updated%20it) [\[17\]](https://kinde.com/learn/ai-for-software-engineering/prompting/prompt-patterns-that-scale-reusable-llm-prompts-for-dev-eams/#:~:text=,reliably%20in%20your%20application%20code) [\[19\]](https://kinde.com/learn/ai-for-software-engineering/prompting/prompt-patterns-that-scale-reusable-llm-prompts-for-dev-eams/#:~:text=One%20common%20misconception%20is%20that,inconsistent%20outputs%20and%20degraded%20performance) [\[20\]](https://kinde.com/learn/ai-for-software-engineering/prompting/prompt-patterns-that-scale-reusable-llm-prompts-for-dev-eams/#:~:text=For%20product%20features%3A) [\[21\]](https://kinde.com/learn/ai-for-software-engineering/prompting/prompt-patterns-that-scale-reusable-llm-prompts-for-dev-eams/#:~:text=,for%20new%20functions%20or%20components) Kinde Prompt Patterns That Scale Reusable LLM Prompts for Dev Teams

<https://kinde.com/learn/ai-for-software-engineering/prompting/prompt-patterns-that-scale-reusable-llm-prompts-for-dev-eams/>

[\[8\]](https://launchdarkly.com/blog/prompt-engineering-best-practices/#:~:text=If%20you%20have%20a%20complex,the%20steps%20in%20the%20prompt) [\[9\]](https://launchdarkly.com/blog/prompt-engineering-best-practices/#:~:text=Clearly%20specify%20the%20type%20of,into%20another%20model%20or%20process) [\[10\]](https://launchdarkly.com/blog/prompt-engineering-best-practices/#:~:text=Use%20formatting%20to%20make%20critical,constraints%20that%20cannot%20be%20ignored) [\[13\]](https://launchdarkly.com/blog/prompt-engineering-best-practices/#:~:text=) [\[14\]](https://launchdarkly.com/blog/prompt-engineering-best-practices/#:~:text=input%20validation%20invites%20prompt%20injection,commands%20that%20override%20original%20instructions) [\[15\]](https://launchdarkly.com/blog/prompt-engineering-best-practices/#:~:text=AI%20systems%20are%20targets%20for,model%20to%20ignore%20embedded%20instructions) [\[16\]](https://launchdarkly.com/blog/prompt-engineering-best-practices/#:~:text=Specify%20the%20type%20of%20expert,Example) [\[18\]](https://launchdarkly.com/blog/prompt-engineering-best-practices/#:~:text=Insufficient%20context%20in%20prompts%20leads,and%20damaging%20the%20brand%27s%20reputation) Prompt Engineering Best Practices: Tutorial & Examples | LaunchDarkly

<https://launchdarkly.com/blog/prompt-engineering-best-practices/>

[\[23\]](https://github.com/promptslab/Awesome-Prompt-Engineering#:~:text=Image) GitHub - promptslab/Awesome-Prompt-Engineering: This repository contains a hand-curated resources for Prompt Engineering with a focus on Generative Pre-trained Transformer (GPT), ChatGPT, PaLM etc

<https://github.com/promptslab/Awesome-Prompt-Engineering>

[\[24\]](https://github.com/dair-ai/Prompt-Engineering-Guide#:~:text=Prompt%20engineering%20is%20a%20relatively,with%20LLMs%20and%20other%20tools) GitHub - dair-ai/Prompt-Engineering-Guide: Guides, papers, lessons, notebooks and resources for prompt engineering, context engineering, RAG, and AI Agents.

<https://github.com/dair-ai/Prompt-Engineering-Guide>
