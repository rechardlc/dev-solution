# Vibecode Iteration Reference

> Canonical source for templates and gate rules. If wording here ever conflicts with `SKILL.md` or `../../开发功能迭代.md`, this file wins; update the other two to match.

## Iteration Plan Template

```markdown
# [Feature Name] Iteration Plan

## Background & Goals
[One sentence: what to change and why]

## Call Chain Summary
[Entry → key nodes → terminal sink, with core dependencies]
- Entry: [HTTP/UI/job/event/function entry]
- Terminal sink: [render/state | persistence | external API | event emission]
- Evidence: [paths, symbols, branches]

## Iteration Steps
1. [Step 1: what to do and why this order]
2. [Step 2: ...]

## Impact Scope
| Type | Path/Module | Change |
|------|-------------|--------|
| Code | ... | ... |
| API | ... | ... |
| Config | ... | ... |

## Scope Priority
| Priority | Item | Reason | Concrete signal |
|----------|------|--------|-----------------|
| Required | ... | Necessary to satisfy [specific user goal] | [Explicit ask / behavior needed for acceptance] |
| Optional | ... | Can be deferred without defeating the goal | [Polish / tech debt / independent enhancement] |

## Risk Gate
| Risk | Item | Reason | Concrete signal |
|------|------|--------|-----------------|
| High | ... | Incorrect change has a sensitive or externally visible consequence | [persisted data shape / auth/permission / money / external API contract / security/encryption / concurrency/idempotency] |
| Normal | ... | No High-risk signal found in traced scope | [Code-backed reason] |

## Feature Comparison Expectation
[Mandatory style: assumptions + ID-bearing dimension table + ID-bearing example table — see section below]

## Open Questions
- [ ] ...

## Existing Test Inventory
- Covered: [paths/cases]
- Gaps: [scenarios to add]

## Test Cases
| ID | Scenario | Preconditions | Steps | Expected | Priority | Method | Linked Comparison Row |
|----|----------|---------------|-------|----------|----------|--------|-----------------------|
| TC-01 | Happy path | ... | ... | ... | P0 | Auto | DIM-01, EX-01 |
| TC-02 | Boundary/error | ... | ... | ... | P0 | Auto | DIM-02, EX-02 |
| TC-03 | Regression | ... | ... | ... | P1 | Manual | DIM-02 |

## Test Execution Results
| TC ID | Status | Actual Result | Evidence/Log |
|-------|--------|---------------|--------------|
| TC-01 | Pass / Fail / Blocked / Not Run | [Observed behavior, or why execution did not occur] | [Command, report, screenshot, or log location] |
| TC-02 | ... | ... | ... |

## Implementation
- Planned edits: [minimal files/symbols and intended behavior]
- Gate status: [High-risk Required comparison confirmed / explicit skip recorded / no confirmation gate applies]
- Reversibility: [rollback or compatibility approach]
- Optional handling: [deferred items and trade-offs]

## Verification
- Commands/checks: [exact test, lint, build, or manual commands]
- Actual results: [TC ID → observed result, not only pass/fail labels]
- Alignment: [reuse every applicable DIM/EX row in Feature Alignment]
- Completion status: [Complete / Incomplete, with blocking evidence]
```

### Call-chain rules

- Trace the real chain from **entry → terminal sink**, including key dependencies and core branch boundaries.
- A terminal sink may be **render/state**, **persistence**, **external API**, or **event emission**.
- Select the sink supported by code evidence. If the behavior does not write persisted data, do not invent a database, cache, file, or other persistence step.

## Scope Priority and Risk Gate

These are independent classifications. Every planned item must state both classifications with a reason and a concrete signal.

### Scope Priority

| Priority | Definition | Typical concrete signals |
|----------|------------|--------------------------|
| **Required** | Must be delivered to satisfy the user's stated goal or acceptance behavior | Explicit ask; omission leaves the target behavior unmet; necessary defect fix |
| **Optional** | Can be deferred without defeating the user's goal | Polish; tech debt; cleanup; independent enhancement |

### Risk Gate

| Risk | Definition | High-risk concrete signals |
|------|------------|----------------------------|
| **High** | The traced change touches a sensitive behavior or contract | persisted data shape; auth/permission; money; external API contract; security/encryption; concurrency/idempotency |
| **Normal** | No High-risk signal is present in the traced scope | Cite the inspected code/contract boundaries that support this conclusion |

Risk does not determine scope. A High-risk Optional item remains Optional and does not block implementation. A Normal-risk item may still be Required because it is necessary for the user's goal.

If scope or risk is uncertain, record the uncertainty under Open Questions; do not substitute one classification for the other.

## Feature Comparison Expectation

