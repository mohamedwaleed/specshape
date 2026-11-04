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
base_commit: 3f9c2e1         # HEAD when the plan was last written; omit for a project with no code
approved:                    # date, set on approval
tracker: PROJ-500            # the epic, once tickets exist
---
```

## Sections, in this order

```markdown
# <id>: <title>

> Changed since v<n-1>: <what changed, or "first version">

## Summary
- **What changes:** <one or two lines, in the user's words>
- **Size:** <size>: <the deciding signals>
- **Footprint:** <n new, n changed, n removed files; n tasks>  or  <n stories>
- **Decisions needed:** <n open> (D1–Dn)
- **Top risk:** <the one thing most likely to go wrong or surprise someone>

## Decisions
### D1 <title> — open
<the question, with the context needed to answer it>
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
⬜ T1 <task>                                      done when: <test or command>
⬜ T2 <task>                        needs T1      done when: <test or command>

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

## A story once it's planned

`/specshape-plan <story>` fills in the story under **Stories**:

```markdown
### PROJ-501 Money type + schema — planned
**Goal:** … **Acceptance:** … **Needs:** none

**Shape:** <components and sketches, citing the code as it is now>
**Decisions:** D4 Store amounts as integer minor units — decided (full entry in Decisions)
**Change map:** <table>

⬜ PROJ-501/T1 Money value type                          done when: money.test.ts green
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
