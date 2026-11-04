# AGENTS.md — Previz

Read this before doing anything in this repo. Several different coding agents work on this
project, so everything an agent needs to continue lives in files, not in anyone's chat history.

## What Previz is

An open-source, agent-agnostic skill framework: **show the change before making it, and keep
every decision.** It sizes the work, plans it against the real code, previews each task right before
it's built, and checks the code against the plan afterwards.

## Read these first

| File | What |
|------|------|
| `docs/design.md` | **Source of truth.** Every decision (D1–D20, P1–P7) with its reasons and the alternatives rejected, plus the workflows |
| `skills/previz-plan/SKILL.md` | The only skill written so far, and its reference files next to it |

## Layout

```
docs/design.md              design record
skills/<command>/SKILL.md   one skill per command; frontmatter name + description, then instructions
skills/<command>/*.md       reference files the skill loads when needed
```

## Status

| Command | State |
|---------|-------|
| `/previz-plan` | Written. Not yet tried on a real ticket |
| `/previz-build` | Designed (D4, D5, D13, D14), not written |
| `/previz-check` | Designed (D12, D20), not written |
| `/previz` (status) | Designed (D3), not written |

Next, in order:

1. Try `/previz-plan` on a real small ticket and a real big one; fix what falls short.
2. Discuss and then write `/previz-build`.
3. Discuss and then write `/previz-check`, then `/previz`.
4. README, an example plan, install instructions; publish (no remote yet).

Update this table and list when a step is done.

## How to work here

- **Discuss before writing.** When the user is discussing a design, don't create or edit files
  until they explicitly say to (for example "write it" or "go ahead"). Answering a design question
  is not approval to build.
- **Decisions first.** Change or add a decision in `docs/design.md`, with its reason, before
  changing a skill. Skills must never contradict the design record.
- **Skills are agent-agnostic.** Write what to do ("ask the user", "read the ticket", "hand
  exploration to helpers if you can"), never which tool or agent-specific folder to use.
- **Keep skills lean.** Only write what an agent will read again (D8). Put detail in reference
  files next to `SKILL.md`, not in `SKILL.md` itself.
- **No proprietary content.** Examples must not contain internal code, ticket ids or company names.
- **Commits:** imperative subject line, a body saying why. Commit only when the user asks.

## Trying a skill locally

Point your agent at the skill file, or install it where your agent loads skills from. For
example, for Claude Code: `ln -s ~/projects/previz/skills/previz-plan ~/.claude/skills/previz-plan`.