Output **before code changes**. The mandatory structure has three parts: assumptions, dimensions, and self-enumerated examples.

### Required structure

```markdown
### Before vs After

Assumption: … (concrete checkable values: dates, flags, N, user state, …)

| Row ID | Dimension | Scope Priority | Risk Gate | Before | After | Code/branch evidence |
|--------|-----------|----------------|-----------|--------|-------|----------------------|
| DIM-01 | … | Required | Normal | Code-backed current behavior | Target behavior | `path:symbol`, inspected branch/query |
| DIM-02 | … | Optional | High | Code-backed current behavior | unchanged / same as before | `path:symbol`, inspected branch/query |

#### Example (…key params…)

| Row ID | Sample | Key fields… | Scope Priority | Risk Gate | Before | After | Branch evidence |
|--------|--------|-------------|----------------|-----------|--------|-------|-----------------|
| EX-01 | In range | … | Required | Normal | ✅ / ❌ (optional note) | ✅ / ❌ (optional note) | `path:symbol`, condition |
| EX-02 | Boundary | … | Required | High | … | … | `path:symbol`, condition |
```

### Rules

| Part | Rule |
|------|------|
| Assumptions | Use concrete values so readers can recompute; prefer real domain fields from code |
| Stable row IDs | Dimension rows use `DIM-xx`; example rows use `EX-xx`; keep IDs unchanged through test design and alignment |
| Dimension table | One row per logic axis (filter, limit, invalid input, side effects…). Unchanged behavior is explicit as `unchanged` or `same as before`; every row cites inspected code/branch evidence |
| Example table | **Self-enumerated** — cover in-range, out-of-range, and at least one real boundary; use ✅/❌ where applicable |
| Boundary source | Every boundary/edge row traces to an actual branch condition (`if`/`else`, guard clause, matcher, limit) found during call-chain tracing |
| Before | Code-backed facts only; narration or desired behavior is not Before evidence |
| Scope and risk | Record both independently; High risk never promotes Optional to Required |

### Hard gate

- **High-risk Required:** present the comparison and obtain user confirmation before editing code, unless the user explicitly skips this gate.
- **Normal-risk Required:** no additional confirmation gate applies.
- **Optional:** never blocks implementation, regardless of risk.

### Style example

```markdown
### Before vs After

Assumption: user birthday `08-15`, current year `2026`, `birthday_noread = 3`; `illustrative/` paths below belong only to this style example and do not imply files in the current repository.

| Row ID | Dimension | Scope Priority | Risk Gate | Before | After | Code/branch evidence |
|--------|-----------|----------------|-----------|--------|-------|----------------------|
| DIM-01 | Unread switch | Required | Normal | `birthday_noread <= 0` → `[]` | same as before | `illustrative/birthday_service.go:listBlessings`, early-return guard |
| DIM-02 | Year | Required | Normal | no limit | `year = 2026` only | `illustrative/birthday_repository.go:listLatest`, current query has no year predicate |
| DIM-03 | Gift send time | Required | Normal | no limit | `send_date < 2026-08-16 00:00:00` only (includes Aug 15) | `illustrative/birthday_repository.go:listLatest`, current query has no send-date predicate |
| DIM-04 | Count | Required | Normal | latest N items (N=`birthday_noread`) | filter by year/send time, then take latest N | `illustrative/birthday_repository.go:listLatest`, descending order plus `LIMIT N` |
| DIM-05 | Invalid birthday | Required | Normal | may still return a list | return `[]` | `illustrative/birthday_service.go:listBlessings`, birthday-parse error currently falls through |
| DIM-06 | Popup / unread +/- | Required | Normal | unchanged | unchanged | `illustrative/birthday_popup.go:updateUnread`, existing state-transition branches |
| DIM-07 | Response fields | Required | Normal | unchanged | unchanged | `illustrative/birthday_dto.go:BlessingResponse`, existing serialized field set |

#### Example (noread=3)

| Row ID | Blessing | send | year | Scope Priority | Risk Gate | Before | After | Branch evidence |
|--------|----------|------|------|----------------|-----------|--------|-------|-----------------|
| EX-01 | A | 2026-08-10 | 2026 | Required | Normal | ✅ may enter list | ✅ | `illustrative/birthday_repository.go:listLatest`, `year == current_year && send_date < next_day_start` true branch |
| EX-02 | B | 2026-08-15 23:00 | 2026 | Required | Normal | ✅ | ✅ (birthday-day boundary included) | `illustrative/birthday_repository.go:listLatest`, `send_date < next_day_start` true branch |
| EX-03 | C | 2026-08-16 00:00 | 2026 | Required | Normal | ✅ | ❌ | `illustrative/birthday_repository.go:listLatest`, `send_date < next_day_start` false branch |
| EX-04 | D | 2025-08-10 | 2025 | Required | Normal | ✅ (if within latest N) | ❌ | `illustrative/birthday_repository.go:listLatest`, `year == current_year` false branch |
| EX-05 | E | 2026-07-01 | 2026 | Required | Normal | ✅ | ✅ | `illustrative/birthday_repository.go:listLatest`, `year == current_year && send_date < next_day_start` true branch |
```

