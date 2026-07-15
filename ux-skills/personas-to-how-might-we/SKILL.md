---
name: personas-to-how-might-we
description: Turn a persona or user-research document into design-oriented "How Might We" statements plus a short problem/opportunity framing. Use when the user has personas, an interview synthesis, or research notes and wants HMW prompts for ideation (not a spec).
---

# Personas → How Might We

Convert a persona/research document into 5-6 "How Might We" (HMW) statements and a one-paragraph problem/opportunity framing. Output is for **ideation, not a spec** — keep it design-oriented, open-ended, and grounded only in the source.

## Inputs

- A persona document, user-interview synthesis, or research notes (a file path or pasted text).
- Optional: the feature/area the HMWs should focus on (e.g. "an AI chat feature"). If not given, ask once.

## Workflow

1. **Read the source in full.** Identify each persona's frustrations, jobs-to-be-done, mental models, and behaviors. Pull real pains, not paraphrased assumptions.
2. **Cluster pains into themes.** Group recurring or cross-persona frustrations. Aim for 5-6 distinct themes — each becomes one HMW.
3. **Write one HMW per theme.** Follow the rules below.
4. **Write the framing paragraph** (see template).
5. **Deliver in chat** as markdown (framing first, then the HMW list). Do not create files unless asked.

## Writing good HMW statements

- Start each with "How might we…".
- Frame a **problem/opportunity**, never a solution ("HMW help users trust an answer" — not "HMW add a citations panel").
- Keep the scope open enough to invite multiple ideas, narrow enough to be actionable.
- Ground each one in a specific pain from the source; make the user benefit explicit.
- Prefer the user's own language/mental models where it sharpens the frame.
- Produce **5-6** — enough to diverge, few enough to focus.

**Good:** "How might we let people get a single, direct answer to a plain-language question, so they don't have to do head-math?"
**Avoid (too solution-y):** "How might we add a natural-language search bar with autocomplete?"

## Problem/Opportunity framing

One paragraph (~4-6 sentences): what breaks down today, why it matters across the personas, the shared underlying need, and the opportunity the feature opens. Keep it evocative and design-oriented, not a requirements list. Do not invent facts the source doesn't support.

## Output template

```markdown
## Problem / Opportunity Framing

[one paragraph]

## How Might We statements

1. **How might we** … ?
2. **How might we** … ?
...
```

## Guardrails

- Only use pains and needs present in the source — no fabricated personas or data.
- If the source is thin or the target feature is unclear, ask one batched round of questions before writing.
