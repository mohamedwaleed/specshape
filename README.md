# Specshape

### Code review, before the code exists.

![status: alpha](https://img.shields.io/badge/status-alpha-orange)
![agents: any](https://img.shields.io/badge/agents-any-blue)
![license: MIT](https://img.shields.io/badge/license-MIT-green)

Your coding agent is fast. It's also happy to build the wrong design, and you find out when you
read a 900-line diff.

**Specshape shows you the design of a change while it's still a sketch:** the real class names,
where each decision lives in the code, what gets added, changed or removed. You make the design
calls. The agent writes the code. Your architecture stays yours.

```bash
npx skills add <owner>/specshape        # then, in your agent:
/specshape-plan PROJ-123
```

One command on a real ticket, in the codebase you already have. No setup, no new process to adopt.

> **Status: alpha.** `/specshape-plan` is ready to try. `/specshape-build`, `/specshape-check` and
> `/specshape` are next; see [Roadmap](#roadmap).

---

## The moment Specshape was built for

An engineer asked an agent to migrate a data pipeline to a new API. The agent wrote a plan first,
and the engineer approved it. One line in the plan said:

> *"In `processElement` it picks a downloader based on `apiVersion`."*

Fine, on paper. Then the code arrived, and with it an `if` that checked the API version for every
element, and a `V1` prefix on every new class. The engineer's reaction was immediate: *"this
feels hacky; use a factory. And no version numbers in class names, they'll be wrong the day the
API moves to v2."*

Both were good calls. Both came **after** the code existed, so fixing them meant restructuring a
class and renaming every versioned class and package.

The plan wasn't wrong. It just described the design in a sentence instead of showing it. **Specshape
shows it**, so calls like these take one line of feedback at plan time instead of a rework after.

---

## Three pillars

### 1. Right-sized

Specshape sizes the work from the code, not from the ticket's wording, and gives it exactly as much
process as it needs:

| Size | You get |
|---|---|
| **Tweak** | One line: what to change and where. No plan. |
| **Change** | A short plan with 1–4 tasks. No questions. |
| **Feature** | One round of questions, then the design sketched against your code |
| **Initiative** | Questions, then stories; each becomes a ticket and is planned when you reach it |
| **Project** | A glossary first, then the system's shape, starting with a walking skeleton |

A typo never gets a ceremony. A "simple" ticket that crosses three services never sneaks through
without one.

### 2. Design before code

Every plan shows the design in a form you can judge in minutes:

- **Real names.** The classes, functions and files that will exist, not "a new component".
- **Where each decision lives.** Dispatch, branching and seams appear as sketches, not sentences.
- **Grounded in your code.** Every claim about existing code cites `path:line`; anything unverified
  is marked as an assumption.
- **Decisions up front.** Each one has the options, a recommendation, and what changes for you
  either way.

You steer it while it costs a sentence. Then each task is previewed again right before it's built,
against the code as it is at that moment. That matters most in large systems, where many people
and agents change the same code, and the architecture erodes one reasonable-looking MR at a time.

### 3. Knowledge that compounds

Say *"never put version numbers in class names"* once. Specshape saves it to `.specshape/context.md`,
alongside your **glossary** (what an "Account" is, and which word not to use for it) and your
**contracts** ("money is integer minor units, everywhere").

Every later plan, by every engineer, in every agent, follows them. The tenth plan needs fewer
corrections than the first, and a new teammate's plan follows the same rules as yours.

---

## What a plan looks like

```
you> /specshape-plan PAY-88 "support a second payment provider"

Size: Feature
- crosses 2 modules (checkout, payments)
- adds a public interface other code depends on
- 3 decisions needed

## Summary
- What changes: checkout can charge through Adyen as well as Stripe, chosen per merchant
- Footprint: 4 new, 2 changed files; 5 tasks
- Top risk: refunds call Stripe directly in two places (src/refunds/refund.ts:31, :77)

### D1 Where is the provider chosen? (open)
Today `PaymentService.charge()` calls Stripe directly (src/payments/service.ts:42).
- a) an `if (merchant.provider === ...)` inside charge() and refund()
- b) a `PaymentGateway` interface, picked once per merchant by `GatewayFactory`
➡️ Recommended: b. With a), every new method repeats the check, and a third provider means
   editing every one of them.

## Shape
interface PaymentGateway {
  charge(order: Order): Promise<Charge>
  refund(charge: Charge, amount: Money): Promise<Refund>
}
class GatewayFactory {
  forMerchant(merchant: Merchant): PaymentGateway   // the only place providers are named
}
// PaymentService keeps its API; it asks the factory instead of calling Stripe

## Tasks
⬜ T1 PaymentGateway + StripeGateway (behaviour unchanged)   done when: payments tests green
⬜ T2 GatewayFactory                          needs T1        done when: factory.test.ts green
⬜ T3 AdyenGateway                            needs T1        done when: adyen sandbox test green
...

Answer D1–D3, give feedback, or say approved.
```

You see the shape of the change, the decision that matters, and the evidence in your own code,
before a single file is touched. The plan is saved to `.specshape/PAY-88.md` and committed with the
code, so reviewers see what was agreed.

---

## Why not just use plan mode?

Most agents now have a built-in plan mode. Use it for quick tasks. Specshape is for work where the
design matters:

| | Built-in plan mode | Specshape |
|---|---|---|
| Shows the design as | prose and a step list | real names and sketches of where each decision lives |
| Grounded in your code | sometimes | every claim cites `path:line` |
| After the session | gone | a file, committed and reviewed with the code |
| Remembers your rules | no | `context.md` applies them to every plan |
| Adapts to the size of the work | no | Tweak to Project |
| Checks the code against the plan | no | yes, with `/specshape-check` (planned) |
| Works across agents | one agent each | any agent; the same plan moves between them |

**Compared with spec frameworks** such as spec-kit: they generate a set of documents about *what*
to build, with the same steps for every size of work. Specshape produces one plan, shows *how* it will
be built in your code, and scales from nothing to stories.

**Compared with interview skills** such as grill-me: those sharpen an idea by asking questions, and
the answers stay in the conversation. Specshape asks only what your code can't answer, then shows you
a design to react to and saves every decision. (grill-me's question rounds inspired Specshape's ask
step.)

---

## How it works

```
/specshape-plan <ticket | spec | "plain description">
  1. Size     Tweak? → one-line answer, done
  2. Ask      only the questions your code can't answer
  3. Plan     shape, decisions, change map, tasks (or stories) → .specshape/<id>.md
  4. Review   your feedback → v2, v3 … → approved

/specshape-build <id> <task>   preview the task against the current code → build it      (planned)
/specshape-check <id>          does the code follow the approved design?                 (planned)
/specshape                     progress, and the next command to run                     (planned)
```

- **The plan is read-only.** `/specshape-plan` never touches your source code; it writes only under
  `.specshape/`. Code is written by `/specshape-build`, for the tasks you name.
- **Two files, nothing else.** `.specshape/context.md` for the shared glossary, contracts and rules,
  and one `.specshape/<id>.md` per piece of work. No spec, research or task documents to wade through.
- **Stories become tickets.** For an epic, Specshape offers to create a ticket per story in your
  tracker, using the access your agent already has.

---

## Install

> Not published yet. Until it is, use the manual install below.

**Skills CLI**, for any agent it supports:

```bash
npx skills add <owner>/specshape
```

**Claude Code plugin:**

```
/plugin marketplace add <owner>/specshape
/plugin install specshape
```

**Manual**, for any agent: clone it and link the skills into your agent's skills folder. With a
symlink, `git pull` updates it.

```bash
git clone https://github.com/<owner>/specshape.git ~/specshape
ln -s ~/specshape/skills/specshape-plan ~/.claude/skills/specshape-plan   # use your agent's skills folder
```

**Agents without skill support:** add a line to your `AGENTS.md`: *"To plan a change, follow
`~/specshape/skills/specshape-plan/SKILL.md`."*

**Teams:** install per project and commit it, so everyone runs the same Specshape version as the plans
in the repo.

### Try it in two minutes

Pick a ticket you'd hand to your agent this week, and run:

```
/specshape-plan <ticket>
```

Read the plan, then disagree with something. Rename a class, reject a pattern, state a rule. That
loop is the whole point.

---

## Roadmap

- [x] `/specshape-plan`: size, ask, plan, review
- [ ] `/specshape-build`: preview and build one task at a time
- [ ] `/specshape-check`: does the code follow the approved design?
- [ ] `/specshape`: progress and the next command
- [ ] Evaluation suite, run across several agents
- [ ] Published to the Skills CLI and the Claude Code plugin marketplace

## Contributing

Contributions are welcome, especially **a real (or anonymized) plan that went wrong**: a vague
sketch, a missed design problem, a question it shouldn't have asked. That's the most useful bug
report there is.

Before changing a skill, read [`docs/design.md`](docs/design.md). It records every decision with
its reasons and the alternatives that were rejected; change the decision there first, then the skill.
[`AGENTS.md`](AGENTS.md) has the working rules, for humans and agents alike.

## License

[MIT](LICENSE)
