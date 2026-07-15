# assess-feature-impact — Reference

Detailed material for the `assess-feature-impact` skill: the landmine checklist,
ADO tool snippets, the report template, and a worked example.

## Landmine checklist

Grep for these patterns in **every repo**. Each is a place where a "harmless"
change in one repo silently breaks another. For each match, capture `file:line`.

| # | Landmine | What to grep | Why it breaks |
|---|----------|--------------|---------------|
| 1 | **Closed schema** | `"additionalProperties": false` in `contracts/*.schema.json` | A new field added by a producer makes every consumer that validates against the schema reject the whole payload. |
| 2 | **Required fields** | `"required": [ ... ]` in schemas | Removing/renaming a field, or making an optional field required, invalidates existing data. |
| 3 | **Exact-string matching** | `WHERE version = ?`, `== desired_version`, `=== version`, `quote(version...)` | Identifier/format changes (versions, IDs, enums) silently miss on exact lookups — 404s, "degraded", "not found". |
| 4 | **Schema-derived / hand-mirrored types** | structs/records that mirror a contract (Go structs, C# records, TS interfaces) | A contract field change must ripple to every mirrored type; silent drift otherwise. |
| 5 | **Polled / SSE endpoints** | `desired-version`, `/telemetry/live`, poll loops in `simulator` | Changing a polled contract changes behavior across the poll boundary without a direct call. |
| 6 | **Store-and-forward / buffering** | `StoreAndForward`, `buffered`, `INGEST_URL` | If a downstream sink starts rejecting, the producer buffers indefinitely and looks "offline". |
| 7 | **Derived health / status** | `compute_health`, `reportedFirmwareVersion`, `lastSeenTs` | Health is computed from downstream data; a break upstream shows as a station health regression, not an error. |
| 8 | **Shared E2E** | `repo-quality-hub/e2e/tests/*.spec.ts` | The one test that exercises the whole loop — almost any cross-repo break fails it. |

## Dependency-seam sources (read these first)

| Source | What it tells you |
|--------|-------------------|
| `repo-quality-hub/contracts/architecture-map.md` | Named cross-repo dependency edges (#1 telemetry schema, #2 registry compatibility). |
| `docker-compose.yml` | Service wiring via env vars: `INGEST_URL`, `DEVICE_REGISTRY_URL`, `EDGE_URL`, `FLEET_URL`, `SCHEMA_PATH`. |
| `repo-quality-hub/contracts/*` | Source of truth: `telemetry.schema.json` + four `*.openapi.yaml`. |
| Each repo's `AGENTS.md` / `README.md` | Documented behaviors and known second-order effects. |

## ADO tool snippets

Read a tool's descriptor under
`mcps/user-ado/tools/<tool>.json` before calling if unsure of arguments.

**Read the work item (with linked tests):**
```
CallMcpTool(server="user-ado", toolName="wit_get_work_item", arguments={
  "project": "FieldEng",
  "id": <ID>,
  "expand": "relations"
})
```

**Read a linked test case** (resolve each `TestedBy` relation's id):
```
CallMcpTool(server="user-ado", toolName="wit_get_work_item", arguments={
  "project": "FieldEng", "id": <TEST_CASE_ID>
})
```

**Post the impact summary as a comment (on confirmation):**
```
CallMcpTool(server="user-ado", toolName="wit_add_work_item_comment", arguments={
  "project": "FieldEng", "workItemId": <ID>, "comment": "<markdown summary>"
})
```

**Create a recommended test case linked to the work item (on confirmation):**
```
CallMcpTool(server="user-ado", toolName="testplan_create_test_case", arguments={
  "project": "FieldEng",
  "title": "<test title>",
  "priority": 2,
  "testsWorkItemId": <ID>,
  "steps": "1. Step|Expected\n2. Step|Expected"
})
```
Steps format: `1. action|expected\n2. action|expected`. Use `|` only as the
step/expected delimiter; never inside step text.

## Report template (ADO comment body)

This is the comment posted to the work item, so keep it tight and scannable. Use
bullets, not wide tables; lead with the verdict; cite `file:line` inline. Aim for
something a reviewer reads in under a minute.

```markdown
**Cross-repo impact** — <N> repos affected: <X> breaking, <Y> silent/degrading.
Change surface: <one line — the concrete artifact this feature changes>.

**Impacts**
- 🔴 `<repo>` (breaking) — <what happens>. `path:line`
- 🟠 `<repo>` (silent) — <what degrades, no error>. `path:line`
- ⚪ `<repo>` (cosmetic) — <display-only effect>. `path:line`

**Test gaps → add**
- <impact> → <what the test should assert> (`<layer>`, no existing guard)
- <impact> → <what the test should assert> (`<layer>`)

**Sweep:** <repos + patterns searched, so "no impact" is auditable>
```

Notes:
- Omit any section that is empty (e.g. no cosmetic impacts → drop that bullet).
- If nothing is impacted, say so in one line and keep the **Sweep** line as proof.
- Keep the recommended test cases as the **Test gaps → add** bullets; the actual
  `testplan_create_test_case` calls happen only after the user confirms.

## Worked example: change the firmware version scheme in ChargeFleet

A realistic ticket that *sounds* self-contained but ripples upstream and
downstream. Use this as the model for depth and evidence.

**Work item (as written):** "Support pre-release / build-tagged firmware versions
(e.g. `1.3.0-rc1`, `1.3.0+arm64`) in the ChargeFleet deploy wizard so we can roll
out release candidates to a station before GA."

**Change surface:** the firmware **version string format** accepted by ChargeFleet
deploy — broadened from plain `MAJOR.MINOR.PATCH` to include pre-release/build tags.

**Impact trace (evidence-backed):**

| Repo | Direction | Severity | Evidence | What happens |
|------|-----------|----------|----------|--------------|
| `repo-platform-services/device-registry` | downstream | **breaking** | `src/firmware/firmware.store.ts:90-97` (`SELECT version FROM firmware WHERE version = ?` → `found: false`) | Compatibility is an exact-string lookup. `1.3.0-rc1` is not in the catalog, so the check returns not-found. |
| `repo-platform-services/device-registry` | downstream | silent | `src/firmware/firmware.store.ts:16-20` (seed catalog `1.0.0/1.2.0/2.0.0`) + `:74-76` (`ORDER BY version` string sort) | Catalog has no tagged versions, and version ordering is lexical, not semver — `1.3.0-rc1` sorts oddly relative to `1.3.0`. |
| `repo-charge-fleet` | upstream (caller) | **breaking** | `backend/compat.py:30-42` (builds `/firmware/{version}/compatibility`, treats `404` as "firmware version not found in registry") | Every pre-release deploy is rejected by Fleet's own compat gate before `desiredVersion` is ever set. |
| `repo-charge-fleet` | downstream | silent/degrading | `backend/ingest_health.py:61-65` (`if current == desired_version: healthy ... else: degraded`) | Health is an exact version-string compare. If Edge reports `1.3.0-rc1` but desired is normalized differently, the station shows **degraded** despite running correctly. |
| `repo-quality-hub/e2e` | downstream | **breaking** (if scheme used in tests) | `e2e/lib/services.ts:10` (`DEPLOY_VERSION = '1.2.0'`), `:117-126` (`waitForEdgeVersion` exact `=== version`) | The shared loop asserts exact version equality; a tagged version flows through unchanged and any normalization mismatch fails the assertion. |

**Why a naive author misses it:** the feature looks like a ChargeFleet UI/parsing
change. The breaks live in a *different* repo and language (the NestJS registry's
SQL lookup) and in a *silent* health computation, not in ChargeFleet's own code.

**Test-coverage gaps + recommendations:**

| Impact | Existing guard? | Recommended test | Layer |
|--------|-----------------|------------------|-------|
| Registry exact-match misses tagged versions | none | Nest spec: `checkCompatibility('1.3.0-rc1', model)` returns the intended result (after catalog/normalization handling) | unit (`firmware.store.spec.ts`) |
| Fleet rejects tagged version at compat gate | none | pytest: deploying `1.3.0-rc1` is accepted/normalized, not 404-rejected | unit (`backend/tests`) |
| Health degrades on version-format mismatch | none | pytest for `compute_health` with tagged `current` vs `desired` returns `healthy` when they refer to the same build | unit (`backend/tests`) |
| Whole loop with a tagged version | none | Playwright: deploy a pre-release version → Edge reports it → Fleet shows healthy | E2E (`repo-quality-hub/e2e`) |

**Recommended decision to surface to the author:** pick one canonical version
representation (normalize at the boundary) and apply it consistently in the
registry catalog, Fleet compat + health compare, and the E2E fixtures — otherwise
the format diverges per repo.
