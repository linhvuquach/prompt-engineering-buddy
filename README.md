# Prompt Engineering Buddy

A production-focused playbook for writing, testing, and maintaining LLM prompts for real-world applications.

## 📖 Documentation

**[Start Here: Full Documentation →](./docs/README.md)**

### Quick Links

- **[Core Mental Models](./docs/core-mental-models.md)** - Foundation principles (output-first design, defensive prompting)
- **[Prompt Patterns](./docs/patterns/)** - 7 production-ready patterns with templates and examples
- **[Hands-On Exercises](./docs/exercises.md)** - Practice converting flaky prompts to production-grade
- **[Debugging Guide](./docs/debugging.md)** - Fix hallucinations, format drift, and cost bloat
- **[Real-World Use Cases](./docs/use-cases.md)** - Applied patterns for SaaS, automation, and agents
- **[Resources](./docs/resources.md)** - Tools, libraries, and learning materials
- **[Production Checklist](./docs/checklist.md)** - Quality gate before deployment

## Philosophy

Treat prompts as **code**: structured, testable, and maintainable. Every pattern includes:
- When to use it
- Template structure
- Working examples
- Evaluation criteria

No fluff. Production-grade only.

## Quick Start

1. **New to prompt engineering?** Read [Core Mental Models](./docs/core-mental-models.md)
2. **Building something specific?** Browse [Patterns](./docs/patterns/) by use case
3. **Having issues?** Check the [Debugging Guide](./docs/debugging.md)
4. **Ready to ship?** Run through the [Production Checklist](./docs/checklist.md)

## What's Inside

### 7 High-Impact Patterns

1. **[Role + Task + Rules + Format](./docs/patterns/role-task-rules-format.md)** - Universal template for predictable prompts
2. **[Structured Output](./docs/patterns/structured-output.md)** - Guaranteed parseable JSON for APIs
3. **[Few-Shot Style Alignment](./docs/patterns/few-shot-style.md)** - Consistent tone and voice
4. **[Controlled Chain-of-Thought](./docs/patterns/chain-of-thought.md)** - Debuggable reasoning
5. **[Guardrailed Q&A](./docs/patterns/guardrailed-qa.md)** - Zero-hallucination RAG
6. **[Task Decomposition](./docs/patterns/task-decomposition.md)** - Multi-step workflows
7. **[Defensive Error Reporting](./docs/patterns/defensive-error-reporting.md)** - Structured failures

### Practice Exercises

Transform bad prompts into production-grade implementations:
- Support ticket classification
- Release notes generation
- Context-only documentation Q&A
- Defensive date extraction

### Real-World Applications

- **SaaS:** Support triage, analytics enrichment, docs copilots
- **Automation:** Email routing, data cleaning, report summarization
- **Agents:** Tool-calling, debuggable reasoning, guardrails

## References

- See [Resources](./docs/resources.md) for comprehensive learning materials and tools
- [Prompt Patterns Explorer](https://prompts.duyth.site/guide) - Interactive Guide to LLM Prompt Engineering
