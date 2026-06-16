# agent-skills

Generic engineering craft skills for AI coding assistants.

Book-grounded, employer-neutral, and reusable by any engineer. Works with
Claude Code, OpenCode, Cursor, and any tool that reads `~/.claude/skills/`.

## Agentic Coding Workflow

These skills chain into a single feature-development pipeline — ticket in,
reviewed code out. This section is the single source of truth for the flow;
individual skills only reference their immediate neighbors, not the whole chain.

```
[0] using-git-worktrees ......... isolate before any code            (superpowers — front)
        │
[1] fetching-tickets ............ Jira → TICKET-KEY.md               (keep)
        └─ self-review: AC present, images local, section order, blocking deps surfaced
        │
[2] planning-from-ticket ........ ticket → PLAN-KEY.md               (keep)
        ├─ REQUIRED: superpowers:brainstorming        (already wired)
        ├─ ADOPT:    superpowers:writing-plans rigor   (no placeholders, exact commands)
        └─ self-review: no placeholders, decisions complete, scope tight, grounding verified
        │
[3] generating-tasks ............ plan → PLAN + "# Tasks"            (keep)
        ├─ ADOPT:    superpowers:writing-plans bite-sized-task discipline
        └─ self-review: AC coverage, behavioral tests, no orphan ACs, right-sized tasks
        │
[4] reviewing-plan .............. AI-as-judge verdict                 (keep — superior)
        ├─ fresh-context subagent on a strong model (bias guardrail)
        └─ emits verdict marker → PLAN file (implementing-tasks hard gate)
        │
[5] implementing-tasks .......... task spec → code via TDD            (wrap)
        ├─ HARD GATE: plan must have reviewing-plan verdict marker before starting
        ├─ REQUIRED: superpowers:test-driven-development  (Iron Law / cycle)
        │            + pytest-expert | vitest-react        (per-project conventions)
        ├─ ON RED-WRONG / ≥2 failed fixes: superpowers:systematic-debugging
        ├─ ON independent multi-failures:  superpowers:dispatching-parallel-agents
        ├─ BEFORE marking done:            superpowers:verification-before-completion
        ├─ MID-TASK: superpowers:requesting-code-review before each next task
        └─ ALT EXECUTION ENGINE:           superpowers:subagent-driven-development
        │
[6] reviewing-code .............. triage-first review                 (keep + layer)
        ├─ fresh-context subagents on a strong model (bias guardrail)
        ├─ ADOPT: superpowers:requesting-code-review  (BASE_SHA/HEAD_SHA diff convention)
        └─ ADOPT: superpowers:receiving-code-review   (verify-before-fix, no performative agreement)
        │
[6.5] crafting-commits .......... clean conventional-commit history   (MANDATORY — human-gated)
        ├─ self-review: file reconciliation, concern separation, script matches diff
        └─ developer triggers the rewrite — no auto-commit
        │
[7] finishing-a-development-branch  advisory close-out               (superpowers — advisory only)
        ├─ (1) verify tests green   (2) clean up worktree   (3) PRINT exact merge/PR commands
        └─ developer runs all git writes; skill never commits/pushes/merges/PRs
```

| Skill | What it does |
|---|---|
| `fetching-tickets` | Pull a Jira ticket to a local markdown file with all images downloaded |
| `planning-from-ticket` | Turn a local ticket/spec file into a reviewed implementation `PLAN-<KEY>.md` beside it |
| `generating-tasks` | Append TDD-ready task specs into the `PLAN-<KEY>.md` |
| `reviewing-plan` | Judge the PLAN+TASKS against the ticket *before* any code is written |
| `implementing-tasks` | Implement a task spec via TDD, auto-selecting the project's testing skill |
| `reviewing-code` | Triage-first code review of implemented code / a PR / a diff |
| `crafting-commits` | Rewrite branch commit history into clean conventional commits (human-gated) |

Each step is independently usable — you can enter at any point if the upstream
artifact already exists (e.g. run `reviewing-code` on any PR with no plan).

## Composes with superpowers

