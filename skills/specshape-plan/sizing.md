# Sizing

Size decides how much process the work gets. Judge the signals against what you found in the code,
not against the ticket's wording. **When the signals point to different sizes, take the biggest.**

| Size | Pick it when |
|------|------|
| **Tweak** | 1–2 files, no new public interface, no behaviour change for callers, nothing to decide |
| **Change** | One module, a handful of files, at most 2 decisions, no schema or external contract change |
| **Feature** | Any Feature signal below, and the work fits one session |
| **Initiative** | Any Initiative signal below |
| **Project** | There is no codebase yet |

## Feature signals

Any one of these makes it at least a Feature:

- crosses module or service boundaries
- adds or changes a public interface, a seam other code depends on, or a config surface
- changes a schema, or needs a migration or backfill
- changes an external contract: an API, event, file format or CLI flag
- changes behaviour for existing users or callers
- has a step that's hard to undo: deleting data, sending to a third party, an irreversible migration
- needs 3 or more decisions from the user

## Initiative signals

- the work won't fit one session: more than about 10 tasks
- parts can ship independently and would be reviewed separately
- a decision waits on a fact nobody has yet (how a vendor API behaves, the shape of production data)

## Reporting the size

Open the reply with it, then carry on:

```
Size: Feature
- crosses 3 modules (downloader, DoFn, pipeline)
- adds a public enum and a validator
- ad-hoc runs with groupBy will now fail fast (behaviour change)
(say "treat as change" to skip the questions)
```
