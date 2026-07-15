---
name: triage-e2e-pipeline-failure
description: >-
  Triage a failing Azure DevOps E2E pipeline end-to-end: pull the failing build
  and logs, pinpoint the root cause, trace it to the change and work item that
  introduced it, find why it slipped past cheaper test stages, post a root-cause
  comment on that work item, and create earlier-stage Test Cases that would catch
  it before E2E. Use when an ADO pipeline or E2E run is red, when given a
  dev.azure.com build/pipeline URL, or when asked to investigate, diagnose, or
  find the root cause of a CI/pipeline failure.
---

# Triage E2E Pipeline Failure

Diagnose a red Azure DevOps (ADO) pipeline, attribute it to the work that caused
it, and close the loop by commenting the root cause on the work item and adding
the cheaper tests that should have caught it.

All ADO actions use the `user-ado` MCP server. **Always read a tool's descriptor
JSON in `mcps/user-ado/tools/<tool>.json` before calling it** with `CallMcpTool`.

## Operating principles

- **Evidence or it didn't happen.** Every root-cause claim cites a real
  `file:line` (read or grepped) and the exact failing assertion from the log.
- **Find the cheapest missed gate.** The deliverable isn't just "what broke" —
  it's "what fast test (unit > component > contract > integration) should have
  caught this before the expensive E2E stage, and why it didn't."
- **Implementation vs. test.** Decide which is wrong by checking the work item's
  acceptance criteria. The failing test may be correct and the code the bug.
- **Read-only on code.** Analyze statically; recommend/author Test Case work
  items. Do not run builds or fix code unless the user asks.
- **Confirm before writing to ADO.** Post the comment and create Test Cases only
  after the root cause is established; surface the plan, then execute.

## Inputs

Collect before starting:

1. **Pipeline or build URL / definition id** (e.g. `_build?definitionId=5`) — required.
2. **Project** — default `FieldEng` unless the URL or user says otherwise.

If only a pipeline is given, find the latest failing build yourself.

## Workflow

Track progress with a todo list. Steps 1–4 are investigation; 5–6 write to ADO.

### 1. Find the failing build

- `pipelines_get_builds` with `definitions: [<id>]`, `top: 10`,
  `queryOrder: QueueTimeDescending`.
- Result codes: `2` = succeeded, `8` = failed, `4` = partial, `32` = canceled
  (`statusFilter`/`result` are bitmasked enums). Pick the most recent failing
  build; note whether it's a first failure or a standing red.

### 2. Pinpoint the root cause from logs

- `pipelines_get_build_log` lists log segments (id, lineCount). The last large
  segment is usually the failing step.
- `pipelines_get_build_log_by_id` for that segment. Large output is written to a
  file — `Grep`/`Read` it rather than dumping. Search for
  `error|fail|✘|✗|expect|Expected|Received|exited with code`.
- Capture the **exact** failing assertion: test name, file:line, expected vs.
  received, and the step exit. Note what passed (upstream stages green ⇒ the
  defect is downstream of those gates).

### 3. Attribute it to a change and work item

- `pipelines_get_build_changes` (`includeSourceChange: true`) for the build's
  commits/PRs. Map the failing area to a PR and its `AB#<n>` work item.
- In the affected repo, `git log --oneline -- <path>` to find the commit/PR that
  introduced the behavior (the feature PR) vs. the one that exposed it (often the
  test PR). They may differ.
- `wit_get_work_item` (`expand: relations`) for the work item: read the
  acceptance criteria and any `Tested By` Test Cases. The AC decides whether the
  code or the test is wrong.

### 4. Inspect source + existing coverage; find the gap

- Read the implicated source (`file:line` from the log/trace) and the test that
  failed.
- Read the **fast-stage** suites for that area (unit/component/contract). State
  precisely which sibling cases exist and which variant was never asserted — that
  omission is why it reached E2E.
- Decide the fix direction (one sentence) and the cheapest layer that should
  guard it.

### 5. Comment the root cause on the work item

`wit_add_work_item_comment` (`format: Markdown`) on the introducing work item.
Use this template:

```markdown
**Root cause of the failing E2E pipeline** (<project> pipeline def <id>, build <n>).

**Symptom:** <test file:line / name> fails:
```
<expected vs received, trimmed>
```
Upstream stages (<list>) are green; only this assertion fails.

**Root cause:** <one-paragraph mechanism with file:line>. This <matches/contradicts>
the acceptance criterion "<quote AC>".

**How it was introduced:** PR <x> (<commit>) introduced the behavior; PR <y>
exposed it. <note if merged separately>.

**Why it slipped to E2E (coverage gap):** <which fast suites covered which
variants, and the one they missed>.

**Fix direction:** <one sentence>.

**Earlier-stage tests added:** #<id> (...), #<id> (...), ...
```

### 6. Add earlier-stage Test Cases

`testplan_create_test_case` per proposed test, linked to the work item via
`testsWorkItemId: <id>` (creates the `Tested By` relation, matching existing
cases). Set `project`, `priority`, `areaPath`, `iterationPath` to match siblings.

- Steps format: `1. Action|Expected result\n2. Action|Expected result`. Use `|`
  only as the step/expected delimiter; never inside the text.
- Prefer the cheapest layers and name them in the title, e.g.
  `Unit (<svc>): ...`, `API (<svc>): ...`, `Contract: ...`. Include at least one
  test at the layer that would have failed first (usually unit).
- After creating, list the new IDs back to the user and offer to open a fix PR.

## Test-design heuristics

For each root cause, propose tests across the pyramid, cheapest first:

- **Pure-function/unit** for the exact transform that broke (the highest-value,
  fastest guard — usually the one that was missing).
- **API/integration round-trip** asserting the user-visible contract end to end
  within one service.
- **Boundary/contract** test for what one service sends to another (mock the
  far side, assert the call shape).
- **Acceptance table** turning the work item's acceptance criteria into a
  parametrized unit/API test so the spec is enforced at a cheap layer.
- **Negative path** for the failure mode the feature exists to handle.

## Notes

- Project repos live under the workspace root (`repo-*`); each may have its own
  `AGENTS.md` and toolchain caveats.
- If no formal Test Plan exists, Test Cases linked via `Tested By` to the work
  item is the established pattern here — don't require a plan/suite.
