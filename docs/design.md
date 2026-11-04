# Previz design

**Show the change before making it, and keep every decision.**

This is the record of how Previz works and why. Each decision gives the reasons and the
alternatives that were rejected. Change a decision here before changing the skills.

## Commands and files

```
/previz                         status: plans and progress, suggests the next command   (saves nothing)
/previz-plan <input>            size → ask → plan → review → approved → tickets          (no code)
/previz-build <id> <task…>      preview → "go" → build → checkpoint                      (code)
/previz-check <id>              code vs plan → drift report                              (no code)

.previz/context.md              glossary, contracts and rules: what every agent reads first
.previz/<id>.md                 one plan per piece of work
```

## Decisions

### D1 Size first
Every run starts by sizing the work. The size decides how much process follows.

| Size | When | What Previz does |
|------|------|------|
| Tweak | 1–2 files, no new interface, no behaviour change, nothing to decide | Says what to change. No plan. |
| Change | One module, a handful of files, at most 2 decisions | Plan with a flat task list. No ask round. |
| Feature | Crosses modules, new interface, schema or contract change, behaviour change, or 3+ decisions; fits one session | Ask round, plan with design shape and tasks |
| Initiative | More than one session, or parts that ship separately | Ask round, plan with stories; each story planned when reached |
| Project | No codebase yet | Ask rounds, plan with shape and stories; the first story is a walking skeleton |

**Why:** spec-kit and grill-me give a one-file fix and a new service the same process.

### D2 Grounded in code
Every claim about existing code cites `path:line`. Anything not verified is marked `assumption:`.
Facts are the agent's job; it never asks the user something it can look up.

**Why:** plans written from intent alone don't fit existing codebases (spec-kit's brownfield problem).

### D3 Four commands; only build writes code
`/previz-plan`, `/previz-build`, `/previz-check`, and `/previz` for status.

**Rejected:** one stateful command that continues from saved state, because it would start
writing code depending on hidden state. One command per stage (size, ask, plan …), because the
read-only stages flow into each other and there would be too many to remember. A separate
`/previz-status`, because bare `/previz` does that job.

### D4 Build takes the task as its argument
`/previz-build <id> <task…>` builds only the tasks it's given, the way `/implement` takes its
input. It never picks the next task by itself. With no argument it lists the open tasks and asks.

**Why:** the user decides what gets built and when. The plan is a record, not a driver.

### D5 A preview happens only right before a task is built
Every task gets a preview: its files, a code sketch and its done-check, written against the code as
it is at that moment. The build waits for "go". The preview is shown, not saved.

**Why:** detail written just in time can't go stale. Check doesn't need the sketch; it compares the
code against the task's scope and the decisions.

**Rejected:** previews at project and story level as well. Those are plans.

### D6 Plans are coarse upfront; detail comes just in time
A plan holds the goal, decisions, shape and stories. A story's tasks are planned when you reach
it (`/previz-plan <story>`), against the code as it is then.

**Why:** spec-kit plans everything upfront, and those documents drift as soon as code exists.

### D7 One plan file per piece of work
`.previz/<id>.md` holds everything: goal, decisions, shape, stories, tasks, checkpoint lines,
check results. Plus `.previz/context.md` (D19). No folders and no other artifacts.

**Rejected:** spec-kit's separate spec, plan, research, data-model, contracts, quickstart and tasks
files. A folder per piece of work with a map file and a preview file per slice. Naming the plan
`AGENT.md`: too close to `AGENTS.md`, which agents load automatically as instructions.

### D8 Only save what someone will read again
Something is saved only if the user will read it at review, or an agent will read it at build
or check. The code is the record of the detail; the plan file is the record of the decisions.

**Why:** this is the guard against spec-kit's trap of documents nobody reads.

### D9 Epic → story → task
Big work is split into **stories**, each independently shippable as its own ticket and MR.
Stories are split into tasks.

**Why:** this is the hierarchy developers already know from their trackers.
**Rejected:** slice (unfamiliar), increment (jargon), milestone (suggests a date).

### D10 A ticket for each story
On plan approval, Previz lists the tickets it will create and waits for the user's yes. Stories
go under the input ticket (the epic); for a new project, under an epic it creates. A ticket holds
the title, goal, acceptance criteria and a link to the plan, **not the tasks**, which change while
building. Previz uses whatever tracker access the agent already has. Without any, it keeps local
ids and says so.

### D11 Ids
- A story's id is its ticket key (PROJ-501).
- Before a ticket exists, or with no tracker: `<parent>.<n>` (PROJ-500.1). Numbers are never
  reused, so references stay stable. The ticket key replaces the local id when it's created.
- Tasks are numbered within their story: `PROJ-501/T3`.
- A piece of work without a ticket gets a short kebab-case slug.

### D12 The tracker owns story status
Previz doesn't keep its own copy of story status. After a story's check passes, it offers to move
the ticket to done; if the user says "always", that goes into the rules in `context.md`.

### D13 Tasks are marked done by their done-check
Every task has a done-check (a test or command) and leaves the build green. `/previz-build` marks
the task done when its check passes, and shows that at the checkpoint.

### D14 Changes during a preview become decisions
If the user changes something in a task preview that affects other tasks, it's recorded as a new
decision, the plan moves to the next version, and Previz names the affected tasks and stories.

### D15 Revised decisions mark stories stale
When a plan-level decision changes, stories that are planned but not yet built are marked stale and
must be planned again before building.

