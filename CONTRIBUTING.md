# Contributing to Task State Sync

Keep contributions narrow, practical, and aligned with continuity-file hygiene.

## Scope

Good contributions:

- clearer sync rules for `TODO.md` and `memory/active-task.md`
- better examples of state drift, blocker updates, and completion cleanup
- README improvements that make standalone state-sync usage clearer
- packaging polish that does not widen the repo's purpose

Avoid:

- turning the skill into a scheduler or orchestration policy engine
- adding unrelated restart or cron systems that belong in other skills
- mixing in broad multitask strategy that is outside this repo's lane

## Workflow

1. Make the smallest useful change.
2. Keep README claims aligned with the actual skill behavior.
3. Regenerate `dist/task-state-sync.skill` after material skill changes.
4. Prefer concrete state examples over vague advice.

## Pull request guidance

A good PR should explain:

- what changed
- why the change improves state accuracy or recovery
- whether `dist/task-state-sync.skill` was regenerated

## Repo principle

This repo should remain the continuity-state specialist of the family, not the umbrella workflow.
