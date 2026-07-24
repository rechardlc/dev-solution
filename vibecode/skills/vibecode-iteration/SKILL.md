---
name: vibecode-iteration
description: >-
  Use when work is vibecode iteration of an existing or legacy feature involving requirement
  iteration, impact analysis, before/after feature comparison, or post-change alignment
  (存量功能迭代、需求迭代、影响分析、功能对比、功能对齐); not for ordinary greenfield features.
---

# Vibecode Requirement Iteration

## Workflow

Execute in order. Use [reference.md](reference.md) as the canonical source for complete templates and detailed rules.

### 1. Information Gathering

Search in parallel across:

- Keywords, code (entry points, core functions, callers/callees)
- APIs (paths, request/response, gateway/interceptor rules)
- Docs (README, architecture, API specs, historical designs)
- Tests (unit/integration/E2E, manual records)

Do not rely on user narration or a single file.

### 2. Call Chain Tracing

Trace the real chain from **entry → terminal sink**, including key dependencies and core boundaries. A sink may be render/state, persistence, external API, or event emission. Do not invent persistence when the behavior does not write data.

### 3. Context Validation

**Assume descriptions, docs, and existing code may be wrong.**

Separate **confirmed facts** (code/doc backed) from **assumptions to verify**. Check branches, legacy compatibility, cross-module/cross-platform gaps, and boundaries (auth, encryption, cache, errors, concurrency).

### 4. Planning

Produce the plan from [reference.md](reference.md), including impact, dependencies, test intent, and two independent classifications:

- **Scope Priority — Required / Optional:** Required is necessary to satisfy the user's goal; Optional can be deferred without defeating it.
- **Risk Gate — High / Normal:** High signals include persisted data shape, auth/permission, money, external API contract, security/encryption, and concurrency/idempotency. Everything else is Normal unless evidence shows otherwise.

Risk does not determine scope: **High risk never automatically promotes Optional to Required**.

### 5. Before/After Comparison

Before code changes, output the comparison required by [reference.md](reference.md#feature-comparison-expectation): checkable assumptions, dimensions (including unchanged behavior), and representative boundary/in/out rows. "Before" must be code-backed; each boundary must trace to a real branch.

**Hard gate:**

- **High-risk Required:** present and obtain confirmation before editing code, unless the user explicitly skips the gate.
- **Normal-risk Required:** no extra confirmation is required.
- **Optional:** never blocks implementation, regardless of risk.

### 6. Test Design

Inventory existing tests and design P0 happy-path/critical-error coverage for Required scope plus impact regressions. Every Required `DIM-xx`/`EX-xx` row whose Alignment verdict requires test proof must be referenced by at least one TC through that TC's **Linked Comparison Row** field. Optional or not-applicable comparison rows may remain unreferenced. Only a TC with no corresponding comparison row uses `-` in its own **Linked Comparison Row** field. See [reference.md](reference.md#test-cases).

### 7. Implementation

After applicable gates, implement minimal, verifiable, reversible changes. Keep Optional work non-blocking and document trade-offs.

### 8. Verification and Alignment

Run the designed tests and record each TC in **Test Execution Results** with Status (**Pass / Fail / Blocked / Not Run**), Actual Result, and Evidence/Log. Then reuse the comparison dimensions and rows for alignment. Record Actual and one verdict: **Pass / Fail / Partial / Blocked / Not Run**. Alignment evidence must cite the linked **TC ID and its Test Execution Results actual result**; Optional or not-applicable comparison rows not referenced by a TC must cite another evidence source or state `Not applicable`.

**Done only when:**

- Required rows are Pass, or the user explicitly accepts observed Partial/Fail behavior deviations.
- Related P0 tests have executed with acceptable actual results.
- No Required row or related P0 is Blocked or Not Run.

User acceptance can approve a verified behavior deviation; it cannot turn unexecuted verification into a pass. Never claim completion when Required evidence is Blocked or Not Run.

## Checklist

```
- [ ] Gathered keywords, code, APIs, docs, tests
- [ ] Traced entry → actual terminal sink and dependencies
- [ ] Separated facts from assumptions
- [ ] Classified Scope Priority and Risk Gate independently
- [ ] Produced plan and code-backed before/after comparison
- [ ] Confirmed High-risk Required comparison (or recorded explicit skip)
- [ ] Verified every test-proven Required DIM/EX row is referenced by at least one TC's Linked Comparison Row
- [ ] Implemented only after applicable gates
- [ ] Recorded Test Execution Results and alignment verdicts with linked TC actuals
- [ ] No Required row or related P0 remains Blocked/Not Run at completion
```

## Anti-Patterns

- Trusting user/docs without reading code
- Single-file edits without tracing callers/callees
- Assuming every chain ends in persistence
- Conflating Required/Optional scope with High/Normal risk
- Promoting Optional work merely because it is high risk
- Feature comparison without scenario assumptions or without a self-made example table
- Boundary examples invented without tracing to an actual code branch
- Editing High-risk Required code without confirmation or an explicit skip
- Blocking implementation on Optional work
- Using inconsistent linkage labels instead of Linked Comparison Row
- Claiming Pass without a TC ID and actual result
- Treating accepted behavior deviation as executed verification
- Claiming done while Required or related P0 evidence is Blocked/Not Run

## Additional Resources

- Plan, comparison, test, and alignment templates: [reference.md](reference.md) — canonical source; keep this file concise rather than copying templates
