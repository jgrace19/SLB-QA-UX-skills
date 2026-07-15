---
name: classify-test-levels
description: Analyze an Azure DevOps (ADO) work item's test plans and test cases and classify each into the right test level — unit, integration, end-to-end (E2E), or manual. Use when the user shares an ADO work item URL or ID and asks which tests should be unit vs integration vs E2E vs manual, how to distribute test cases across the test pyramid, or to recommend the appropriate test level for a feature's test plan.
disable-model-invocation: true
---

# Classify ADO Test Plans into Test Levels

Given an ADO work item and its associated test plans/cases, recommend the right test level for each: **unit**, **integration**, **end-to-end (E2E)**, or **manual**. Use the `user-ado` MCP tools for all ADO access.

## Inputs

Accept either:
- A board/work item URL, e.g. `https://dev.azure.com/{org}/{project}/_boards/board/t/{team}/Issues?workitem=1`
- A bare work item ID plus project name

Parse `org`, `project`, and the `workitem` ID (the `workitem=` query param). If project or ID is ambiguous, ask before proceeding.

## Workflow

Copy this checklist and track progress:

```
- [ ] 1. Read the work item + type
- [ ] 2. Enumerate test plans, suites, and cases
- [ ] 3. Read each case's title/steps to understand what it verifies
- [ ] 4. Classify each case by level using the decision rules
- [ ] 5. Report classification + pyramid balance
```

### 1. Read the work item

- `wit_get_work_item` with the ID and `project` (fields: `System.Title`, `System.Description`, `System.WorkItemType`, `Microsoft.VSTS.Common.AcceptanceCriteria`, `System.Tags`).
- Note the architecture cues (UI, API, service, data) that influence classification.

### 2. Enumerate test plans and cases

- `testplan_list_test_plans` (active) → `testplan_list_test_suites` → `testplan_list_test_cases`.
- Also use `search_workitem` on key terms to catch cases not linked to a formal plan.

### 3. Understand each case

Read each case's title and steps. Classify from **what the case actually exercises**, not its current suite name or where it lives today.

### 4. Classify by level

Apply [classification-rules.md](classification-rules.md). Assign exactly one recommended level per case, and flag cases that are currently at the wrong level (e.g., a pure logic check run as a slow manual E2E).

### 5. Report

Use the template below.

## Decision rules (summary)

- **Unit** — verifies a single function/class/module in isolation; no real I/O, DB, network, or UI; dependencies mocked; fast and deterministic.
- **Integration** — verifies two or more components together: a service + its database, an API endpoint + its handlers, or module boundaries/contracts. Real dependencies within a bounded scope.
- **E2E** — exercises a full user-facing flow across the deployed stack (UI → API → data), automatable via the app's real entry points.
- **Manual** — requires human judgment or is impractical to automate: exploratory, usability/visual, one-off setup, hardware/external dependencies, or not-yet-automatable flows.

See `classification-rules.md` for tie-breakers and signals.

## Output template

```markdown
# Test Level Classification — #<id> <title>

**Test cases analyzed:** <n>

## Recommended classification
| Test case | Currently | Recommended | Why |
|---|---|---|---|
| TC #123 <title> | Manual | Unit | Pure calc, no I/O — mock inputs |
| TC #124 <title> | — | Integration | API + DB round trip |
| TC #125 <title> | E2E | E2E | Full checkout flow across UI/API |

## Pyramid balance
- Unit: <n> · Integration: <n> · E2E: <n> · Manual: <n>
- <1–2 sentences: is the distribution healthy (many unit, fewer E2E)? Note over-reliance on manual/E2E.>

## Recommendations
- 🔴 **Re-level**: <case> should move from <x> to <y> because <reason>.
- 🟡 **Consider**: <coverage better served at a different level>.
```

## Notes

- Prefer pushing coverage to the lowest level that can meaningfully verify the behavior (test pyramid): unit > integration > E2E > manual.
- A single acceptance criterion may warrant tests at multiple levels; call that out rather than forcing one.
- Classification-only by default — do not create, move, or modify test cases unless the user explicitly asks.
