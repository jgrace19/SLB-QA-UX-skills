---
name: assess-feature-impact
description: >-
  Assess an Azure DevOps work item for cross-repo impact. Reads the work item,
  derives the workspace's dependency seams, traces upstream and downstream impacts
  of the proposed feature across all repos with file-level evidence, finds test
  coverage gaps, and reports a summary plus recommended test cases. Use when asked
  to assess feature impact, triage a work item for downstream effects, or find what
  else an ADO ticket might break.
disable-model-invocation: true
---

# Assess Feature Impact

Assess the cross-repo blast radius of a proposed feature described in an Azure
DevOps (ADO) work item. The job is to find the impacts a competent engineer who
only looked at their own repo would miss, then recommend the tests that would
catch them.

## Operating principles

- **Evidence or it didn't happen.** Every claimed impact must cite a real
  `file:line` found by reading or grepping the code. No speculative impacts.
- **"No impact" must be earned.** Only conclude there is no downstream effect
  after a documented full sweep of every repo, not by assumption.
- **Read-only on code.** Analyze statically. Recommend tests; never run builds or
  tests (toolchains here are not always reachable — see each repo's `AGENTS.md`).
- **Post the assessment; propose the rest.** Posting the summary comment on the
  work item is the final step. Still only *propose* test cases, and create them
  with `testplan_create_test_case` after the user confirms.

## Inputs

Collect before starting:

1. **Work item ID or URL** (e.g. `4` or an ADO `_workitems/edit/4` link) — required.
2. **Project** — default `FieldEng` unless the URL/user says otherwise.

If the ID is missing, ask for it before proceeding.

## Workflow

Copy this checklist and track progress:

```
- [ ] 1. Read the work item + existing linked tests
- [ ] 2. Identify the change surface
- [ ] 3. Derive the dependency seams
- [ ] 4. Trace impact across all repos
- [ ] 5. Analyze test-coverage gaps
- [ ] 6. Post the assessment comment on the work item
```

### Step 1: Read the work item

Fetch the work item and its relations:

```
CallMcpTool(server="user-ado", toolName="wit_get_work_item", arguments={
  "project": "FieldEng",
  "id": <ID>,
  "expand": "relations"
})
```

Extract: title, description, acceptance criteria, and the **already-linked test
cases** (the `Microsoft.VSTS.Common.TestedBy` relations). The linked tests are the
author's coverage baseline — what they already thought of. Read each linked test
case so you know what is NOT yet covered.

### Step 2: Identify the change surface

Translate the prose feature into the **concrete artifact(s)** it changes. This is
the most important step — most missed impacts come from a vague change surface.

Ask: does this feature add or change a…

- **shared data field** (telemetry record, DTO)?
- **schema / contract** in `repo-quality-hub/contracts/`?
- **API endpoint** (request/response shape, new route, status codes)?
- **identifier or format** (version string, ID scheme, units, enum values)?
- **config value** or environment variable?

Write the change surface as a short list of artifacts, e.g.
"new field `powerKw` on the telemetry record (schema `telemetry.schema.json`)".

### Step 3: Derive the dependency seams

Do not guess the dependency graph — read it. In priority order:

1. `repo-quality-hub/contracts/architecture-map.md` — explicit dependency edges.
2. `docker-compose.yml` — service env vars (`*_URL`, `SCHEMA_PATH`) reveal who
   calls/reads whom.
3. `repo-quality-hub/contracts/` — the declared single source of truth. **If the
   change surface touches anything here, the blast radius is automatically every
   service that reads that contract.**
4. Each affected repo's `AGENTS.md` and `README.md` — documented behaviors and
   second-order effects.

If these artifacts are absent (different workspace), fall back to a full-repo grep
sweep for the changed symbols.

Treat the architecture map as a hint and **verify against live code** — maps drift.

### Step 4: Trace impact across all repos

For each artifact in the change surface, grep **every repo** for producers and
consumers. Use `Grep` across the workspace root, not one repo at a time.

For each hit, classify and record with `file:line` evidence:

- **Direction** — upstream (produces/owns the artifact) or downstream (consumes it).
- **Severity** —
  - **breaking**: rejects data, fails a build, or fails an existing test.
  - **silent / degrading**: no error, but wrong or degraded behavior (buffering
    forever, stale health, mismatched version). These are the highest-value finds.
  - **cosmetic**: display-only.

Run the **landmine checklist** in [reference.md](reference.md) — the specific
patterns (closed schemas, exact-string matching, schema-derived structs, polled
endpoints, store-and-forward / health derivation) that turn a "harmless" change
into a break. Grep for each.

### Step 5: Analyze test-coverage gaps

Enumerate existing tests across stacks and the linked ADO cases:

- ChargeEdge: xUnit (`*.Tests`), `repo-charge-fleet`: pytest (`backend/tests`),
  Telemetry Ingest: `go test` (`*_test.go`), Device Registry: Nest spec
  (`*.spec.ts`), shared loop: Playwright (`repo-quality-hub/e2e`).

For each impact from Step 4, ask: **is there a test that would fail if this break
were real?** If not, it is a gap. Recommend a test at the right layer:

- contract/schema test, consumer unit test, cross-service integration test, or the
  shared E2E loop.

Explicitly recommend the **negative and cross-repo tests** that a happy-path suite
omits (e.g. "ingest still accepts the batch after the schema change", not just "the
field appears on the console").

### Step 6: Post the assessment comment

1. Build the report using the **compact comment template** in
   [reference.md](reference.md). Keep it short — this renders as an ADO work-item
   comment, not a doc. Bullets over wide tables; lead with the verdict.
2. Post it as a comment on the work item:

   ```
   CallMcpTool(server="user-ado", toolName="wit_add_work_item_comment", arguments={
     "project": "FieldEng", "workItemId": <ID>, "comment": "<markdown report>"
   })
   ```

3. Print the same report to chat, and note the work item it was posted to.
4. Then *offer* to create the recommended test cases linked to the work item
   (`testplan_create_test_case` with `testsWorkItemId: <ID>`). Only create them
   after explicit confirmation.

## Edge cases

- **Change surface touches no shared artifact:** still do the full grep sweep.
  "No cross-repo impact found" is a valid, useful result — state the sweep you ran.
- **Work item is vague:** if you cannot identify a concrete change surface, report
  what is ambiguous and ask the author to clarify before tracing.
- **Impact spans a stack you can't run:** keep it static; note that confirmation
  would require the relevant toolchain.

## Reference

- Landmine checklist, ADO tool snippets, report template, and a worked example
  (firmware version scheme change) are in [reference.md](reference.md).
