# Specshape design

**Code review, before the code exists.**

Specshape keeps engineers in control of the architecture while agents write the code.

## Positioning

**Code review, before the code exists.** Three pillars:

1. **Right-sized.** The process matches the change, judged from the code rather than the ticket,
   from a one-line fix to a new system. Small work isn't slowed down; big work isn't under-planned.
2. **Design before code.** Engineers make the design calls (names, patterns, seams, where logic
   lives) while the design is a sketch. The agent writes the code, and checks hold the code to
   what was approved. This matters most in large systems, where many people and agents change the
   same code and the architecture otherwise erodes one reasonable-looking MR at a time.
3. **Knowledge that compounds.** The glossary, contracts and rules in `.specshape/context.md` apply to
   every plan, by every engineer, in every agent, and approved plans add to them.

Supporting traits: grounded in existing code (`path:line`), agent-agnostic and file-based, light on files.

**Why this direction:** in a real migration planned with an early version of the preview, the
engineer's two design calls (replace a runtime `if` with a factory; drop version numbers from class
names) came only after the code existed, because the preview described the design in prose. Fixing
them meant restructuring a class and renaming every versioned class and package. Specshape has to
make the design visible early enough for those calls to happen before any code exists.

**The name.** Specshape: you see and approve the *shape* of a change before it's built, and "spec"
places it in the spec-driven development category. Chosen over Previz, the working name (a film
term most engineers don't know, and it pointed at the preview step only), and over Redline,
Trellis, Blueprint, Preflight, Specsight, Specline, Specward, Speclens, Specvista and Forespec
(taken on GitHub, too generic, or less to the point). The cost: commands are long
(`/specshape-plan`), and "spec" invites comparison with spec-kit, which the messaging answers with
"the shape, not documents".

**The real competitor** is the plan mode built into each agent. It lives in the chat, describes
the design in prose, doesn't learn the user's rules, doesn't check the code afterwards, and is tied
to one agent.

**Automation happens only in `/specshape-build`.** The plan is made once, with the engineer. Build is
where work runs without the engineer watching, stopping when something new comes up.

**Rejected:** positioning Specshape as an autonomy or trust tool (humans check decisions, machines
check code, autonomy levels). It drifted from the need Specshape started from: reviewing the design
of a change.

## About this document

This is the record of how Specshape works and why. Each decision gives the reasons and the
alternatives that were rejected. Change a decision here before changing the skills.

## Commands and files

```
/specshape                         status: plans and progress, suggests the next command   (saves nothing)
/specshape-plan <input>            size → ask → plan → review → approved → tickets          (no code)
/specshape-build <id> <task…>      preview → "go" → build → checkpoint                      (code)
/specshape-check <id>              code vs plan → drift report                              (no code)

.specshape/context.md              glossary, contracts and rules: what every agent reads first
.specshape/<id>.md                 one plan per piece of work
```

## Decisions

### D1 Size first
Every run starts by sizing the work. The size decides how much process follows.

| Size | When | What Specshape does |
|------|------|------|
| Tweak | 1–2 files, no new interface, no behaviour change, nothing to decide | Says what to change and directs the user to ask their agent to make the edit directly. No plan. |
| Change | One module, a handful of files, at most 2 decisions | Plan with a flat task list. No ask round. |
| Feature | Crosses modules, new interface, schema or contract change, behaviour change, or 3+ decisions; fits one session | Ask round, plan with design shape and tasks |
| Initiative | More than one session, or parts that ship separately | Ask round, plan with stories; each story planned when reached |
| Project | No codebase yet | Ask rounds, plan with shape and stories; the first story is a walking skeleton |

**Why:** spec-kit and grill-me give a one-file fix and a new service the same process.

### D2 Grounded in code
Every claim about existing code cites `path:line`. Anything not verified is marked `assumption:`.
Facts are the agent's job; it never asks the user something it can look up. Citations must
point to lines that support the claim, including claims about existing tests. Project history
needs evidence too; never invent a past incident to justify a recommendation. Git status and file
listings alone do not establish that contents match an earlier survey, especially with untracked
files or no commits. Reread affected files or compare their contents with a verified baseline.

**Why:** plans written from intent alone don't fit existing codebases (spec-kit's brownfield problem).

### D3 Four commands; only build writes code
`/specshape-plan`, `/specshape-build`, `/specshape-check`, and `/specshape` for status.

