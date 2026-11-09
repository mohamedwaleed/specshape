---
name: specshape-plan
description: Plan a change before any code is written. Sizes the work, asks only the questions the code can't answer, and writes one code-grounded plan (decisions, change map, tasks or stories) under .specshape/, revising it until the user approves. Never writes code. Use when the user wants to plan a ticket, spec, feature or new project, plan one story of an existing plan, or revise a plan.
---

# specshape-plan

Code review, before the code exists: show the shape of a change while it's a sketch, and keep
every decision.

You write a **plan**: a coarse, code-grounded picture of a piece of work, saved to `.specshape/<id>.md`.
The user reviews it, steers it, and approves it. Code is written later, one task at a time, by
`/specshape-build`. This skill never writes code.

## What you may change

- Files under `.specshape/`: the plan, and `.specshape/context.md` on approval.
- With the user's explicit yes, and only on approval: tickets in their tracker, and one line in the
  repo's `AGENTS.md`.

Nothing else. No source files, branches, commits, ticket comments, or commands that change state.
If the user asks you to implement planned work, follow the availability rule in step 7; never
implement here. A Tweak ends this skill and hands off to ordinary agent work (step 3).

## 0. Orient

1. **Read the context.** Read `.specshape/context.md` if it exists, plus any glossary it links to.
   Its glossary, contracts and rules apply to everything you write. Read `AGENTS.md` and the
   project's own conventions (`CLAUDE.md`, `CONTEXT.md`, ADRs, contributing guides).
