---
name: sdlc-start
description: >
  Universal pipeline entry point. Use when starting any new work — accepts a
  free-form idea ("add dark mode"), a Jira ticket URL, a Jira key (PROJ-42),
  or a local ticket file path. Routes ideas to brainstorming and tickets to
  picking-up-task, then proceeds to planning-from-spec. Also use with no
  argument to resume in-progress work.
license: MIT
---

# /sdlc-start — Universal Pipeline Entry Point

> Before proceeding, read [`references/sdlc-start-usage.md`](references/sdlc-start-usage.md).
> It contains the exact detection rules, resume logic, active state schema, and mode details.

Single entry point for all new work. Detects input type and routes accordingly.

## Mode

Parse the argument list for `auto`. If present, **collaborative is the default**.

```
/sdlc-start PROJ-42          → collaborative
/sdlc-start PROJ-42 auto     → auto
/sdlc-start "add dark mode" auto → auto
```

Store the resolved mode and **pass it to every downstream skill invocation** — `picking-up-task`, `superpowers:brainstorming`, `planning-from-spec`, `generating-tasks`. Do not let any delegated skill default independently.

See `references/sdlc-start-usage.md` — "Modes" for what auto removes and what it never removes.

## Where You Sit in the Pipeline

```
[0] /sdlc-start         ← YOU ARE HERE
      │
      ├─ idea ──────────→ superpowers:brainstorming → local-dev/specs/<slug>-design.md
      │                                                         │
      └─ ticket/key/file → picking-up-task → ticket file       │
                                                                ↓
[1] planning-from-spec  ← converges here
[2] generating-tasks
[3] reviewing-plan
[4] implementing-tasks
[5] reviewing-code
[6] crafting-commits
```

## Step 1 — Detect input type

Apply detection rules in order (see `references/sdlc-start-usage.md` — "Input detection rules").

- **URL / Jira key / local file** → Ticket path (Step 2a)
- **Free-form text** → Idea path (Step 2b)
- **No argument** → Resume check (Step 2c)

## Step 2a — Ticket path

Invoke `picking-up-task <argument> [auto]` — pass mode flag if set. Do not duplicate fetch, branch, or review gate logic — `picking-up-task` owns all of it.

After it completes and the user approves, write `.agentic-sdlc/active/<KEY>.md` using the schema in `references/sdlc-start-usage.md` — "Active state file". Then invoke `planning-from-spec <ticket-file> [auto]`.

## Step 2b — Idea path

Invoke `superpowers:brainstorming <text> [auto]` — pass mode flag if set. Direct spec output to `local-dev/specs/YYYY-MM-DD-<topic>-design.md`. Do not re-implement the clarifying-question loop — the brainstorming skill owns it.

After brainstorming completes and the user approves, derive a slug from the spec filename. Write `.agentic-sdlc/active/<slug>.md` using the schema in `references/sdlc-start-usage.md` — "Active state file". Then invoke `planning-from-spec <spec-file> [auto]`.

## Step 2c — Resume check (no argument)

Read `.agentic-sdlc/active/`. Follow the resume logic in `references/sdlc-start-usage.md` — "Resume logic". When resuming, pass the stored mode (default collaborative unless `auto` was given).

## Step 3 — Converge at planning-from-spec

Both paths end at `planning-from-spec`. After it completes, continue with:

```
/generating-tasks <plan-file> [auto]
```

## You Must NOT

- Duplicate ticket-fetching, branch-creation, or brainstorming logic — delegate entirely
- Proceed past a no-argument case without checking `.agentic-sdlc/active/` first
- Accept ambiguous input silently — if detection is uncertain between rules, ask
- Invoke `planning-from-spec` before the upstream skill has completed and produced its output file
- Write to `.agentic-sdlc/active/` before the upstream skill completes and the user approves