**Rejected:** one stateful command that continues from saved state, because it would start
writing code depending on hidden state. One command per stage (size, ask, plan …), because the
read-only stages flow into each other and there would be too many to remember. A separate
`/specshape-status`, because bare `/specshape` does that job.

### D4 Build takes the task as its argument
`/specshape-build <id> <task…>` builds only the tasks it's given, the way `/implement` takes its
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
it (`/specshape-plan <story>`), against the code as it is then.

**Why:** spec-kit plans everything upfront, and those documents drift as soon as code exists.

### D7 One plan file per piece of work
`.specshape/<id>.md` holds everything: goal, decisions, shape, stories, tasks, checkpoint lines,
check results. Plus `.specshape/context.md` (D19). No folders and no other artifacts.

**Rejected:** spec-kit's separate spec, plan, research, data-model, contracts, quickstart and tasks
files. A folder per piece of work with a map file and a preview file per slice. Naming the plan
`AGENT.md`: too close to `AGENTS.md`, which agents load automatically as instructions.

### D8 Only save what someone will read again
Something is saved only if the user will read it at review, or an agent will read it at build
or check. The code is the record of the detail; the plan file is the record of the decisions.

**Why:** this is the guard against spec-kit's trap of documents nobody reads.

### D9 Epic → story → task
Big work is split into **stories**, each independently shippable as its own ticket and MR.
Stories are split into tasks. A story must deliver a bounded, independently verifiable outcome.
A walking skeleton is the smallest useful end-to-end path through the system; do not bundle
all account, membership and administration work into it just because those are foundational.
Larger foundations may be separate stories, with dependencies stated rather than hidden.

**Why:** this is the hierarchy developers already know from their trackers.
**Rejected:** slice (unfamiliar), increment (jargon), milestone (suggests a date).

### D10 A ticket for each story
On plan approval, Specshape lists the tickets it will create and waits for the user's yes. Stories
go under the input ticket (the epic); for a new project, under an epic it creates. A ticket holds
the title, goal, acceptance criteria and a link to the plan, **not the tasks**, which change while
building. Specshape uses whatever tracker access the agent already has. Without any, it keeps local
ids and says so. Check connected tracker capabilities and the ticket source before concluding
access is unavailable; Git remotes do not establish tracker access. If access exists but the
tracker destination is unknown, ask where to create tickets, after presenting the proposed set.

### D11 Ids
- A story's id is its ticket key (PROJ-501).
- Before a ticket exists, or with no tracker: `<parent>.<n>` (PROJ-500.1). Numbers are never
  reused, so references stay stable. The ticket key replaces the local id when it's created.
- Tasks are numbered within their story: `PROJ-501/T3`.
- A piece of work without a ticket gets a short kebab-case slug.

### D12 The tracker owns story status
Specshape doesn't keep its own copy of story status. After a story's check passes, it offers to move
the ticket to done; if the user says "always", that goes into the rules in `context.md`.

### D13 Tasks are marked done by their done-check
Every task has a done-check (a test or command) and leaves the build green. `/specshape-build` marks
the task done when its check passes, and shows that at the checkpoint.
A planned done-check must exercise the behavior that task adds or changes; an existing green
suite alone cannot prove behavior it never tests. Add relevant verification in the same task,
and distinguish proposed checks from checks already run. Do not split a behavior's implementation
and its verification into separate tasks; use behavior slices, each with its own checks (D25).

### D14 Changes during a preview become decisions
If the user changes something in a task preview that affects other tasks, it's recorded as a new
decision, the plan moves to the next version, and Specshape names the affected tasks and stories.

### D15 Revised decisions mark stories stale
When a plan-level decision changes, stories that are planned but not yet built are marked stale and
must be planned again before building.

### D16 Read-only except `.specshape/`
`/specshape-plan`, `/specshape-check` and `/specshape` write only under `.specshape/`. The one exception is a
single line in the repo's `AGENTS.md`, added on the first run only if the user agrees: "Read
`.specshape/context.md` if it exists before working on this code. Plans live in `.specshape/`." Append
"For planned work, use `/specshape-build`." only when that skill is available (P9).

### D17 Agent-agnostic
Skills are plain-markdown `SKILL.md` files and all state is plain files. No installer and no
per-agent adapters. The `AGENTS.md` line is how any agent finds Specshape.