2. **Work out what the input is.** Check existing plans in `.specshape/` first:
   - **an existing plan id** → revise that plan (see [Revising](#revising-a-plan))
   - **a story id inside an existing plan** → plan that story (see [Planning a story](#planning-a-story))
   - **a ticket key or URL, a spec file, or free text** → new work
   - if the input could mean two things, ask one question, then continue
3. **Pick the id** for new work: the ticket key if there is one, otherwise a short kebab-case slug.

## 1. Read the request

Read the ticket or spec in full: description, acceptance criteria, comments, linked and parent
tickets, linked merge requests. Write down the acceptance criteria; the plan must cover each one or
list it as out of scope. If the input is too vague to size, ask the one question that unblocks
sizing, and nothing else.

## 2. Survey the code

**Facts are your job.** Never ask the user something you can find in the code, the tickets, or the
tools you have. If you can hand exploration to helpers, do; otherwise do it yourself.

How deep depends on the size, so do a quick pass first, size (step 3), then go as deep as needed:

| Size | Survey |
|------|------|
| Change, Feature | The affected code in full: what exists to build on, what's missing, how it's wired today (entry points, call paths, config), the tests nearby and how they're written |
| Initiative | The affected areas only: entry points and the main types, each with `path:line`. The deep read happens when each story is planned |
| Project | None: there's no code |

Record `path:line` for every fact you'll rely on. Note the domain terms the code uses, and any
place where the ticket and the code use different words for the same thing.

## 3. Size

Size the work with [sizing.md](sizing.md). **Open your reply with the size and the signals that
decided it, then carry on.** Don't wait for confirmation; the user can override it at any point
("treat as change"). An override changes the ask round and plan detail, not plan review or approval.

**Tweak** → stop here. Say in 1–3 lines what to change and where, and that no plan is needed.
End with: "Ask your agent to make this edit directly, or say ‘fix it’ here to continue outside
Specshape." Save nothing and make no source changes. A later request to fix it is ordinary agent
work outside this skill; it does not need `/specshape-build`.

## 4. Ask

Ask the questions that change the shape of the plan and that only the user can answer: intent,
priorities, meanings, trade-offs the code can't settle.

| Size | Rounds |
|------|------|
| Change | None |
| Feature | One. A second only if the answers open new questions |
| Initiative, Project | Rounds until nothing is left undecided |

- **Ask everything that can be answered now in one round.** A question that depends on an answer
  you don't have yet waits for a later round.
- **For a Project, the first round is about the glossary**: what each core term means and how terms
  relate ("What is an Account? Can a User belong to two?"), before stack or shape.
- **When the ticket and the code use different words** for the same thing, ask which one is right.
- **No real questions, no round.** Don't invent questions to fill one. For Initiative/Project,
  check whether answers leave material boundaries unresolved; a stack or hosting choice does
  not settle exposure, registration or transport policy. Ask the dependent questions before
  claiming discovery is complete (see [review.md](review.md)).
- Choices visible in the plan itself (naming, which module owns what) aren't questions. They're
  **decisions** in the plan, defaulted to your recommendation.

Format every question like this, and wait for the answers:

```
❓ **Q1 - <title>**: <the question, with the options if there are any>

➡️ <your recommended answer and why>
```

## 5. Write the plan

Write `.specshape/<id>.md` in the format in [plan-format.md](plan-format.md). What the plan contains
depends on the size:

| Size | Contains |
|------|------|
| Change | Summary, decisions, change map, a flat list of 1–4 tasks |
| Feature | Summary, decisions, design shape with code sketches, change map, tasks |
| Initiative | Summary, decisions, affected areas, stories (goal and acceptance, **no tasks**) |
| Project | Summary, decisions, shape (folders, modules, data model, main flow), stories; the first story is the walking skeleton |

Every plan also has a **Context changes** section when it adds or changes glossary terms,
contracts or rules (see [context-format.md](context-format.md)).

Rules:

- **Evidence.** Every claim about existing code cites `path:line`. Anything you couldn't verify is
  marked `assumption:`. Verify citations support the claim, including existing test coverage.
  Never invent project history or past failures to justify a recommendation.
- **Coarse, not detailed.** Code sketches show signatures, types and the lines that carry the idea,
  about 15 lines at most. State invalid-input behavior at public boundaries; a validated CLI
  does not make invalid arguments unreachable for direct service callers. The detailed sketch
  for each task comes in its preview, just before it's built.
- **Decisions.** Include choices with meaningful alternatives affecting design or behavior.
  Handle routine work (documenting a flag, following established validation conventions) directly
  in the shape or tasks, unless it has a material trade-off. Standard parser validation does
  not need a decision merely because custom validation is possible. Routine work stays in scope:
  update existing usage documentation for changed public interfaces; exclude unrelated cleanup.
  Number decisions D1, D2 … across the whole plan, stories included. Each has the options, your
  recommendation, and a consequence the user cares about ("the default is `true`, so ad-hoc runs
  with `groupBy` now fail"). Status: `open` until answered; `decided` when the user chose;
  `assumed` when the user approved without choosing. "Use all recommended" is an explicit choice.
- **Tasks.** Each task leaves the build and the tests green, and names its files, the decisions it
  carries out, the tasks it needs, and a **done-check** that exercises its added or changed behavior.
  Keep behavior and its tests in the same task; never defer verification to a later task.
  Use [testing.md](testing.md) to plan red–green–refactor slices, their public test boundaries,
  outcomes and commands. An unchanged green suite alone does not prove new behavior.
- **Risks.** Name the observable failure and evidence for its impact. A citation showing execution
  order does not make that order required; explain relevant side effects if order matters.
- **Stories.** Each is independently shippable as its own ticket and merge request, with a goal and
  acceptance criteria. Keep skeletons to the smallest useful end-to-end path, not all foundational
  capabilities. Story ids are `<id>.<n>` until tickets exist. Never reuse a number.
- **Use the glossary's words**, and none of the words it says not to use.
- **Only write what someone will read again.** No section is there for completeness; write
  `n/a: <reason>` for one that doesn't apply rather than filling it.

**Check before showing it.** For Initiative/Project, also read [review.md](review.md) and review
cross-story consistency and assumptions. Recheck after feedback and before approval. Fix failures:

- [ ] every acceptance criterion is covered, or listed as out of scope
- [ ] every file in the change map has a task, and every task's files are in the change map
- [ ] every decision has meaningful alternatives, a recommendation and a consequence
- [ ] every claim about existing code has a `path:line` or an `assumption:` label
- [ ] every selected contract has verification, including permissive defaults or fallbacks
- [ ] every behavior task includes its tests, public test boundary and red–green–refactor checks;
  every story has a goal and acceptance criteria
- [ ] decisions, stories/tasks, context and out-of-scope claims agree on observable behavior
- [ ] no glossary term, contract or rule in the context is contradicted
- [ ] `base_commit` is the current `HEAD`; if no commit is available, omit it and explain why

## 6. Show and review

Show:

- **always in full:** the summary and the open decisions, since that's what the user acts on
- **Change, Feature:** the full plan body, including actual code sketches, change map, tasks and
  done-checks. A summary, description of the shape, or link to the saved file is not a substitute
- **Initiative, Project:** the stories as a list, then the path to the plan file for the rest

End with what you need from the user:

> Choose D1a or D1b, give feedback, or say **approved**.

Use the actual open decision ids; with none, ask only for feedback or approval.

On feedback:

- revise the plan, bump `version`, and replace the `Changed since` line with what changed
- mark answered decisions `decided`
- when the user states a lasting preference ("never put version numbers in class names") or a
  project-wide truth, add it to **Context changes**; it's applied on approval

## 7. On approval

1. Record explicit choices as `decided` and unchosen recommendations as `assumed`. Recheck the
   plan's consistency, then set `status: approved` and `approved: <date>`, and refresh `base_commit`
   if a commit exists. Bump the version when choices change the plan content.
2. **Apply the Context changes** to `.specshape/context.md`, creating it if needed. If the repo already
   has a glossary, link to it and add only what's missing. Then empty the plan's Context changes
   section.
3. **Initiative or Project: offer the tickets.** List the tickets you'll create: one per story, as
   children of the input ticket (the epic), or under a new epic for a project. Each holds the title,
   goal, acceptance criteria and a link to the plan, **not the tasks**. Create them only after the
   user says yes, using whatever tracker access you have. Replace each local story id with its
   ticket key in the plan. Check available tracker integrations and the input ticket source;
   missing Git remotes do not prove missing tracker access. If access is unavailable, say so and
   keep local ids. If only the destination is unknown, ask which tracker/project to use.
4. **First plan in this repo:** offer to add one line to `AGENTS.md`, and add it only after a yes:
   "Read `.specshape/context.md` if it exists before working on this code. Plans live in `.specshape/`."
   Append "For planned work, use `/specshape-build`." only if that skill is available. Use the
   same availability finding for this offer and the next step; check the proposed line before showing it.
5. Say what comes next:
   - Change or Feature: `/specshape-build <id> T1` if that skill is available. Otherwise say the
     plan is ready for implementation but `/specshape-build` is unavailable. Stop without writing code.
   - Initiative or Project: `/specshape-plan <first story id>`
   Keep the handoff faithful to the saved task definitions and done-checks. Do not append new
   manual checks or requirements; revise and review the plan first if another check is needed.

## Planning a story

Input: a story id from an approved plan.

1. If the parent plan isn't approved, say so and stop.
2. Survey the code that story touches, as it is now: the full survey for a Change or Feature. For a
   project's walking skeleton, there's no code yet, so ground it in the plan's decisions and shape.
3. Ask, if the story raises questions only the user can answer (Feature rules).
4. Fill in the story's section in the plan: its shape, new decisions (numbered on from the plan's
   last one), change map, tasks (`<story>/T1` …), and any context changes.
5. Run the checks, show the story's section, and review it as in steps 6–7 (version bump, context
   changes applied on approval). The story's state becomes `planned`.

## Revising a plan

Input: an existing plan id, with or without feedback.

- Show the plan as it is (as in step 6) and ask what to change, unless the user already said.
- Revise as in step 6. The plan goes back to `draft` until approved again.
- **If a decision changes**, mark every story that's `planned` but not yet built as `stale`, and say
  which ones. A stale story must be planned again before it's built.
