# Useful AI Skills

A collection of reusable AI agent skills.

These skills are written to be portable across agent harnesses where possible. Each skill lives in its own folder and includes a `SKILL.md` file as the canonical instruction source.

## Skills

| Skill | Description |
| --- | --- |
| [`plan-review-loop`](./plan-review-loop/SKILL.md) | Reviews implementation plans with an autonomous planner/reviewer loop until the plan is approved or needs human input. |

## Installation

Clone this repository somewhere stable:

```sh
git clone https://github.com/akorede/useful-ai-skills ~/useful-ai-skills
```

Then copy or symlink the skill you want into your agent skills directory.

For Codex-style local skills:

```sh
mkdir -p ~/.agents/skills
ln -s ~/useful-ai-skills/plan-review-loop ~/.agents/skills/plan-review-loop
```

If your agent does not auto-discover skills, explicitly tell it:

```text
Read ~/useful-ai-skills/plan-review-loop/SKILL.md and follow it for plan review work.
```

## Structure

```text
useful-ai-skills/
  README.md
  <skill-name>/
    SKILL.md
    README.md
```

Each skill folder should be self-contained. Keep setup instructions, examples, and harness-specific notes in the skill folder README, while keeping `SKILL.md` focused on the reusable protocol.

## Conventions

- Use uppercase `SKILL.md`.
- Keep each skill in its own folder.
- Prefer platform-neutral instructions inside `SKILL.md`.
- Put installation notes, examples, and compatibility details in `README.md`.
- Avoid including project-specific secrets, private paths, or local-only assumptions.

## License

Choose a license before sharing broadly. MIT is a simple default for reusable instruction files.