**Why:** spec-kit needs an integration for every agent.

### D18 Output
Plans are saved as markdown and shown in the terminal; an HTML view may come later. Code sketches
show signatures and key lines, not full diffs. Decisions are numbered and answered in free text.
Each one gives the options, a recommendation and the consequence. Reserve decisions for choices
with meaningful alternatives affecting design or behavior; routine documentation and established
validation conventions belong in the plan without a separate approval question. Using a parser's
standard validation is routine unless error wording or validation ownership has a material
trade-off. Recommend it directly rather than inventing an alternative just to fill a decision.
Routine supporting work still belongs in scope: document a changed public interface in the
existing usage documentation without asking whether to do so. Exclude unrelated cleanup.

**Why:** the first manual status-filter run asked the user whether to document a new flag,
adding ceremony without a useful design choice.

### D19 A project context file every agent reads first
`.specshape/context.md` has three sections: **Glossary** (each term, its definition, the words not to
use for it, how terms relate, and where it lives in the code), **Contracts** (truths that hold
across the project: invariants, formats, lifecycle rules; links to OpenAPI files or schemas rather
than copies), and **Rules** (the user's lasting preferences). The `AGENTS.md` line points to it.

**Why:** most design mistakes start with two people meaning different things by the same word.
This is the job the user wanted `AGENT.md` for, without clashing with the `AGENTS.md` standard.
**Rejected:** separate `glossary.md`, `contracts.md` and `rules.md` files: three pointers, three
things to forget.

### D20 How the context changes
- Only `/specshape-plan` changes the context, through a **Context changes** section in a plan, applied
  on approval. Build and check flag mismatches (a name not in the glossary, a broken contract, a
  word the glossary says not to use) but never edit the context.
- A plan decision that makes something true project-wide moves into Contracts.
- If the repo already has a glossary (`CONTEXT.md`, `docs/glossary.md`, ADRs), `context.md` links to
  it and adds only what's missing. Never two sources of truth for the same terms.
- `context.md` is created only when a plan has something to put in it. When a story adds terms,
  use the same canonical glossary as the parent plan; do not schedule a second definition in
  project docs after applying it to context. Source glossary edits stay outside the plan skill.

### D21 Three ways to install, one copy of each skill
- **Skills CLI:** `npx skills add <owner>/specshape`, the cross-agent installer (skills.sh). Our
  `skills/<name>/SKILL.md` layout is the layout it expects.
- **Claude Code plugin marketplace:** `/plugin marketplace add <owner>/specshape`, then
  `/plugin install specshape`, via two small manifests in `.claude-plugin/`.
- **Manual:** clone the repo and symlink `skills/*` into the agent's skills folder, so `git pull`
  updates it.

All three read the same `skills/` folder.
**Rejected:** our own installer (D17).

### D22 Installed per user by default; per project for teams
Per user (installed once, works in every repo) is the default. Teams are told to install per
project and commit it, so the Specshape version matches the `.specshape/` plans in the repo.

### D23 Skill names keep the `specshape-` prefix
Claude Code shows plugin skills as `/specshape:specshape-plan`. Renaming the skills to `plan`, `build` and
`check` would read better there, but every other agent would show a bare `/plan`, which clashes with
built-in commands and other skills.

### D24 Versions
Releases are git tags (`v0.1.0`) with a short `CHANGELOG.md`. Every plan records the format it was
written in (`specshape: 0.1` in its frontmatter), so later commands can warn about a plan written in an
older format.

### D25 Plan behavior slices for red–green–refactor
Behavior-changing tasks use TDD: one failing behavioral test, the minimum implementation to pass,
then refactoring with tests green, repeated one behavior at a time. Refactoring stays within the
approved scope and interfaces; a new design decision returns to the user.

The plan names the public boundary under test, expected outcomes and verification commands.
Plan approval confirms those test boundaries; it does not authorize implementation. Detailed test
code and the cycles happen during build, not planning. At a build checkpoint, report observed red
and green results and final checks; do not claim a cycle happened without running it.

Each task includes tests and production files needed for its behavior, and finishes green. Never
plan separate tasks for all tests and all implementation. Documentation-only work uses a relevant
check rather than a contrived failing test. If a behavior cannot be tested automatically, explain
the constraint and propose verification for review rather than silently claiming TDD.

**Why:** the second manual run put implementation before its comprehensive tests. The user wants
tests to drive implementation through red–green–refactor, while plans remain coarse and read-only.
**Rejected:** writing all tests up front; making red, green and refactor separate tasks; requiring
another test-boundary approval after those boundaries have already been approved in the plan.

## `/specshape-plan` decisions

### P1 Inputs
A ticket key or URL, a spec file, free text, an existing plan id (revise it), or a story id (plan
that story). Existing plans are checked first; an input that could mean two things gets one question.

### P2 No stop after sizing
The reply always opens with the size and its reasons, then carries on. For a Change, size and plan
come in one reply. For Feature and up, the ask round is the natural place to override the size.
A size override changes the ask round and plan detail, not the requirement to review and approve
the plan.

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
Initiative and Project: a story table with goals, acceptance summaries and dependencies, a
dependency tree, plus the path to the file (P12).
For Change and Feature, show the actual sketches, change map, tasks and done-checks, not a summary
or a file link in their place. This also applies to a planned story's Change/Feature section.

**Why:** the first manual run saved a design sketch but omitted it and the tasks from the review
response, leaving the user to approve a summary instead of the visible design.

### P6 No plan history in the file
`version: n` and a "Changed since v(n-1)" line. Git holds the history.

### P7 Commit `.specshape/`
It's committed with the code it describes, so plans go through MR review and any machine or agent
can pick up the work.

### P8 Repository baseline
Record the current `HEAD` as `base_commit`. If there is no commit (or no Git repository), omit
it and explain that no commit baseline is available. Distinguish a Git repository with no commits
from a directory outside Git; a failed `HEAD` lookup alone cannot establish which it is.
Do not create a commit to fill the field.

**Why:** the manual fixture had code but no commits; having code does not guarantee a `HEAD`.

### P9 Available next steps
Recommend `/specshape-build` only when it is available to the agent. Otherwise explain that
building is the next stage but that command is unavailable; leave the approved plan ready for
later implementation without writing code. Closing summaries must match the saved plan's tasks
and checks; do not add a manual check or requirement in the handoff. If a new check is needed,
revise and review the plan before approval. The first-run `AGENTS.md` offer must likewise avoid
directing agents to an unavailable command. Use the same availability finding for the next-step
response and the proposed `AGENTS.md` line; validate both before offering them.

**Why:** the first manual run directed the user to a command that has not been written yet.
A later run also added a manual verification in its closing summary that the task did not specify.

### P10 Risks and interface boundaries
Rank risks by acceptance criteria and evidence, not incidental implementation preferences. Do not
turn an implementation choice (such as validating before object construction) into a requirement
unless the request or code establishes it. A citation showing execution order is not evidence
that the order is required: for example, rejecting a CLI value need not precede harmless object
construction unless construction has relevant side effects. Tie the risk to the observable
failure and its evidence. When a public interface accepts constrained input, show or state its
invalid-input behavior; CLI validation does not protect direct service callers. Every selected
contract needs verification, including permissive fallback behavior.

**Why:** manual runs invented a parser-order requirement and treated invalid service arguments
as unreachable. A later run defined a fallback alternative but wrongly said it needed no tests.
Removing documentation as a decision also led to excluding it instead of doing it routinely.

### P11 Review the plan as one design
Before showing or approving a plan, compare its decisions, affected areas, stories, context and
out-of-scope claims. They must describe the same observable behavior. For a system with multiple
modes, show how a caller selects each mode and what happens with existing invocations. Do not
defer contradictory behavior to story planning.

Record what the user actually answered separately from derived assumptions. A hosting or stack
choice does not settle network exposure, transport protection or registration policy. Ask another
round when an answer leaves a material product boundary unresolved. A risk paragraph is not a
substitute for that answer, nor permission to exclude required behavior. Recommendations must
address viable alternatives honestly; distinguish a prerequisite from an implementation preference.
For lifecycle choices, compare lifetime and expiry, revocation and the actual user/operator path
for exercising them. A trusted network does not settle token lifetime; a possible database edit
is not an implemented revocation feature.

Explicitly selecting options, including "use all recommended", marks them `decided`. Approval
without choosing leaves recommended options `assumed`. Recheck consistency after choices change,
and revise the version and changed-since line when the plan content changes.

**Why:** the Initiative run contradicted its own CLI mode selection, inferred transport policy
from self-hosting, and incorrectly treated open registration as necessary for member-led teams.
It also overloaded a skeleton story and inferred missing tracker access from missing Git remotes.
**Rejected:** accepting those conflicts because the headings and story structure were correct;
adding tasks to resolve product contradictions later.

### P12 Visible structure and story readiness
Use Markdown tables for comparable material: summary fields, decision options, affected areas,
change maps, story overviews, tasks and checks. Show story and task dependencies in a text tree;
retain explicit dependency ids for multiple prerequisites. Use Mermaid for flows, interactions
or dependency graphs when it conveys relationships beyond the tables; provide a text fallback
when rendering is unavailable. Keep sketches and explanations outside table cells. Tweak stays
brief. Presentation must expose the design and required checks, not replace them with decoration.

A story review shows its summary, acceptance criteria, open decisions, shape, full change map,
and all task rows with files, qualified ids, dependencies, test boundaries, behavioral checks and
commands. A shared suite command alone is not a per-task done-check. The story becomes planned
only on approval; do not hide new decisions elsewhere in the parent plan during review.

Before planning or recommending another story, inspect its dependencies and available evidence.
If prerequisite code is absent, make assumptions about its interfaces explicit. Planning ahead
is allowed, but name what must be revalidated after those prerequisites are implemented and before
building. Do not describe a dependent story as ready to build merely because it has a plan.

**Why:** the first story run asserted unchanged code without content evidence, omitted its change
map and task checks from review, and offered the next dependent story without explaining missing
prerequisite code. The user requested tables, dependency trees and useful diagrams to make the
review easier to inspect.
**Rejected:** dense prose lists for comparable records; charts that add no information; forcing
all stories to be built before any dependent story can be explored.

## Workflows

### Small feature (Change)

```
/specshape-plan PROJ-612        size: change → plan (summary, decisions, change map, 2 tasks) → approved
/specshape-build PROJ-612 T1    preview → go → build → checkpoint
/specshape-build PROJ-612 T2    preview → go → build → checkpoint
/specshape-check PROJ-612       code vs plan → shipped
```

Saved: `.specshape/PROJ-612.md`, then a checkpoint line per task, then the check result.
A Tweak stops at sizing and saves nothing. End its response with an explicit handoff: ask the
agent to make the edit directly, or say "fix it" to continue outside Specshape. The planning run
ends without code changes; a later implementation request is handled as ordinary agent work,
without requiring `/specshape-build`.

**Why:** the manual Tweak run correctly skipped a plan but gave no next step. Small work should
return to direct implementation rather than leave the user at a dead end.

### Big feature (Initiative)

```
/specshape-plan PROJ-500        size: initiative → ask → plan (goal, decisions, affected areas,
                             stories without tasks) → approved → creates PROJ-501…504
/specshape-plan PROJ-501        plan that story: shape, decisions, tasks → approved
/specshape-build PROJ-501 T1    preview → go → build → checkpoint   (repeat per task)
/specshape-check PROJ-501       code vs story → check line, offers to move the ticket
/specshape                      tree of stories, suggests /specshape-plan PROJ-502
```

Saved: `.specshape/PROJ-500.md`, which grows as each story is planned, built and checked.
A Feature that fits one session works like a Change plus an ask round and a design shape,
with no stories or tickets.

### New project

```
/specshape-plan "habit-tracking API"   size: project → ask rounds → plan (goal, decisions, shape,
                                    stories) → approved → creates the epic and stories
/specshape-plan HAB-2                  walking skeleton, grounded in the plan's decisions and shape
/specshape-build HAB-2 T1 … / check    same loop as above
/specshape-plan HAB-3                  from here on, grounded in the real code
```

Saved: `.specshape/habits.md`. If a project decision changes, `/specshape-plan habits` revises the plan
and marks planned-but-unbuilt stories stale.

### What each command saves

| Command | Saves |
|---|---|
| `/specshape-plan <id>` | the plan file; tickets for stories (after confirmation); context changes in `context.md` on approval |
| `/specshape-plan <story>` | that story's shape, decisions and tasks in the plan file |
| `/specshape-build <id> <task…>` | code for those tasks; a checkpoint line per task; new decisions |
| `/specshape-check <id>` | a check result; accepted drift as decisions; ticket status if the user agrees |
| `/specshape` | nothing |

## Rules for this repo

- Change a decision here before changing a skill.
- Examples must not contain proprietary code or internal ticket ids.
