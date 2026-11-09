# Plan format

One file per piece of work: `.specshape/<id>.md`. Use only the sections the size calls for. Write
`n/a: <reason>` for a section that's required but doesn't apply.

## Frontmatter

```yaml
---
specshape: 0.1                  # plan format version (D24)
id: PROJ-500                 # ticket key or kebab-case slug
title: Multi-currency billing
source: <ticket URL, spec path, or "conversation">
size: initiative             # change | feature | initiative | project
status: draft                # draft → approved → shipped
version: 1
base_commit: 3f9c2e1         # HEAD when last written; omit if no commit is available
approved:                    # date, set on approval
tracker: PROJ-500            # the epic, once tickets exist
---
```

For a repo with no commits, or no Git repository, omit `base_commit` and explain the missing
baseline in the review response. Distinguish no Git repository from a repository with no commits;
a failed `HEAD` lookup alone does not establish which it is. Never invent a hash or create a commit.

## Sections, in this order

```markdown
# <id>: <title>

> Changed since v<n-1>: <what changed, or "first version">

## Summary
- **What changes:** <one or two lines, in the user's words>
- **Size:** <size>: <the deciding signals>
- **Footprint:** <n new, n changed, n removed files; n tasks>  or  <n stories>
- **Decisions needed:** <n open> (D1–Dn)
- **Top risk:** <the most consequential risk to requirements, supported by code or evidence>

## Decisions
### D1 <title> — open
<a meaningful design or behavior choice, with evidence for its context>
- **a)** <option>
- **b)** <option>

➡️ **Recommended: a.** <why>
⚠ **Consequence:** <what changes for users, callers or operations>

## Context changes
- **Glossary:** + **Money** — an amount in integer minor units plus an ISO currency code.
- **Contract:** + C4 Prices are stored in the currency they were set in; never converted on write.
- **Rule:** + No version numbers in class names.

## Shape                     (Feature, Project)
<Feature: one block per component, what it is and why, citing the code it builds on, with a sketch.>
<Project: folder tree, modules and their interfaces, data model, main flow diagram.>

## Affected areas            (Initiative)
- <area>: <what changes there> (`path:line`)

## Change map                (Change, Feature)
| | File | What | Size |
|---|------|------|------|
| + | `path/New.ext` | <purpose> | ~<n> lines |
| ~ | `path/Existing.ext` | <what changes> | ~<n> lines |
| - | `path/Old.ext` | <why it goes> | |

## Tasks                     (Change, Feature)
⬜ T1 <behavior slice: tests + implementation>    done when: <command> proves <outcomes>; suite green after refactor
⬜ T2 <next behavior slice>    needs T1   done when: <command> proves <outcomes>; suite green after refactor

**T1 test boundary:** <public interface>. **Behaviors:** <expected outcomes>.
**Method:** one failing test → minimum implementation → refactor green; repeat per behavior.

| Task | Files | Decisions |
|------|-------|-----------|
| T1 | `path/New.ext`, `path/NewTest.ext` | D1 |

## Stories                   (Initiative, Project)
### PROJ-500.1 <title> — not planned
**Goal:** <what's true when it ships>
**Acceptance:** <criteria>
**Needs:** <other stories, or none>

## Behaviour changes and risks
- ⚠ <what changes for existing users, callers or operations>
- **Out of scope:** <what this deliberately doesn't do>

## Alternatives rejected     (Feature and up)
- **<alternative>:** <why not>
```

Follow [testing.md](testing.md): each behavior task owns its tests and implementation, with a
public test boundary, expected outcomes and verification command. Do not split tests into a
later task. Done-checks must cover the task's behavior, not only unchanged behavior. Cite existing
tests you build on, and label future checks as proposed until run. Routine documentation or
validation choices need no decision entry unless a real trade-off needs review; use `—` in the
task's Decisions column when none applies. Include existing usage documentation for changed
public interfaces in the change map and tasks. Do not classify it as unrelated cleanup.

For Change and Feature reviews, show the complete plan body above, including the sketches and
task checks. Do not replace them with a prose summary or a link.

For Initiative and Project plans, use [review.md](review.md) before showing or approving the plan.
Check that decisions and story acceptance criteria agree, especially how modes are selected and
which constraints came from user answers. A story's name must reflect its actual scope.

## A story once it's planned

`/specshape-plan <story>` fills in the story under **Stories**:

```markdown
### PROJ-501 Money type + schema — planned
**Goal:** … **Acceptance:** … **Needs:** none

**Shape:** <components and sketches, citing the code as it is now>
**Decisions:** D4 Store amounts as integer minor units — decided (full entry in Decisions)
**Change map:** <table>

⬜ PROJ-501/T1 Money value type + behavioral tests       done when: money.test.ts proves amount invariants; suite green after refactor
⬜ PROJ-501/T2 currency column + migration   needs T1    done when: migration up/down test green
```

Story states: `not planned` → `planned` (→ `stale` when a plan decision changes before it's built).
Workflow status (in progress, done) lives in the tracker, not here.

## Written by other commands

`/specshape-plan` leaves these alone:

- **checkpoint lines**, added by `/specshape-build` to a task:
  `✅ T1 Money value type — commit a1b2c3, money.test.ts 6/6, no deviations`
- **check results**, added by `/specshape-check` under a plan or story:
  `Check (2026-11-04): passed, 0 drift`
