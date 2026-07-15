---
name: assess-test-quality
description: >-
  Assess the quality of existing unit tests against the real behavior of the code
  under test, then propose concrete, evidence-backed improvements to the project's
  test-authoring rules and skills (e.g. the write-unit-test skill and the
  test-authoring rule). Use when asked to evaluate test coverage or quality, judge
  how well a test suite exercises a unit, reflect on how a test-writing agent
  performed, or improve the rules/skills that guide test writing.
disable-model-invocation: true
---

# Assess Test Quality

Judge how well a unit test file actually exercises the code under test, then turn the
gaps you find into specific edits to the rules and skills that produced it. The job has
two halves: **score the tests against ground truth**, and **fix the guidance so the next
test is better** — never one without the other.

## Operating principles

- **Ground truth is the code, not the ticket.** Coverage is measured against the
  behaviors and branches of the unit under test, not against the ADO step list. A suite
  that maps every ADO step but misses half the branches is incomplete.
- **Evidence or it didn't happen.** Every gap and every proposed rule change must cite a
  concrete `file:line` — a branch in the code with no test, or an assertion the test
  skipped. No vibes-based scoring.
- **Every proposed edit traces to an observed gap.** Do not add guidance for problems the
  suite didn't actually have. Each rule/skill change names the specific miss it prevents.
- **Find the root cause, not just the symptoms.** Several missed cases usually share one
  cause (e.g. guidance over-fit to one example unit shape). Fix the cause.
- **Static by default.** Toolchains here are not always reachable (see each repo's
  `AGENTS.md`). Read and reason about tests statically; if no SDK is present, say runtime
  confirmation is pending rather than guessing pass/fail.
- **Propose, then apply on confirmation.** Present the assessment and the proposed edits
  first. Apply edits to rules/skills only after the user agrees.

## Inputs

Collect before starting:

1. **Test file(s)** to assess — required.
2. **Code under test** — the class/service the tests target (derive from the test file if
   not given).
3. **Guidance artifacts** to critique — the test skill and rule that should have shaped
   the suite. Default targets in this workspace:
   - `repo-charge-edge/.cursor/skills/write-unit-test/SKILL.md`
   - `repo-charge-edge/.cursor/rules/test-authoring.mdc`

If the test file is missing, ask for it before proceeding.

## Workflow

Copy this checklist and track progress:

```
- [ ] 1. Establish ground truth (behaviors + branches of the unit)
- [ ] 2. Score the tests (scorecard + behavior-coverage matrix)
- [ ] 3. Diagnose gaps and trace each to a root cause
- [ ] 4. Propose rule/skill improvements (grounded wording per gap)
- [ ] 5. Apply on confirmation, then optionally validate by re-running
```

### Step 1: Establish ground truth

Read the code under test and enumerate, with `file:line`, every behavior a thorough suite
owes a test:

- **Happy path(s)** — each documented success behavior.
- **Branches** — every `if`/early-return/`catch`, including config and environment
  branches (e.g. an env var set vs unset), and empty/null inputs.
- **Boundaries** — exact comparison edges (`>` vs `>=`), off-by-one traps.
- **State transitions** — for stateful units, set → clear → re-arm, not just first trigger.
- **External contracts** — for units that call another service, the endpoint, method, and
  serialized payload shape (field names + values) that cross the wire.
- **Suspected bugs** — lines that look wrong (e.g. state mutated *before* the I/O that
  confirms it succeeds), which a rigorous suite should pin with a failing or skipped test.

This enumerated list is the denominator for everything below. Do not derive it from the
test names — derive it from the code.

### Step 2: Score the tests

Produce two tables (the same format the `compare-test-agents` skill uses, so results are
comparable across runs).

**Criteria scorecard** — counts where it's a count, ✓ / — where it's yes/no:

```markdown
| Criterion | Value |
|-----------|:---:|
| Total tests | _ |
| ADO steps → explicit assertions | _ |
| Negative / error branches covered | _ |
| Boundary cases | _ |
| Config / env branches covered | _ |
| State-transition cases | _ |
| External contract (endpoint/method/payload) asserted | _ |
| Suspected bug pinned by a test | _ |
| Suspected bug flagged in summary | _ |
| Determinism + global-state isolation | _ |
| Ran green / statically verified | _ |
```

**Behavior-coverage matrix** — one row per behavior enumerated in Step 1; mark ✓ / — :

```markdown
| Behavior (file:line) | Covered |
|----------------------|:---:|
| <behavior 1> | _ |
| <behavior 2> | _ |
```

### Step 3: Diagnose gaps and trace to a root cause

For each `—` in the matrix, write one line: the missing behavior, its `file:line`, and why
it matters (what real regression would slip through). Then group the gaps and name the
**root cause** — most clusters trace to one of:

- Guidance **over-fit to one unit shape** (e.g. threshold/pure-logic advice applied to an
  I/O service), so the agent had no cue for the missing dimension.
- A **satisficing instruction** ("at least one negative case") that invites stopping early
  instead of enumerating every branch.
- A **missing dimension** entirely (no mention of external contracts, env-var isolation,
  or bug-flagging).

### Step 4: Propose rule/skill improvements

For each root cause, propose a concrete edit to the skill or rule — quote the current text
and the replacement wording, and tie it to the gap it closes. Keep edits surgical and
preserve existing worked examples. Good proposals are specific enough to paste in:

- Broaden a narrow rule (e.g. extend a threshold-only bug hint into a short bug taxonomy
  that includes I/O ordering and config branches).
- Replace satisficing language ("at least one") with enumeration ("cover every branch").
- Add a missing requirement (external-contract assertions; env-var/global-state isolation;
  a bug-flagging principle with a `Skip = "BUG: ..."` pattern).
- Add a no-SDK static-verification fallback where a rule hard-requires running tests.

Decide placement by scope: cross-cutting standards go in the **rule**
(`test-authoring.mdc`); workflow steps and worked guidance go in the **skill**
(`write-unit-test/SKILL.md`). Note that the rule auto-attaches only via its `globs`, so it
shapes only files under the matching path.

### Step 5: Apply and validate

After the user confirms, apply the edits with precise replacements. Then, if useful,
validate by re-running the `compare-test-agents` skill on the same unit into fresh
directories and reporting the delta — did the previously missed behaviors now get covered?

## Anti-patterns

- Scoring against the ticket's step list instead of the code's branches.
- Proposing generic "write better tests" advice not tied to a concrete miss.
- Adding so much to a rule that it stops being actionable — prefer a few sharp requirements
  over a long wishlist.
- Encoding a suspected bug as the expected result; pin the *correct* behavior and flag it.