### D16 Read-only except `.previz/`
`/previz-plan`, `/previz-check` and `/previz` write only under `.previz/`. The one exception is a
single line in the repo's `AGENTS.md`, added on the first run only if the user agrees: "Read
`.previz/context.md` before working on this code. Plans live in `.previz/`; for planned work, use
`/previz-build`".

### D17 Agent-agnostic
Skills are plain-markdown `SKILL.md` files and all state is plain files. No installer and no
per-agent adapters. The `AGENTS.md` line is how any agent finds Previz.

**Why:** spec-kit needs an integration for every agent.

### D18 Output
Plans are saved as markdown and shown in the terminal; an HTML view may come later. Code sketches
show signatures and key lines, not full diffs. Decisions are numbered and answered in free text.
Each one gives the options, a recommendation and the consequence.

### D19 A project context file every agent reads first
`.previz/context.md` has three sections: **Glossary** (each term, its definition, the words not to
use for it, how terms relate, and where it lives in the code), **Contracts** (truths that hold
across the project: invariants, formats, lifecycle rules; links to OpenAPI files or schemas rather
than copies), and **Rules** (the user's lasting preferences). The `AGENTS.md` line points to it.

**Why:** most design mistakes start with two people meaning different things by the same word.
This is the job the user wanted `AGENT.md` for, without clashing with the `AGENTS.md` standard.
**Rejected:** separate `glossary.md`, `contracts.md` and `rules.md` files: three pointers, three
things to forget.

### D20 How the context changes
- Only `/previz-plan` changes the context, through a **Context changes** section in a plan, applied
  on approval. Build and check flag mismatches (a name not in the glossary, a broken contract, a
  word the glossary says not to use) but never edit the context.
- A plan decision that makes something true project-wide moves into Contracts.
- If the repo already has a glossary (`CONTEXT.md`, `docs/glossary.md`, ADRs), `context.md` links to
  it and adds only what's missing. Never two sources of truth for the same terms.
- `context.md` is created only when a plan has something to put in it.

## `/previz-plan` decisions

### P1 Inputs
A ticket key or URL, a spec file, free text, an existing plan id (revise it), or a story id (plan
that story). Existing plans are checked first; an input that could mean two things gets one question.

### P2 No stop after sizing
The reply always opens with the size and its reasons, then carries on. For a Change, size and plan
come in one reply. For Feature and up, the ask round is the natural place to override the size.

### P3 Ask rounds by size
None for a Change; one round for a Feature (a second only if answers open new questions); rounds
until nothing is undecided for an Initiative or Project. Every question carries a recommendation.
No round when there are no real questions. For a Project, the first round is about the glossary.
When a ticket and the code use different words for the same thing, that's a question.

### P4 Survey depth by size
Change and Feature: the full survey of the affected code, its tests and conventions. Initiative:
the affected areas only; the deep read happens when each story is planned. Project: none. The skill
says what to find, never which tool to use.

### P5 Terminal output
The summary and open decisions always in full. Change and Feature: the rest of the plan too.
Initiative and Project: the stories as a list, plus the path to the file.

### P6 No plan history in the file
`version: n` and a "Changed since v(n-1)" line. Git holds the history.

### P7 Commit `.previz/`
It's committed with the code it describes, so plans go through MR review and any machine or agent
can pick up the work.

## Workflows

### Small feature (Change)

```
/previz-plan PROJ-612        size: change → plan (summary, decisions, change map, 2 tasks) → approved
/previz-build PROJ-612 T1    preview → go → build → checkpoint
/previz-build PROJ-612 T2    preview → go → build → checkpoint
/previz-check PROJ-612       code vs plan → shipped
```

Saved: `.previz/PROJ-612.md`, then a checkpoint line per task, then the check result.
A Tweak stops at sizing and saves nothing.

### Big feature (Initiative)

```
/previz-plan PROJ-500        size: initiative → ask → plan (goal, decisions, affected areas,
                             stories without tasks) → approved → creates PROJ-501…504
/previz-plan PROJ-501        plan that story: shape, decisions, tasks → approved
/previz-build PROJ-501 T1    preview → go → build → checkpoint   (repeat per task)
/previz-check PROJ-501       code vs story → check line, offers to move the ticket
/previz                      tree of stories, suggests /previz-plan PROJ-502
```

Saved: `.previz/PROJ-500.md`, which grows as each story is planned, built and checked.
A Feature that fits one session works like a Change plus an ask round and a design shape,
with no stories or tickets.

### New project

```
/previz-plan "habit-tracking API"   size: project → ask rounds → plan (goal, decisions, shape,
                                    stories) → approved → creates the epic and stories
/previz-plan HAB-2                  walking skeleton, grounded in the plan's decisions and shape
/previz-build HAB-2 T1 … / check    same loop as above
/previz-plan HAB-3                  from here on, grounded in the real code
```

Saved: `.previz/habits.md`. If a project decision changes, `/previz-plan habits` revises the plan
and marks planned-but-unbuilt stories stale.

### What each command saves

| Command | Saves |
|---|---|
| `/previz-plan <id>` | the plan file; tickets for stories (after confirmation); context changes in `context.md` on approval |
| `/previz-plan <story>` | that story's shape, decisions and tasks in the plan file |
| `/previz-build <id> <task…>` | code for those tasks; a checkpoint line per task; new decisions |
| `/previz-check <id>` | a check result; accepted drift as decisions; ticket status if the user agrees |
| `/previz` | nothing |

## Rules for this repo

- Change a decision here before changing a skill.
- Examples must not contain proprietary code or internal ticket ids.
