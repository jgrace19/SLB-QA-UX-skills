---
name: refine-prd
description: >-
  Turn a rough or vague PRD into a polished, actionable product document. Reads
  an existing rough PRD (and any available research, interviews, and the actual
  codebase/product), produces a NEW PRD document structured with best-practice
  sections, fills each section from the source material without inventing facts,
  then asks the user a single batched round of targeted questions to resolve the
  remaining ambiguous or human-only decisions. Use when the user asks to refine,
  enhance, polish, flesh out, or "make actionable" a PRD, spec, or product brief.
---

# Refine PRD

Transform a rough PRD into an actionable one. Always create a NEW document; never
overwrite the rough draft (it is the "before" and may be referenced later).

## Operating principles

- Ground everything. Every requirement, persona, and metric must trace to the
  rough PRD, the research/interviews, or the actual product/codebase. Do not
  invent requirements.
- Mark gaps explicitly. Where the source lacks information, write
  `[NEEDS INPUT: ...]` rather than guessing. These become the questions you ask.
- Derive before asking. Only ask the user for decisions a human must make (see
  "What to ask vs. derive"). Everything derivable, you derive.
- Tier by feasibility. If the product/codebase is available, tag each requirement
  as in-scope (data/capability exists) vs. deferred (capability missing), and put
  the deferred items in a phased "Out of scope / later" section.
- Be concise and concrete. Prefer specific, testable statements over adjectives.

## Workflow

```
- [ ] Step 1: Ingest sources
- [ ] Step 2: Map into the template (fill + mark [NEEDS INPUT])
- [ ] Step 3: Create the new polished PRD document
- [ ] Step 4: Ask one batched round of questions
- [ ] Step 5: Incorporate answers and finalize
```

### Step 1 - Ingest sources
Read, in order of priority:
1. The rough PRD provided by the user (the primary input).
2. Any research artifacts referenced or available (customer interviews, notes,
   support tickets, prior specs).
3. The actual product/codebase to ground feasibility (data models, existing
   features, what is and isn't already supported).

### Step 2 - Map into the template
Fill every section of the template below using the ingested sources. For each
user story, cite its evidence (which interview/quote or source). For anything not
supported by a source, insert `[NEEDS INPUT: specific question]`.

### Step 3 - Create the new polished PRD document
Write the filled template to a NEW location:
- If the rough PRD is a file, create a sibling file (e.g. `prd-<feature>.md`),
  do not edit the original.
- If publishing to a wiki/Confluence where pages can only be updated (not
  created) via tooling, create or request an empty page shell first, then write
  into it. Keep the rough PRD page intact.

### Step 4 - Ask one batched round of questions
Collect every `[NEEDS INPUT]` plus the human-only decisions and ask them in a
single batched round (use the AskQuestion tool if available; otherwise a single
numbered list). Do not drip questions across multiple turns. Provide a
recommended default for each where you have one.

### Step 5 - Incorporate answers and finalize
Fold the answers in, remove all `[NEEDS INPUT]` markers, confirm scope and
metrics are concrete and testable, and summarize what changed from the rough PRD.

## What to ask vs. derive

Derive (do not ask): problem statement, personas, user stories + acceptance
criteria, scope tiering from feasibility, draft success metrics, non-goals,
risks, and open questions already implied by the sources.

Ask (human-only): target values/thresholds for success metrics, final scope
cut confirmation, feature name/branding, prioritization tie-breaks, hard
timeline/rollout constraints, and any business or org constraints not in the
sources.

## PRD template

Produce the new document with these sections in this order:

```markdown
# PRD: [Feature name]

| Field | Value |
| --- | --- |
| Status | Draft / In review / Approved |
| Author | [name] |
| Stakeholders | [eng, design, data, ...] |
| Last updated | [date] |
| Related docs | [rough PRD, research, designs] |

## TL;DR
One paragraph: what we're building, for whom, and why now.

## Problem
The user problem and its impact, with evidence (cite research/quotes).

## Goals and non-goals
- Goals: the outcomes this feature must achieve.
- Non-goals: explicitly out of scope, to prevent scope creep.

## Success metrics
Each metric: name, how it's measured, and a target. Mark targets that need a
human decision as [NEEDS INPUT].

## Personas
The target users and what each cares about (cite research).

## User stories and requirements
Prioritized (P0/P1/P2). Each story:
- As a [persona], I want [goal], so that [reason].
- Acceptance criteria (Given/When/Then).
- Evidence: [source/quote].
- Feasibility: in-scope | deferred (why).

## Scope
- In scope (v1): [list]
- Out of scope / later phases: [list with rationale]

## Solution overview
Key user flows and the high-level approach. Reference design directions if any.

## Requirements and constraints
Cross-cutting requirements (e.g. accuracy, performance, privacy), dependencies,
and assumptions.

## Risks and mitigations
Top risks and how each is addressed.

## Rollout
Release approach (flags, phased rollout, beta), and how success is monitored.

## Open questions
Anything still unresolved after the question round.

## Appendix
Links and supporting references.
```

## Notes
- Keep the original rough PRD untouched; this skill is additive.
- If a section genuinely does not apply to the feature, keep the heading and write
  "N/A - [reason]" rather than deleting it, so reviewers see it was considered.