The `illustrative/` paths above demonstrate the required `path:symbol` format only. In an actual iteration, every DIM/EX evidence cell must cite the real inspected code location; a prose claim such as `no limit` or a condition without `path:symbol` is not Before evidence.

## Test Cases

**Inventory:** Find related test files; mark reusable regression coverage and new scenarios.

**Design dimensions:**

| Type | Focus |
|------|-------|
| Happy path | Normal end-to-end behavior |
| Boundaries | Empty/null, limits, auth edges, state transitions |
| Errors | Invalid input, timeout, dependency failure, concurrency |
| Regression | Upstream/downstream behavior in impact scope not directly changed |

**Minimum bar:**

- Required scope has P0 happy-path and critical-error coverage.
- Impact scope has regression cases; prefer automation where practical.
- Every Required `DIM-xx`/`EX-xx` row whose Alignment verdict requires test proof is referenced in at least one TC's **Linked Comparison Row** field. This includes, but is not limited to, Required boundary/edge rows.
- `Linked Comparison Row` is a Test Cases field: it contains the `DIM-xx`/`EX-xx` row IDs referenced by that TC, while the test case retains its independent `TC-xx` ID.
- Optional or not-applicable comparison rows may remain unreferenced by any TC. Their Alignment entries must still cite the available evidence source or explicitly state `Not applicable`.
- Only a TC with no corresponding comparison row uses `-` in its `Linked Comparison Row` field.
- After implementation, record every designed TC in Test Execution Results, including unexecuted P0 cases as `Blocked` or `Not Run`, and use those actual results as Feature Alignment evidence.

**Example continuing the birthday comparison:**

| ID | Scenario | Preconditions | Steps | Expected | Priority | Method | Linked Comparison Row |
|----|----------|---------------|-------|----------|----------|--------|-----------------------|
| TC-01 | Current-year in-range list and count | Valid birthday; noread=3 | Load EX-01 (A), EX-05 (E), F=`2026-06-30`, G=`2026-06-20`, H=`2026-06-10`; request list | A, E, F returned in descending send time; G/H omitted; result count=3 | P0 | Auto | DIM-02, DIM-03, DIM-04, EX-01, EX-05 |
| TC-02 | Birthday-day boundary | Valid birthday | Load EX-02 record | Record included | P0 | Auto | DIM-03, EX-02 |
| TC-03 | Next-day exclusion | Valid birthday | Load EX-03 record | Record excluded | P0 | Auto | DIM-03, EX-03 |
| TC-04 | Prior-year exclusion | Valid birthday | Load EX-04 record | Record excluded | P0 | Auto | DIM-02, EX-04 |
| TC-05 | Invalid birthday | Invalid birthday | Request list | Empty list returned | P0 | Auto | DIM-05 |
| TC-06 | Unread-zero and existing response behavior | Valid birthday; initial noread=0 | Request list and assert `[]`; reset noread=3; open popup, increment/decrement unread, inspect response | noread=0 returns `[]`; popup/unread transitions and response fields match baseline | P0 | Auto | DIM-01, DIM-06, DIM-07 |

### Test Execution Results

| TC ID | Status | Actual Result | Evidence/Log |
|-------|--------|---------------|--------------|
| TC-01 | Pass | A, E, F returned in descending send time; G/H omitted; count=3 | `go test ./illustrative/... -run TestListBlessings_CurrentYear` |
| TC-02 | Pass | Boundary record B at 2026-08-15 23:00 was included | `go test ./illustrative/... -run TestListBlessings_BirthdayBoundary` |
| TC-03 | Pass | Record C at 2026-08-16 00:00 was excluded | `go test ./illustrative/... -run TestListBlessings_NextDayExcluded` |
| TC-04 | Pass | Prior-year record D was excluded | `go test ./illustrative/... -run TestListBlessings_PriorYearExcluded` |
| TC-05 | Pass | Invalid birthday returned an empty list | `go test ./illustrative/... -run TestListBlessings_InvalidBirthday` |
| TC-06 | Pass | noread=0 returned `[]`; popup/unread transitions and response fields matched baseline | `go test ./illustrative/... -run TestListBlessings_UnreadAndResponseRegression` |

## Implementation

