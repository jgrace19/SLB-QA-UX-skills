---
name: assess-test-plan-gaps
description: Assess test coverage gaps for an Azure DevOps (ADO) work item by reading the item's requirements/acceptance criteria, finding linked and related test cases, and reporting what is untested. Use when the user shares an ADO work item URL or ID and asks to assess test plan gaps, review test coverage, check what tests are missing, or evaluate testing completeness for a feature, user story, bug, or epic.
disable-model-invocation: true
---

# Assess Test Plan Gaps for an ADO Work Item

Analyze an Azure DevOps work item and report where its testing is incomplete: acceptance criteria without tests, missing negative/edge cases, and untested child items. Use the `user-ado` MCP tools for all ADO access.

## Inputs

Accept either:
- A board/work item URL, e.g. `https://dev.azure.com/{org}/{project}/_boards/board/t/{team}/Issues?workitem=1`
- A bare work item ID plus project name

Parse `org`, `project`, and the `workitem` ID from the URL. The `workitem=` query param is the ID. If project or ID is ambiguous, ask before proceeding.

## Workflow

Copy this checklist and track progress:

```
- [ ] 1. Read the target work item + type
- [ ] 2. Gather child/linked/related items
- [ ] 3. Find existing test cases and results
- [ ] 4. Extract testable requirements
- [ ] 5. Map requirements to tests
- [ ] 6. Report gaps
```

### 1. Read the target work item

- `wit_get_work_item` with the ID and `project`. Request fields including `System.Title`, `System.Description`, `System.WorkItemType`, `System.State`, `Microsoft.VSTS.Common.AcceptanceCriteria`, and `System.Tags`.
- `wit_list_work_item_comments` for clarifications or agreed test scope in discussion.
- If the type is unfamiliar, use `wit_get_work_item_type` to understand its fields.

### 2. Gather related work

- Use `wit_get_work_item` relations (or `wit_get_work_items_batch_by_ids` on linked IDs) to pull children and linked items. For an Epic/Feature, enumerate child User Stories/Bugs — coverage is assessed across the tree.
- Note existing "Tested By" / test-case links on the item and its children.

### 3. Find existing test cases and results

- `testplan_list_test_plans` (active only) to locate relevant plans, then `testplan_list_test_suites` and `testplan_list_test_cases` to enumerate cases.
- `search_workitem` with the feature name/key terms to catch test cases not formally linked.
- If a related build is known, `testplan_show_test_results_from_build_id` for recent pass/fail signal.
- Read the case titles/steps to judge what behavior each actually verifies — do not assume a linked case is adequate.

### 4. Extract testable requirements

From the description and acceptance criteria, list discrete, verifiable behaviors. For each, also consider the categories in [coverage-checklist.md](coverage-checklist.md): happy path, negative/error, boundary/edge, permissions/auth, data validation, and non-functional (perf, accessibility, security).

### 5. Map requirements to tests

Build a matrix: each requirement → existing test case(s) that cover it → `Covered` / `Partial` / `Missing`. A requirement is `Partial` when only the happy path is tested but obvious negative/edge cases are not.

### 6. Report gaps

Use the template below. Do not create or modify test cases unless the user explicitly asks; this skill is assessment-only by default.

## Output template

```markdown
# Test Plan Gap Assessment — #<id> <title>

**Type:** <type> · **State:** <state> · **Existing test cases found:** <n>

## Coverage matrix
| Requirement / Acceptance criterion | Existing test(s) | Status |
|---|---|---|
| ... | TC #123 | Covered |
| ... | — | Missing |
| ... | TC #124 (happy path only) | Partial |

## Gaps
- 🔴 **Missing**: <requirement with no test> — suggested case: <one line>
- 🟡 **Partial**: <requirement> — add: <negative/edge case>
- 🟢 **Nice to have**: <non-functional or exploratory idea>

## Child items (if epic/feature)
- #<id> <title>: <covered | gaps summary>

## Summary
<1–2 sentences: overall coverage level and the top 1–3 things to test next.>
```

## Notes

- Prefer linked test cases as the source of truth, but always confirm what a case verifies by reading its steps.
- If no test plan or cases exist at all, say so plainly and treat every requirement as `Missing`.
- Keep suggestions concrete and tied to a specific requirement; avoid generic advice.
