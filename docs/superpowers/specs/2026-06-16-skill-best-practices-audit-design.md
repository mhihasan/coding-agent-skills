# Skill Best-Practices Audit — Design Spec

**Date:** 2026-06-16  
**Scope:** All 12 pipeline skills in `skills/`  
**Goal:** Identify and fix every skill that violates the four best-practice dimensions below, so the repo ships a consistent, high-quality skill set.

---

## Problem

The pipeline has grown to 12 skills across multiple development sessions. There has been no systematic audit against a shared rubric. Known failure modes include:

- Descriptions that summarize workflow rather than stating when to trigger (causes models to skip reading the body)
- Missing or inconsistent frontmatter fields
- Skill bodies with vague steps, missing self-review gates, or prose that should be structured lists
- Pipeline handoff language that differs across skills (conflicting artifact paths, inconsistent `auto` flag semantics, missing PROCEED-marker references)

---

## Audit Dimensions

### D1 — Description quality

Each description must answer: **"When should the AI trigger this skill?"** — not "what does this skill do?"

A description that summarizes the workflow is an anti-pattern: the model reads it, thinks it understands the skill, and skips the body. The body is where the rigor lives.

**Pass criteria:**
- Contains concrete trigger phrases ("triggers on ...", "use when the user says ...")
- Does NOT narrate the workflow or list the skill's steps
- Does NOT duplicate the opening sentence of the body
- Is concise enough to scan quickly (under ~200 words)

### D2 — Frontmatter completeness

**Required fields:**
- `name` — must match the directory name exactly (casing, hyphens)
- `description` — required (see D1)

**Expected fields (warn if absent):**
- `model: inherit` — should be explicit to prevent accidental pinning to a stale model ID
- `color` — UI accent; not critical but missing = inconsistent
- `license` — missing on some skills

**Check for:**
- `name` mismatches vs. directory name
- Any undeclared/unknown fields
- Skills that pin a specific model ID (anti-pattern)

### D3 — Skill body quality

Each body should:
1. Open with a single sentence stating what the skill does
2. State workflow steps as an ordered list or labeled sections — not prose paragraphs
3. Include a self-review gate at each artifact boundary (for producing skills)
4. Not duplicate content already covered by the description
5. Use imperative style throughout ("Read the ticket", not "You should read the ticket")

**Check for:**
- Vague steps ("explore the codebase" without specifying what to look for)
- Missing self-review gates on skills that produce artifacts (plan files, commit sequences, ticket files)
- Prose-heavy sections that should be structured lists
- Placeholders or TBD sections
- Sections that describe WHAT the skill does rather than HOW to execute it

### D4 — Pipeline consistency

Skills that hand off to each other must agree on:

| Contract point | Expected |
|---|---|
| Artifact paths | `tickets/TICKET-KEY/TICKET-KEY.md`, `tickets/TICKET-KEY/PLAN-TICKET-KEY.md` |
| PROCEED marker format | `> **Plan Review:** PROCEED — YYYY-MM-DD` |
| `auto` mode invariants | No self-commit, no self-push, halt on BLOCKER, ask on unresolvable ambiguity |
| Handoff language | "next step is X" / "suggests X" must match the actual next skill name |
| Mode flag | Skills that support `auto` must document it; skills that don't must not imply it |

**Check for:**
- Conflicting artifact path references across skills
- `implementing-tasks` PROCEED-marker check wording vs. `reviewing-plan` append wording
- `auto` mode described differently (or missing) across skills that support it
- Post-skill "next step" prompts pointing to wrong or deprecated skill names

---

## Audit Execution

For each dimension, an independent subagent reads all 12 SKILL.md files and produces a findings table:

```
| Skill | Finding | Severity |
|---|---|---|
| skill-name | description of the issue | BLOCKER / SHOULD-FIX / NIT |
```

**Severity definitions:**
- **BLOCKER** — will cause model misbehavior (e.g., description causes body-skip, name mismatch breaks invocation)
- **SHOULD-FIX** — degrades quality or consistency without causing hard failures
- **NIT** — polish: inconsistent style, missing optional field, minor wording

Findings are consolidated, deduplicated across dimensions, and converted into implementation tasks. Each task targets one skill or one cross-cutting fix.

---

## Output Artifacts

| Artifact | Path |
|---|---|
| This design spec | `docs/superpowers/specs/2026-06-16-skill-best-practices-audit-design.md` |
| Implementation plan | `docs/superpowers/plans/2026-06-16-skill-best-practices-audit-plan.md` |
| Edited skill files | `skills/<name>/SKILL.md` (one edit per finding, committed atomically) |

---

## Out of Scope

- Book skills in `book-skills/` — not part of the pipeline, separate audit if needed
- `references/` files within skills — content quality of reference files is not audited here; only whether skills correctly reference them
- New skill creation — this audit covers existing skills only
- `install.sh` and tooling — infrastructure, not skills
