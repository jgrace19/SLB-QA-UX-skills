# SLB QA & UX Skills (examples)

Example [Cursor Agent Skills](https://cursor.com/docs/context/skills). They show patterns you can set up for QA and UX workflows in your own environment.

**Treat these as templates.** Copy what is useful, then adapt tool names, repo layout, work-item systems (e.g. Azure DevOps), and team conventions to your project. They are not drop-in production skills for every codebase.

---

## QA skills (`qa-skills/`)


| Skill                           | What it does                                                                                                             |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| **assess-feature-impact**       | Reads an ADO work item and traces cross-repo blast radius with file-level evidence, then recommends tests for gaps.      |
| **assess-test-plan-gaps**       | Compares work-item requirements/acceptance criteria to linked test cases and reports what is untested.                   |
| **assess-test-quality**         | Scores existing unit tests against real code behavior and proposes concrete improvements to test-authoring rules/skills. |
| **classify-test-levels**        | Classifies ADO test cases into unit, integration, E2E, or manual and checks pyramid balance.                             |
| **triage-e2e-pipeline-failure** | Diagnoses a failing ADO E2E pipeline, attributes root cause, and recommends cheaper earlier-stage tests.                 |




## UX skills (`ux-skills/`)


| Skill                        | What it does                                                                                                      |
| ---------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| **personas-to-how-might-we** | Turns personas or research notes into design-oriented “How Might We” statements plus a short problem framing.     |
| **refine-prd**               | Turns a rough PRD into a structured, actionable product doc grounded in sources (without inventing requirements). |


---



## How to use

1. Browse a skill folder and open its `SKILL.md`.
2. Copy it into your own Cursor skills location (or team skill pack).
3. Rewrite references to ADO, MCP tools, repos, and process so they match your stack.
4. Iterate with your team until the skill matches how you actually work.