This pipeline is the **spine** (artifact-centric, Jira-native, resumable). The
[superpowers plugin](https://github.com/anthropics/claude-code) provides
**cross-cutting discipline grafted onto it** at key points (see diagram above).

| Layer | Owned by |
|---|---|
| Jira fetch, artifact files, plan review, triage-first code review | **this repo** |
| TDD Iron Law, debugging, verification gate, git worktrees, commit history, close-out | **superpowers** |

**The superpowers plugin is a required dependency for the full pipeline.**
Steps [0], the `superpowers:*` sub-skills in [2]–[6], and [7] require it.

Install in Claude Code:
```
/plugin install superpowers@claude-plugins-official
```
Or visit https://claude.com/plugins/superpowers. Then re-run `./install.sh` here.

### Review tiers

The pipeline uses two complementary review layers, split to avoid self-preference bias:

| Tier | Who | Scope | When |
|---|---|---|---|
| **Self-review** | The producing skill checks its own output | *Objective / mechanical* checks only (placeholders, file coverage, format) — verifiable yes/no | Every artifact boundary; runs in both modes |
| **AI-as-judge** | Independent fresh-context subagent on a strong model | *Subjective* quality calls (scope, over-engineering, breaking changes, design) with BLOCKER/SHOULD-FIX/NIT severity gate | `reviewing-plan` (before code) · `reviewing-code` (after code) |

Self-review is cheap and always runs. AI-as-judge is expensive and targeted. The split exists because a producer judging its own subjective quality is the strongest failure mode in AI evaluation (self-preference bias).

### Mode contract

Every pipeline skill accepts an optional `auto` argument. **Collaborative is the default.**

| Behavior | Collaborative | Auto |
|---|---|---|
| Forward-progress pauses (approve plan, confirm test plan, triage scope) | Pause for human | Proceed on own judgment |
| Git writes (commit/push/merge/PR) | Human-initiated | **Never self-initiated** (invariant) |
| Destructive overwrite of existing PLAN/ticket file | Ask | **Ask** (invariant) |
| Judge halt (DO NOT PROCEED / FAIL verdict) | Halt | **Halt** (invariant) |
| Unresolvable ambiguity | Ask | **Ask** (invariant) |

`auto` means "no forward-progress pauses" — not "self-ship." The git boundary and judge halts are invariants in both modes.

### Recommended model tiers

Skills keep `model: inherit` (honoring your session model). Judge subagents are dispatched with a strong model at dispatch time — not pinned in brittle frontmatter.

| Step | Role | Recommended tier |
|---|---|---|
| `fetching-tickets`, `generating-tasks` | Mechanical / extraction | Any capable model |
| `planning-from-ticket`, `crafting-commits` | Reasoning + writing | Default session model |
| `implementing-tasks` | TDD cycle | Default session model |
| `reviewing-plan` judge subagent | Subjective quality judgment | **Strong model** (e.g. `claude-opus-4-8`) |
| `reviewing-code` check subagents | Subjective quality judgment | **Strong model** (e.g. `claude-opus-4-8`) |

## Craft Skills

Standalone, book-grounded skills usable on their own or within the workflow above.

| Skill | Grounded in |
|---|---|
| `clean-architecture` | Robert C. Martin — *Clean Architecture* (2017) |
| `clean-coding` | Robert C. Martin — *Clean Code* (2008) |
| `ddd-expert` | Eric Evans — *Domain-Driven Design* (2003) |
| `design-patterns-expert` | Alexander Shvets — *Dive Into Design Patterns* (2022) |
| `design-doc-generator` | Generates production-grade architecture docs from a codebase |
| `pragmatic-engineer` | Thomas & Hunt — *The Pragmatic Programmer* (2019) |
| `system-designing` | Kleppmann & Riccomini — *Designing Data-Intensive Applications* (2nd ed.) |
| `pytest-expert` | Opinionated pytest best practices for Python |
| `vitest-react` | Unit testing for React + Vitest + TypeScript projects |
| `crafting-commits` | Analyzes a branch and rewrites commit history using conventional commits (human-gated) |

## Installation

```bash
git clone git@github.com:mhihasan/agent-skills.git
cd agent-skills
./install.sh
```

`install.sh` symlinks all skills into `~/.claude/skills/` (and any other
agent tool directories detected on your machine). Safe to re-run — existing
symlinks are updated, real directories are never overwritten.

