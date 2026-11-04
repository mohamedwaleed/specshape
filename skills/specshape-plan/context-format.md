# Context format

`.specshape/context.md` is what every agent reads before working on the code. It holds three things
that are true across the whole project. Create it only when a plan has something to put in it, and
change it only by applying an approved plan's **Context changes**.

```markdown
# Context

> Also see: docs/glossary.md        <- if the repo already has a glossary; add only what's missing

## Glossary
**Account** — the company that pays and owns habits. One per company.
  Not: customer, tenant.                                    (src/accounts/account.ts:8)
**User** — a person who signs in. Belongs to exactly one Account.
  Not: member.                                              (src/users/user.ts:5)

Account 1─* User 1─* Habit 1─* Reminder

## Contracts
C1 Money is integer minor units plus an ISO currency code, everywhere.
C2 API timestamps are UTC ISO-8601.
C3 A deleted Account is soft-deleted; its data is kept for 30 days.   (see openapi.yaml)

## Rules
- No version numbers in class names.
- Move tickets to done after a passing check without asking.
```

## What goes where

- **Glossary:** each term, its definition, the words not to use for it, how terms relate, and
  where it lives in the code once code exists. Use the code's names; when the code and the user's
  words differ, the user decides which is right, and the plan records it.
- **Contracts:** truths that hold project-wide: invariants, formats, lifecycle rules. Link to an
  OpenAPI file, schema or ADR instead of copying it. A plan decision that becomes true project-wide
  moves here.
- **Rules:** the user's lasting preferences about how work is done.

Number contracts C1, C2 … and never reuse a number. To change one, edit it in place, and say in the
plan's Context changes what it was before.
