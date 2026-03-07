---
name: task-state-sync
description: "Keep multitask continuity files accurate while work is in flight. Use when: (1) an agent is juggling multiple tasks across messages, (2) work should survive restarts or session resets, (3) TODO.md and memory/active-task.md need to stay aligned, (4) priorities, blockers, or next steps changed materially during execution."
---

# Task State Sync

Keep the task board honest. A brilliant plan with stale continuity files is just future confusion with better branding.

## Purpose

Use this skill to maintain the two key continuity files during live multitask work:

- `TODO.md` for the per-chat unfinished queue
- `memory/active-task.md` for the single top task that must resume first

This skill does not decide strategy. That belongs to `task-orchestrator`.

This skill decides when and how to write durable task state so restarts, resets, and long waits do not erase the plot.

## Ownership model

Treat the continuity files like this:

- `task-orchestrator` chooses ordering, parallelism, and progress rhythm
- `todo-continuity` defines the durable per-chat queue shape in `TODO.md`
- `restart-continuity` defines the top-task scratchpad in `memory/active-task.md`
- `task-state-sync` keeps those files updated during real work

If these files disagree materially, fix them before continuing complex work.

## When to sync state

Sync immediately when any of these happens:

- a new task meaningfully changes priority
- one task becomes the clear top task
- a blocker appears or clears
- a long-running background task starts
- important IDs appear: approvals, process IDs, session IDs, run IDs, job IDs, ports, or URLs
- the next concrete action changes
- a restart or reset becomes likely
- a task finishes and should be removed from the unfinished queue

Do not rewrite the files after every tiny step. Sync on material state changes.

## What to write where

### Update `TODO.md` when

- the current chat has unfinished or blocked work
- multiple tasks are active and the queue matters
- the next step or blocker changed
- a finished task should be removed from the board

Use `TODO.md` as the durable queue for the current chat, not as a global brain dump.

### Update `memory/active-task.md` when

- one task is the clear top priority
- the task is likely to survive the turn
- a restart or session reset would be painful without a scratchpad
- the exact next step or user-facing resume sentence changed

Use `memory/active-task.md` for the one task that must resume first, not for every side quest.

## Sync workflow

### 1. Rebuild the true state

Before writing, determine:

- what tasks are still active
- which task is top priority right now
- what is blocked vs merely waiting
- which IDs matter later
- what the next action actually is

Do not copy stale text forward blindly.

### 2. Write the per-chat queue

In `TODO.md`, keep the current chat section short and operational:

- `Context`
- `Goal`
- `In progress`
- `Next step`
- `Blockers`
- `Important IDs`
- `Resume message`

If the chat has no unfinished work, remove its section.

### 3. Write the top-task scratchpad

In `memory/active-task.md`, keep only:

- `Goal`
- `Current status`
- `Latest important IDs`
- `Next step`
- `If blocked`
- `User update after restart`
- `Done when`

If there is no active top task, clear the file instead of leaving fiction behind.

### 4. Verify consistency

After writing:

- the top task in `memory/active-task.md` should also make sense inside the chat section in `TODO.md`
- IDs should not contradict each other
- finished tasks should not still appear as active
- the recorded next step should still be the next step

## Restart-sensitive rules

When a restart is intentional and active work exists:

- update both files before restart
- if required by `restart-continuity`, schedule the one-shot fallback cron job
- record that cron `jobId` in `memory/active-task.md`
- after successful resumption, clear the stale restart fallback details

Never assume you will "remember later." That's not a system. That's gambling.

## Practical examples

### Example: new blocker appears

A background subagent run fails and produces a session ID plus an error log path.

Sync like this:

- update `TODO.md` so the current chat now shows the blocker and the next diagnostic step
- update `memory/active-task.md` if this failure becomes the top task
- record the session ID and log path in whichever file will matter after restart

### Example: top task changes

A later user message introduces an urgent production issue.

Sync like this:

- rewrite `TODO.md` to show the old task as secondary and the new task as the main active lane
- rewrite `memory/active-task.md` so the urgent issue becomes the resume-first task
- update the resume sentence so the first post-restart reply reflects reality

### Example: task completion

The repo publish finally succeeds.

Sync like this:

- remove the finished item from `TODO.md` or rewrite the section around remaining work
- if no active top task remains, clear `memory/active-task.md`

## Failure modes to avoid

- letting `TODO.md` and `memory/active-task.md` drift apart
- keeping stale IDs or finished tasks in either file
- recording vague next steps like "continue later"
- treating waiting as the same thing as blocked
- forgetting to update the user-facing restart sentence after priorities change
- leaving no state trail for long-running work that obviously spans turns
