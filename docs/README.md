# Practical Prompt Engineering Playbook

A production-focused guide to writing, testing, and maintaining LLM prompts for real-world applications.

## 📚 Documentation Structure

### 1. [Core Mental Models](./core-mental-models.md)
Foundation principles for thinking about prompt engineering:
- Prompt as config, not chat
- Output-first design
- Examples > instructions
- Decompose tasks
- Defensive prompting

### 2. Prompt Patterns

High-impact patterns for production use:

- **[Role + Task + Rules + Format](./patterns/role-task-rules-format.md)**  
  The universal template for predictable, debuggable prompts

- **[Structured Output (Strict JSON)](./patterns/structured-output.md)**  
  Data extraction and ETL with guaranteed parseability

- **[Few-Shot Style & Quality Alignment](./patterns/few-shot-style.md)**  
  Controlling tone, voice, and consistency across generations

- **[Controlled Chain-of-Thought](./patterns/chain-of-thought.md)**  
  Debuggable reasoning traces for complex decisions

- **[Guardrailed Factual Answering](./patterns/guardrailed-qa.md)**  
  Low-hallucination Q&A for RAG and compliance use cases

- **[Task Decomposition](./patterns/task-decomposition.md)**  
  Multi-call orchestration for complex workflows

- **[Defensive Error Reporting](./patterns/defensive-error-reporting.md)**  
  Structured error handling instead of hallucinated data

### 3. [Hands-On Exercises](./exercises.md)
Practice converting flaky prompts to production-grade:
- Support ticket classification
- Release notes generation
- Context-only Q&A
- Error-handling extractors

### 4. [Debugging & Optimization](./debugging.md)
Common failure modes and fix strategies:
- Hallucinations & format drift
- Prompt drift & version control
- Parameter tuning (temperature, top-p, etc.)
- Cost optimization strategies

### 5. [Security Best Practices](./security-best-practices.md)
Protect against attacks and data leaks:
- Prompt injection prevention
- Input validation & sanitization
- PII detection and redaction
- Multi-layer defense strategies

### 6. [Real-World Use Cases](./use-cases.md)
Applied patterns by domain:
- **SaaS:** Support triage, analytics, docs copilots
- **Automation:** Email routing, CI/CD integration, test data generation
- **Agents:** Tool-calling, debuggable reasoning, guardrails

### 7. [Resources](./resources.md)
Curated tools, libraries, and learning materials

### 8. [Production Checklist](./checklist.md)
Quality gate checklist for every production prompt

---

## Quick Start

1. **New to prompt engineering?** Start with [Core Mental Models](./core-mental-models.md)
2. **Building a specific feature?** Jump to the relevant [Pattern](./patterns/)
3. **Debugging issues?** Check [Debugging & Optimization](./debugging.md)
4. **Shipping to production?** Use the [Production Checklist](./checklist.md)

## Philosophy

This playbook treats prompts as **code**: structured, testable, and maintainable. Every pattern includes:
- When to use it
- Template structure
- Working examples
- Evaluation criteria

No fluff. Production-grade only.

