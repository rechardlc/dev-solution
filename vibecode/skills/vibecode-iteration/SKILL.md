---
name: vibecode-iteration
description: >-
  Analyzes feature iteration in existing codebases: gathers keywords/code/APIs/docs/tests,
  traces call chains, validates context against code, and outputs structured plans with
  impact, risks, test cases, and required vs optional changes. Use when the user attaches
  this skill or asks for vibecode iteration, requirement iteration, 需求迭代, or impact
  analysis before changing legacy features.
disable-model-invocation: true
---

# Vibecode Requirement Iteration

**Opening statement:** "I'm using the vibecode-iteration skill for requirement iteration analysis."

Chinese workflow reference: [开发功能迭代.md](../../开发功能迭代.md).

## Workflow

Execute in order. Do not skip to implementation until planning is complete (unless the user explicitly asks).

### 1. Information Gathering

Search in parallel across:

- Keywords, code (entry points, core functions, callers/callees)
- APIs (paths, request/response, gateway/interceptor rules)
- Docs (README, architecture, API specs, historical designs)
- Tests (unit/integration/E2E, manual records)

Do not rely on user narration or a single file.

### 2. Call Chain Tracing

Output full chain (entry → persistence), key dependencies, and core function boundaries. Use mermaid or a step list.

### 3. Context Validation

**Assume descriptions, docs, and existing code may be wrong.**

Separate **confirmed facts** (code/doc backed) from **assumptions to verify**. Check branches, legacy compatibility, cross-module/cross-platform gaps, and boundaries (auth, encryption, cache, errors, concurrency).

### 4. Iteration Planning

Produce a structured plan using the template in [reference.md](reference.md). Include required vs optional changes, impact scope, risks, and test cases.

### 5. Test Cases

Inventory existing tests before changes; verify after. Minimum: P0 happy path + critical errors for required changes; regression cases for impact scope. See [reference.md](reference.md#test-cases).

### 6. Priority

Prefer **minimal, verifiable, reversible** changes. Document trade-offs. See [reference.md](reference.md#change-priority).

## Checklist

```
- [ ] Gathered keywords, code, APIs, docs, tests
- [ ] Traced call chain and dependencies
- [ ] Separated facts from assumptions
- [ ] Produced plan (impact, risks, required/optional, test cases)
- [ ] Verified all test cases after changes
```

## Anti-Patterns

- Trusting user/docs without reading code
- Single-file edits without tracing callers/callees
- No required vs optional distinction
- Claiming done without test verification
- Implementing before risk/plan review (unless user skips planning)

## Additional Resources

- Plan template and test details: [reference.md](reference.md)
