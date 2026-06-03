# Vibecode Iteration Reference

## Iteration Plan Template

```markdown
# [Feature Name] Iteration Plan

## Background & Goals
[One sentence: what to change and why]

## Call Chain Summary
[Entry → key nodes → sink, with core dependencies]

## Iteration Steps
1. [Step 1: what to do and why this order]
2. [Step 2: ...]

## Impact Scope
| Type | Path/Module | Change |
|------|-------------|--------|
| Code | ... | ... |
| API | ... | ... |
| Config | ... | ... |

## Risk Notes
- [Regression / compatibility / performance / security]

## Change Priority
### Required
- [Item] — [Consequence if skipped]

### Optional
- [Item] — [Benefit and why it can wait]

## Open Questions
- [ ] ...

## Existing Test Inventory
- Covered: [paths/cases]
- Gaps: [scenarios to add]

## Test Cases
| ID | Scenario | Preconditions | Steps | Expected | Priority | Method |
|----|----------|---------------|-------|----------|----------|--------|
| TC-01 | Happy path | ... | ... | ... | P0 | Auto |
| TC-02 | Edge/error | ... | ... | ... | P0 | Auto |
| TC-03 | Regression | ... | ... | ... | P1 | Manual |
```

## Test Cases

**Inventory:** Find related test files; mark reusable regression vs new scenarios.

**Design dimensions:**

| Type | Focus |
|------|-------|
| Happy path | Normal end-to-end behavior |
| Boundaries | Empty/null, limits, auth edges, state transitions |
| Errors | Invalid input, timeout, dependency failure, concurrency |
| Regression | Upstream/downstream behavior in impact scope not directly changed |

**Minimum bar:**

- Required changes: P0 happy path + critical error cases
- Impact scope: add regression cases; prefer automation
- After changes: run every case; iteration is not done until all pass

**Example:**

| ID | Scenario | Preconditions | Steps | Expected | Priority | Method |
|----|----------|---------------|-------|----------|----------|--------|
| TC-01 | Submit order | User logged in | Fill required fields, submit | Success; status pending payment | P0 | Auto |
| TC-02 | Unauthenticated | No token | Call order API | 401; no order created | P0 | Auto |
| TC-03 | Legacy orders | Old data exists | Open order list | Legacy orders still render | P1 | Manual |

## Change Priority

| Category | Criteria |
|----------|----------|
| **Required** | Requirement unmet, defect, or security risk without the change |
| **Optional** | UX, cleanliness, tech debt; can be deferred |
