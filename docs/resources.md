# Curated Resources

Tools, libraries, learning materials, and communities for production prompt engineering.

## Table of Contents

- [Learning Resources](#learning-resources)
- [Tools & Platforms](#tools--platforms)
- [Libraries & Frameworks](#libraries--frameworks)
- [Repositories & Examples](#repositories--examples)
- [Provider Documentation](#provider-documentation)

---

## Learning Resources

### Essential Reading

#### Learn Prompting
- **URL:** [learnprompting.org](https://learnprompting.org/)
- **Best for:** Foundational concepts, few-shot learning, chain-of-thought
- **Focus:** Practical guides on roles, examples, and core techniques
- **Key sections:**
  - [Role prompting](https://learnprompting.org/docs/basics/roles)
  - [Few-shot examples](https://learnprompting.org/docs/basics/few_shot)
  - [Zero-shot techniques](https://learnprompting.org/docs/advanced/zero_shot/role_prompting)

#### Prompting Guide
- **URL:** [promptingguide.ai](https://www.promptingguide.ai/)
- **Best for:** Concise technique recipes (CoT, RAG, etc.)
- **Focus:** Quick reference for specific patterns
- **Key sections:**
  - [Chain-of-Thought](https://www.promptingguide.ai/techniques/cot)
  - Structured outputs
  - Advanced prompting techniques

#### The Prompt Engineering Playbook for Programmers
- **URL:** [addyo.substack.com](https://addyo.substack.com/p/the-prompt-engineering-playbook-for)
- **Author:** Addy Osmani (Google Chrome team)
- **Best for:** Developer-focused prompt engineering
- **Focus:** Practical patterns for code generation, automation, and development workflows
- **Topics:** Context precision, reusable prompt templates, task decomposition

#### LaunchDarkly Prompt Engineering Best Practices
- **URL:** [launchdarkly.com/blog/prompt-engineering-best-practices](https://launchdarkly.com/blog/prompt-engineering-best-practices/)
- **Best for:** Production deployment considerations
- **Focus:** Safety, guardrails, and feature flag integration with LLMs
- **Topics:** Input validation, prompt injection prevention, constraint specification

#### Kinde Guide: Prompt Patterns That Scale
- **URL:** [kinde.com/learn](https://kinde.com/learn/ai-for-software-engineering/prompting/prompt-patterns-that-scale-reusable-llm-prompts-for-dev-eams/)
- **Best for:** Team-scale prompt engineering
- **Focus:** Reusable patterns for development teams
- **Topics:** Centralized prompt management, version control, scalable architectures

### Model-Specific Documentation

#### OpenAI Prompt Engineering Guide
- **Best for:** GPT-4, GPT-3.5 specific tips
- **Topics:** Structured outputs, function calling, best practices
- **Key features:**
  - Native JSON mode
  - Function/tool calling
  - System prompts optimization

#### Anthropic (Claude) Best Practices
- **URL:** [claude.ai/blog/best-practices-for-prompt-engineering](https://www.claude.com/blog/best-practices-for-prompt-engineering)
- **Best for:** Claude Opus/Sonnet/Haiku optimization
- **Topics:** XML tags, thinking blocks, long context handling
- **Key features:**
  - Prompt caching
  - Extended context windows (200K tokens)
  - Constitutional AI principles

#### AWS (Bedrock) Nova Prompting Guide
- **URL:** [AWS Nova User Guide](https://docs.aws.amazon.com/nova/latest/userguide/prompting-structured-output.html)
- **Best for:** AWS Bedrock models, enterprise deployments
- **Topics:** Structured outputs, RAG, guardrails

---

## Tools & Platforms

### Prompt Management & Evaluation

#### OpenAI Playground
- **URL:** [platform.openai.com/playground](https://platform.openai.com/playground)
- **Purpose:** Interactive prompt testing and experimentation
- **Key features:**
  - Live testing with multiple models
  - Temperature and parameter tuning
  - System/user message management
  - Export to code
- **Best for:** Rapid prototyping and initial prompt development

#### LangSmith (by LangChain)
- **Purpose:** Prompt versioning, testing, debugging
- **Key features:**
  - Trace LLM calls with full context
  - A/B test prompt versions
  - Build evaluation datasets
  - Monitor production performance
- **Best for:** Development and production monitoring

#### Langfuse
- **Purpose:** Open-source LLM observability
- **Key features:**
  - Prompt management and versioning
  - Cost tracking per prompt
  - Trace chains and agents
  - User feedback collection
- **Best for:** Self-hosted prompt management

#### Humanloop
- **Purpose:** Prompt IDE and evaluation platform
- **Key features:**
  - Collaborative prompt editor
  - Evaluation harnesses
  - Human feedback loops
  - Model comparison
- **Best for:** Team collaboration on prompts

#### PromptLayer
- **Purpose:** Prompt registry and analytics
- **Key features:**
  - Centralized prompt storage
  - [Structured output templates](https://docs.promptlayer.com/features/prompt-registry/structured-outputs)
  - Request logging
  - Cost analytics
- **Best for:** Prompt versioning and compliance

#### Maxim AI
- **Purpose:** Enterprise prompt evaluation
- **Key features:**
  - Automated testing
  - Regression detection
  - Custom evaluation metrics
- **Best for:** Large-scale prompt testing

#### DeepEval
- **URL:** [github.com/confident-ai/deepeval](https://github.com/confident-ai/deepeval)
- **Purpose:** Open-source LLM evaluation framework
- **Key features:**
  - Unit testing for LLM outputs
  - Multiple evaluation metrics (hallucination, toxicity, bias)
  - Integration with pytest
  - CI/CD pipeline support
- **Best for:** Automated testing and quality assurance

#### OpenAI Evals
- **URL:** [github.com/openai/evals](https://github.com/openai/evals)
- **Purpose:** Framework for evaluating LLM performance
- **Key features:**
  - Standardized evaluation templates
  - Custom eval creation
  - Benchmark datasets
  - Community-contributed evals
- **Best for:** Standardized model evaluation and benchmarking

### Guardrails & Safety

#### Guardrails AI
- **Purpose:** Validate and correct LLM outputs
- **Key features:**
  - Schema validation
  - PII detection
  - Custom validators
  - Output correction
- **Best for:** Production safety layers

#### LLM Guard
- **Purpose:** Open-source security toolkit
- **Key features:**
  - Prompt injection detection
  - Content moderation
  - PII scrubbing
  - Jailbreak prevention
- **Best for:** Security-focused applications

#### Moderation APIs
- **OpenAI Moderation API:** Built-in content policy checks
- **Perspective API (Google):** Toxicity scoring
- **Azure Content Safety:** Multi-level content filtering

---

## Libraries & Frameworks

### Orchestration Frameworks

#### LangChain
- **Purpose:** Build LLM applications with chains and agents
- **Use cases:**
  - RAG pipelines
  - Multi-step workflows
  - Agent orchestration
- **Prompt patterns:**
  - Template management
  - Few-shot example selectors
  - Output parsers
- **Trade-offs:** Feature-rich but can be heavy for simple use cases

#### LlamaIndex
- **Purpose:** Data connectors and RAG
- **Use cases:**
  - Index documents for retrieval
  - Query engines
  - Context-aware generation
- **Prompt patterns:**
  - Context retrieval
  - Response synthesis
  - Citation generation
- **Trade-offs:** Excellent for RAG, less focused on general prompting

#### Semantic Kernel (Microsoft)
- **Purpose:** Enterprise LLM orchestration
- **Use cases:**
  - .NET applications
  - [Structured outputs with JSON schema](https://devblogs.microsoft.com/semantic-kernel/using-json-schema-for-structured-output-in-net-for-openai-models/)
  - Plugin-based architecture
- **Trade-offs:** Great for Microsoft ecosystem integration

### Specialized Libraries

#### Instructor (Python)
- **Purpose:** Structured output extraction using Pydantic
- **Key features:**
  - Type-safe LLM outputs
  - Automatic retries on validation failures
  - Native integration with OpenAI
- **Example:**
```python
from instructor import from_openai
from pydantic import BaseModel

class User(BaseModel):
    name: str
    age: int

client = from_openai(openai.OpenAI())
user = client.chat.completions.create(
    model="gpt-4",
    response_model=User,
    messages=[{"role": "user", "content": "Extract: John is 30 years old"}]
)
```

#### Outlines
- **Purpose:** Structured text generation
- **Key features:**
  - Guarantee valid JSON output
  - Regex-constrained generation
  - Grammar-based generation

#### LLMLingua
- **Purpose:** Prompt compression
- **Key features:**
  - Reduce prompt tokens by 50-70%
  - Preserve semantic meaning
  - Maintain task performance
- **Use case:** Cost optimization for long prompts

---

## Repositories & Examples

### Production-Grade Examples

#### OpenAI Cookbook
- **URL:** [github.com/openai/openai-cookbook](https://github.com/openai/openai-cookbook)
- **Contains:**
  - Real-world use case examples
  - Code snippets for common patterns
  - Evaluation methodologies
  - Best practices for embeddings, fine-tuning, and function calling

#### Anthropic Prompt Library
- **URL:** [docs.anthropic.com/claude/prompt-library](https://docs.anthropic.com/claude/prompt-library)
- **Contains:**
  - Production prompt templates
  - Claude-specific optimizations
  - Enterprise use cases
  - Copy-paste ready examples

#### Awesome Prompt Engineering
- **URL:** [github.com/promptslab/Awesome-Prompt-Engineering](https://github.com/promptslab/Awesome-Prompt-Engineering)
- **Contains:**
  - Curated list of prompt engineering guides
  - Research papers and tutorials
  - Tools and frameworks
  - Community resources
- **Best for:** Comprehensive reference of PE resources

#### DAIR.AI Prompt Engineering Guide (Repo)
- **URL:** [github.com/dair-ai/Prompt-Engineering-Guide](https://github.com/dair-ai/Prompt-Engineering-Guide)
- **Contains:**
  - Comprehensive guides and tutorials
  - Academic papers collection
  - Jupyter notebooks with examples
  - Multilingual resources
  - RAG and AI agent patterns
- **Best for:** Academic and research-oriented learning

### Evaluation Frameworks

#### PromptBench
- **URL:** arXiv papers on prompt evaluation
- **Contains:**
  - Benchmark datasets
  - Evaluation metrics
  - Robustness testing

#### LangChain Benchmarks
- **Contains:**
  - Chain evaluation tools
  - RAG performance benchmarks

---

## Provider Documentation

### OpenAI

- **Prompt Engineering Guide:** Best practices and techniques
- **API Reference:** Structured outputs, function calling
- **JSON Mode:** Native JSON generation
- **Structured Outputs:** Schema-enforced generation

### Anthropic (Claude)

- **Best Practices:** [Claude prompt engineering guide](https://claude.com/blog/best-practices-for-prompt-engineering)
- **Prompt Caching:** Reduce cost for repeated context
- **Long Context:** Handling 200K token contexts
- **XML Tags:** Claude's preferred structure format

### Google (Gemini)

- **Prompting Guide:** Gemini-specific techniques
- **Grounding:** Connect to search and user data
- **Function Calling:** Tool use patterns

### AWS (Bedrock)

- **Prompting Best Practices:** Enterprise patterns
- **Structured Output:** [AWS Nova guide](https://docs.aws.amazon.com/nova/latest/userguide/prompting-structured-output.html)
- **Guardrails:** Policy-based output filtering

---

## Community Resources

### Discussion Forums

- **r/PromptEngineering:** Reddit community for techniques and news
- **OpenAI Community Forum:** Official OpenAI discussions
- **Anthropic Discord:** Claude user community
- **LangChain Discord:** Framework-specific help

### Blogs & Newsletters

- **Hamel Husain's Blog:** Production ML and prompt engineering
  - [hamel.dev](https://hamel.dev/) - Practical ML engineering insights
- **Addy Osmani (Google Chrome):** Developer-focused AI content
  - [The Prompt Engineering Playbook for Programmers](https://addyo.substack.com/p/the-prompt-engineering-playbook-for)
- **Latitude AI Blog:** [LLM optimization techniques](https://latitude-blog.ghost.io/blog/5-ways-to-optimize-llm-prompts-for-production-environments/)
  - [10 Best Practices for Production-Grade LLM Prompt Engineering](https://latitude-blog.ghost.io/blog/10-best-practices-for-production-grade-llm-prompt-engineering/)
- **LaunchDarkly Blog:** Feature flags and AI integration
  - [Prompt Engineering Best Practices](https://launchdarkly.com/blog/prompt-engineering-best-practices/)
- **Red Hat AI Blog:** [Hallucination prevention](https://www.redhat.com/en/blog/when-llms-day-dream-hallucinations-how-prevent-them)
- **PromptHub:** [Prompt management platforms comparison](https://www.prompthub.us/blog/three-prompt-engineering-methods-to-reduce-hallucinations)
- **Kinde Learn:** Team-scale prompt engineering
  - [Prompt Patterns That Scale](https://kinde.com/learn/ai-for-software-engineering/prompting/prompt-patterns-that-scale-reusable-llm-prompts-for-dev-eams/)

### Research Papers

#### Essential Papers

- **Chain-of-Thought Prompting:** [arXiv:2201.11903](https://arxiv.org/abs/2201.11903)
- **Few-Shot Learning:** Foundational concepts
- **Role Prompting:** [OpenReview paper](https://openreview.net/pdf?id=_VjQlMeSB_J)
- **Prompt Engineering Survey:** [arXiv:2404.01077](https://arxiv.org/html/2404.01077v1)

---

## Tool Selection Guide

### By Use Case

| Use Case | Recommended Tools |
|----------|-------------------|
| **Prototyping** | OpenAI Playground, Anthropic Console |
| **Team collaboration** | Humanloop, PromptLayer |
| **Production monitoring** | LangSmith, Langfuse |
| **Evaluation & testing** | DeepEval, OpenAI Evals |
| **RAG systems** | LlamaIndex, LangChain |
| **Structured outputs** | Instructor, native APIs |
| **Safety** | Guardrails AI, provider moderation APIs |
| **Cost optimization** | LLMLingua, prompt caching |

### By Team Size

**Solo Developer:**
- OpenAI/Anthropic Playground
- DeepEval for testing
- Simple logging to files
- Native structured output APIs

**Small Team (2-10):**
- Langfuse or PromptLayer
- OpenAI Evals for benchmarking
- Shared prompt repository (Git)
- Basic evaluation scripts

**Large Team/Enterprise:**
- LangSmith or Humanloop
- Dedicated prompt management platform
- Automated evaluation pipelines (DeepEval, custom)
- Custom guardrails

---

## Staying Current

### How to Keep Up

1. **Follow provider changelogs:**
   - OpenAI, Anthropic, Google release notes
   - New features often enable better patterns

2. **Join communities:**
   - Discord/Slack groups for frameworks
   - Reddit discussions on new techniques

3. **Read research:**
   - Follow arXiv for "prompt engineering" papers
   - Focus on reproducible techniques

4. **Experiment:**
   - Test new models on your eval sets
   - A/B test new patterns vs current baseline

### Key Trends to Watch

- **Native structured outputs:** Providers adding schema-enforced generation
- **Prompt caching:** Reducing cost for repeated context
- **Longer contexts:** 200K+ token windows enable new patterns
- **Smaller, faster models:** GPT-3.5-level quality at lower latency/cost
- **Multimodal prompting:** Vision + text combined patterns

---

## Related Documentation

- **[Patterns](./patterns/):** Apply patterns using these tools
- **[Debugging](./debugging.md):** Use tools for debugging workflows
- **[Checklist](./checklist.md):** Quality gates before production
- **[Use Cases](./use-cases.md):** See patterns in action

---

**Back to:** [Main Documentation](./README.md)

