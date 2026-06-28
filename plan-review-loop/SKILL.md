---
name: plan-review-loop
description: Review an implementation plan with an autonomous planner/reviewer loop. Use when the user wants to iterate on a coding plan, validate a plan against repo evidence, review plan findings, update a plan from reviewer feedback, or continue until the plan is approved or needs human input.
---

# Plan Review Loop

Use this skill to run an implementation plan through an autonomous planner/reviewer loop before implementation. The goal is to converge on a plan that is grounded in the current codebase, project instructions, and user intent.

## Core Rules

- Use the best native capabilities available in the current harness for file inspection, code search, delegation, structured output, and progress reporting.
- Prefer an independent reviewer agent, session, or subtask when the harness supports one.
- If independent delegation is unavailable, perform a clearly separated reviewer phase in the same session.
- Do not blindly accept reviewer feedback. The planner must validate every finding against the plan, codebase evidence, and project instructions.
- Never expose private reasoning or raw agent chatter. Report concise findings, evidence, accepted/rejected decisions, plan diffs, and final results.
- Pause for the user only when product intent, unresolved ambiguity, or repeated evidence-backed disagreement requires human judgment.

## Roles

- **Orchestrator**: runs review rounds, reports progress, tracks finding state, and enforces stop conditions.
- **Reviewer**: independently reviews the plan against repo evidence, project conventions, tests, risk, and missing decisions.
- **Planner**: validates reviewer findings, accepts/rejects them with evidence, updates the plan, and returns it for review.
- **Human**: answers unresolved product, intent, or repeated evidence-backed disagreement questions.

## Workflow

1. Confirm the input includes the original goal and the implementation plan. If either is missing, ask for it.
2. Load relevant project instructions and inspect the codebase areas touched by the plan before reviewing.
3. Run up to 7 review rounds by default.
4. In each round:
   - Reviewer returns the reviewer JSON block.
   - Surface a concise progress update with round, phase, issue counts, suggestion count, whether the plan changed, and whether human input is needed.
   - Planner validates every finding and suggestion, updates the plan when appropriate, and returns the planner JSON block.
   - Surface another concise progress update.
5. Stop early only when reviewer approval has no blocking issues, important issues, nitpicks, or unresolved human questions.
6. Optional suggestions do not block approval by themselves, but the planner must accept, reject, or record each one.
7. If the same blocking or important finding is rejected twice and then reasserted with evidence, set `requiresHumanInput: true` unless the reviewer downgrades or withdraws it.
8. After 7 rounds without convergence, stop and report unresolved disagreements, remaining findings, and the latest plan.
9. For high-risk plans, perform one fresh final review after loop approval. The fresh reviewer sees only the original goal, final plan, and repo context, not the earlier debate.
10. If the fresh final review finds a blocking or important issue, reopen the loop with that issue as a new reviewer finding or pause for human input if it contradicts the approved loop outcome or depends on product intent.
11. If the fresh final review finds only nitpicks or optional suggestions, the planner must handle them using the normal accept, reject, or record rules before final output.

Treat a plan as high-risk when it is security-sensitive, migration/data-changing, cross-layer architecture, auth, payments/financial calculations, or broad refactor work.

## Structured Output Rules

- Emit parseable JSON blocks for reviewer and planner phase outputs.
- Do not emit prose-plus-JSON for phase outputs.
- Arrays must default to `[]`; do not use `null` for list fields.
- Finding IDs must be stable across rounds when the same issue is re-raised.
- Severity values must be exactly `blocking`, `important`, or `nit`.
- Evidence must be one or more of:
  - `file:line`
  - command/check output summary
  - project-doc reference
  - explicit codebase observation

## Reviewer Output

Return this JSON shape:

```json
{
  "round": 1,
  "approved": false,
  "requiresHumanInput": false,
  "blockingIssues": [],
  "importantIssues": [],
  "nitpicks": [],
  "optionalSuggestions": [],
  "questionsForHuman": [],
  "summary": ""
}
```

Each finding in `blockingIssues`, `importantIssues`, or `nitpicks` must use this shape:

```json
{
  "id": "R1",
  "title": "",
  "severity": "blocking",
  "evidence": [],
  "reason": "",
  "suggestedPlanChange": "",
  "status": "new"
}
```

Finding `status` must be one of:

- `new`
- `reasserted`
- `downgraded`
- `withdrawn`
- `resolved`

Each optional suggestion must use this shape:

```json
{
  "id": "S1",
  "title": "",
  "reason": "",
  "suggestedPlanChange": "",
  "optional": true
}
```

Questions for the human must be specific and decision-changing:

```json
{
  "id": "Q1",
  "question": "",
  "whyItMatters": "",
  "options": []
}
```

## Planner Output

Return this JSON shape:

```json
{
  "round": 1,
  "acceptedFindings": [],
  "partiallyAcceptedFindings": [],
  "rejectedFindings": [],
  "acceptedSuggestions": [],
  "rejectedSuggestions": [],
  "recordedSuggestions": [],
  "questionsForHuman": [],
  "updatedPlan": "",
  "validationNotes": [],
  "readyForReview": true
}
```

Each validation note must identify the finding or suggestion, the decision, the evidence, and the plan change or rejection reason:

```json
{
  "id": "R1",
  "decision": "accepted",
  "evidence": [],
  "reason": "",
  "planChange": ""
}
```

## Progress Updates

After each reviewer or planner phase, report one concise status update. Use this shape in prose:

```text
Round 2 reviewer: 0 blocking, 1 important, 1 nit, 2 optional suggestions. Human input: no.
Round 2 planner: accepted 1, rejected 1, recorded 1 suggestion. Plan changed: yes. Human input: no.
```

Do not include hidden reasoning, raw transcripts, or long debate logs in progress updates.

## Final Output

When the loop stops, provide a compact final report with:

- final verdict
- short summary
- major issues found
- how each major issue was resolved
- optional suggestions accepted, rejected, or recorded
- rejected findings and reasons
- fresh final review result, if performed
- remaining findings, if any
- final updated plan

Use this JSON shape internally or when a structured final answer is requested:

```json
{
  "finalVerdict": "approved",
  "roundsCompleted": 1,
  "summary": "",
  "majorIssuesResolved": [],
  "rejectedFindings": [],
  "suggestionsHandled": [],
  "remainingFindings": [],
  "humanQuestionsAsked": [],
  "freshFinalReview": {
    "performed": false,
    "result": "",
    "issues": []
  },
  "finalPlan": ""
}
```

## Activation

This skill is written to be platform-neutral. Treat the skill folder containing this file as the canonical source of truth. Do not assume every harness can auto-discover it.

Use the best activation path available in the current harness:

- native skill discovery
- symlink from a harness-specific skill directory
- plugin or tool registration
- project or global instruction reference
- explicit prompt to read and follow this file

Keep harness-specific setup recipes outside this skill body so the reusable protocol does not drift.
