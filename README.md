# Task State Sync

English | [简体中文](README.zh-CN.md)

![Task State Sync banner](assets/social-preview.svg)

![OpenClaw Skill](https://img.shields.io/badge/OpenClaw-Skill-18324A?style=flat-square)
![Focus-State Sync](https://img.shields.io/badge/Focus-State%20Sync-8FD3C8?style=flat-square&labelColor=18324A)
![Works-Standalone](https://img.shields.io/badge/Works-Standalone-F6F4EE?style=flat-square&labelColor=355C7D)
![Artifact-.skill Included](https://img.shields.io/badge/Artifact-.skill%20Included-DCEFE9?style=flat-square&labelColor=355C7D)
![README-Bilingual](https://img.shields.io/badge/README-Bilingual-F6F4EE?style=flat-square&labelColor=B45F3C)
![License-MIT](https://img.shields.io/badge/License-MIT-F6F4EE?style=flat-square&labelColor=18324A)

An OpenClaw skill for keeping `TODO.md` and `memory/active-task.md` accurate while multitask work is still in flight.

## Overview

`task-state-sync` is a narrow continuity-maintenance skill for active work, not a general orchestration framework.

Its job is to keep the two operational continuity files aligned with reality while priorities shift, blockers appear, background runs start, and tasks complete:

- `TODO.md` as the durable per-chat unfinished queue
- `memory/active-task.md` as the single resume-first scratchpad

The skill is deliberately small. It does not choose scheduling strategy, manage broad workflow policy, or replace higher-level coordination skills. It exists to stop state drift before a restart, reset, or interruption turns active work into guesswork.

## Why this exists

Many agents can keep moving, but far fewer keep durable task state accurate while they move. The common failure pattern is boring and expensive:

- `TODO.md` falls behind the real queue
- `memory/active-task.md` points at the wrong top task
- important IDs are lost when they are first introduced
- finished work remains listed as active
- restarts convert a live task into reconstruction work

`task-state-sync` exists to make continuity-file maintenance explicit instead of accidental.

## Scope

Use this repo when the problem is task-state accuracy during execution.

Good fit:

- work spans multiple messages and the active queue changes
- priorities or blockers changed materially during execution
- important IDs need to be preserved for later recovery
- a restart or session reset would be painful without updated continuity files
- `TODO.md` and `memory/active-task.md` risk drifting apart

Not a fit:

- deciding scheduling policy
- general task prioritization frameworks
- restart automation by itself
- a broad multitask operating model

If the main problem is orchestration, use a companion repo. If the main problem is stale continuity files, start here.

## What the skill covers

`task-state-sync` teaches an agent to keep continuity files current at the moments that actually matter:

- sync on material state changes instead of every tiny step
- write the per-chat queue to `TODO.md`
- write the single resume-first task to `memory/active-task.md`
- record important IDs when they first appear
- distinguish blocked work from merely waiting work
- remove completed or stale items instead of preserving fiction
- prepare restart-safe state before resets or planned restarts

The repo stays intentionally narrow: it standardizes continuity-file upkeep, not the rest of the multitask system.

## Workflow summary

A typical `task-state-sync` pass looks like this:

1. Rebuild the true current state: active tasks, top priority, blockers, next action, and important IDs.
2. Update `TODO.md` so the current chat's unfinished queue reflects reality.
3. Update `memory/active-task.md` if one task clearly needs to resume first.
4. Remove stale finished work and keep recorded IDs consistent across both files.
5. Re-check that the next step, blocker state, and resume sentence still match the actual situation.

This is operational hygiene, but it is operational hygiene that determines whether recovery works.

## When to use it

Use `task-state-sync` when you need continuity files to stay trustworthy during live execution.

Typical triggers:

- a new task preempts the current top priority
- a blocker appears or clears
- a background run starts and returns IDs worth preserving
- a long-running task will need clean restart recovery
- the next concrete action changed
- a finished task should disappear from the active queue

## Representative outcomes

### Blocker capture

A background run fails and returns a session ID plus a log path.

A good agent should update `TODO.md`, decide whether that failure is now the top task, promote it into `memory/active-task.md` if needed, and preserve the recovery IDs immediately.

### Priority shift

A new urgent request arrives while other work is still in progress.

A good agent should rewrite the per-chat queue so the old task remains tracked but demoted, then rewrite `memory/active-task.md` so the new task becomes the resume-first lane.

### Completion cleanup

A previously active task finishes successfully.

A good agent should remove it from `TODO.md`, refresh the remaining queue, and clear `memory/active-task.md` if no top task remains.

## Related skill repos

These repositories are related examples, not required dependencies:

- `task-orchestrator`: scheduling, prioritization, and staged progress policy - <https://github.com/ruanrrn/task-orchestrator>
- `multi-task-continuity`: broader operating model that combines orchestration, state sync, and restart-safe continuity - <https://github.com/ruanrrn/multi-task-continuity>

Start here when the failure mode is stale state. Use the broader repos when you want the surrounding workflow policy as well.

## Install

Use either path:

1. Import `dist/task-state-sync.skill` into an OpenClaw environment.
2. Copy `task-state-sync/` into your skills directory if you want the editable source.

## What this repo contains

- `task-state-sync/` - the skill source
- `dist/task-state-sync.skill` - the packaged artifact ready to import
- `assets/social-preview.svg` - the repository banner and suggested social-preview asset

## Social preview

Suggested social preview asset: `assets/social-preview.svg`

Suggested one-line copy:

> Keep `TODO.md` and `memory/active-task.md` accurate while multitask work is still in flight.

GitHub note:

- The current `gh` CLI and GraphQL `UpdateRepositoryInput` do not expose a writable custom social preview field.
- To use this image as the repository social preview, upload `assets/social-preview.svg` manually in the repo settings UI.

## Repository layout

```text
task-state-sync/
├── LICENSE
├── README.md
├── README.zh-CN.md
├── CONTRIBUTING.md
├── assets/
│   └── social-preview.svg
├── task-state-sync/
│   └── SKILL.md
└── dist/
    └── task-state-sync.skill
```

## Contributing

See `CONTRIBUTING.md` for contribution scope, PR expectations, and the boundary that keeps this repo focused on continuity-file maintenance instead of broad orchestration policy.

## Release hygiene

- regenerate `dist/task-state-sync.skill` after each material skill change
- keep `README.md`, `README.zh-CN.md`, and `task-state-sync/SKILL.md` aligned
- preserve the repo's narrow role as the state-sync specialist of the family
- keep examples operational and honest rather than widening the repo's claims

## Repository

- GitHub: `https://github.com/ruanrrn/task-state-sync`
- License: MIT
