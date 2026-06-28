# Plan Review Loop

A reusable AI agent skill for reviewing implementation plans and specs before code is written.

The skill runs a structured planner/reviewer loop: one side critiques the plan or spec against repo evidence and project instructions, while the other validates the feedback, updates the document, and sends it back for another round.

The loop stops when the plan is approved, requires human input, or reaches the configured review limit.

## Use Cases

Use this skill when you want an AI agent to:

- review an implementation plan before coding
- review a spec before turning it into implementation work
- validate a plan or spec against the current codebase
- find missing tests, risky assumptions, or unclear decisions
- iterate until the document is ready for implementation
- separate reviewer critique from planner updates

## Files

```text
plan-review-loop/
  README.md
  SKILL.md
```

`SKILL.md` is the canonical instruction file. This README is only for humans.

## Installation

From a skills collection repo:

```sh
mkdir -p ~/.agents/skills
ln -s ~/useful-ai-skills/plan-review-loop ~/.agents/skills/plan-review-loop
```

Or copy the folder manually into your agent’s skills directory.

## Usage

If your agent supports skill discovery, ask it naturally:

```text
Use plan-review-loop on this plan.
```

```text
Use plan-review-loop on this spec.
```

You can also be more explicit:

```text
Use the plan-review-loop skill to review this implementation plan.
```

If your agent does not auto-discover skills, point it directly at the file:

```text
Read ./plan-review-loop/SKILL.md and follow it to review this implementation plan.
```

If the plan or spec is not in memory (perhaps created by a different agent or session), a good prompt should include both the original goal and the proposed plan or spec:

```text
Original goal:
<what we are trying to build or change>

Plan or spec:
<the plan/spec to review>
```
You can also just pass it the file of the spec or plan if that exists. 

## Expected Output

The agent should produce:

- concise progress updates for each reviewer/planner round
- evidence-backed findings
- accepted, rejected, or recorded suggestions
- an updated final plan or spec
- a final verdict: approved, needs human input, or unresolved after review limit

## Notes

This skill is intentionally platform-neutral. It does not assume a specific AI agent, IDE, repo layout, or tool harness.

For best results, use it in repositories where the agent can inspect files, search the codebase, and read project instructions.