Implementation begins only after the applicable hard gate. Keep this section operational rather than repeating the comparison:

- Name the minimal files/symbols to change and why each is needed.
- Preserve unrelated behavior and existing contracts.
- Record compatibility, rollback, or other reversibility measures where relevant.
- Keep Optional work non-blocking; list deferred items and their trade-offs.
- If the user explicitly skips a High-risk Required comparison confirmation, record the skip before editing.

## Verification

Verification executes the designed cases and produces evidence for alignment:

1. Run the exact test, lint, build, or manual checks appropriate to the impact scope.
2. Record every designed `TC-xx` in Test Execution Results with Status (`Pass`, `Fail`, `Blocked`, or `Not Run`), observed Actual Result, and Evidence/Log. A bare “passed” without the observed result is insufficient alignment evidence.
3. Reuse the same `DIM-xx` and `EX-xx` rows in Feature Alignment.
4. Mark unexecuted or impossible checks honestly as `Not Run` or `Blocked`; do not infer Pass from code inspection.

## Feature Alignment

Output **after changes**, reusing the same dimension and example row IDs.

```markdown
### Feature Alignment

| Row ID | Dimension / Sample | Scope Priority | After (expected) | Actual | Verdict | Evidence | Deviation notes |
|--------|--------------------|----------------|------------------|--------|---------|----------|-----------------|
| DIM-01 | Unread switch | Required | noread <= 0 returns `[]` | Empty list returned when noread=0 | Pass | Test Execution Results TC-06 (Pass): noread=0 returned `[]` | - |
| DIM-02 | Year | Required | year=2026 only | 2025 excluded; 2026 retained | Pass | Test Execution Results TC-01 (Pass): A/E/F retained; TC-04 (Pass): D excluded | - |
| DIM-03 | Gift send time | Required | before 2026-08-16 only | Aug 15 included; Aug 16 excluded | Pass | Test Execution Results TC-02 (Pass): B included; TC-03 (Pass): C excluded | - |
| DIM-04 | Count | Required | filter, then take latest 3 | A, E, F returned; G/H omitted; count=3 | Pass | Test Execution Results TC-01 (Pass): A/E/F returned and count=3 | - |
| DIM-05 | Invalid birthday | Required | return `[]` | Empty list returned | Pass | Test Execution Results TC-05 (Pass): invalid birthday returned `[]` | - |
| DIM-06 | Popup / unread +/- | Required | unchanged | Existing popup and unread transitions preserved | Pass | Test Execution Results TC-06 (Pass): transitions matched baseline | - |
| DIM-07 | Response fields | Required | unchanged | Existing response field set preserved | Pass | Test Execution Results TC-06 (Pass): response fields matched baseline | - |
| EX-01 | 2026-08-10 | Required | ✅ | ✅ included | Pass | Test Execution Results TC-01 (Pass): record A returned | - |
| EX-02 | 2026-08-15 23:00 | Required | ✅ | ✅ included | Pass | Test Execution Results TC-02 (Pass): record B returned | - |
| EX-03 | 2026-08-16 00:00 | Required | ❌ | ❌ excluded | Pass | Test Execution Results TC-03 (Pass): record C absent | - |
| EX-04 | 2025-08-10 | Required | ❌ | ❌ excluded | Pass | Test Execution Results TC-04 (Pass): record D absent | - |
| EX-05 | 2026-07-01 | Required | ✅ | ✅ included | Pass | Test Execution Results TC-01 (Pass): record E returned | - |
```

The `Evidence` column cites a **linked TC ID plus its Status and real observed Actual Result from Test Execution Results**. `DIM-xx` and `EX-xx` are comparison row IDs, never TC IDs. For Optional or not-applicable comparison rows not referenced by a TC, cite the available inspection/review evidence or explicitly state `Not applicable`. A `Linked Comparison Row` value of `-` belongs only to a TC with no corresponding comparison row and is never executed proof for an Alignment row.

### Verdicts

| Verdict | Meaning |
|---------|---------|
| **Pass** | Executed evidence shows Actual matches After |
| **Fail** | Executed evidence shows Actual does not match After |
| **Partial** | Executed evidence shows only part of After is met; document the exact gap |
| **Blocked** | Verification could not execute because a dependency, environment, permission, or prerequisite blocked it |
| **Not Run** | Verification was not executed |

### Completion rule

- Required rows must be Pass, unless the user accepts an observed Partial/Fail behavior deviation in writing.
- Related P0 tests must execute with acceptable actual results.
- Any Required row or related P0 marked Blocked or Not Run prevents completion.
- Written acceptance may approve a verified behavior deviation (Partial/Fail); it cannot convert Blocked/Not Run into Pass.
