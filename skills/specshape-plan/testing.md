# Planning tests and implementation

Plan red–green–refactor within each behavior-changing task. Do not write or run new tests during
planning; those are source changes. Describe outcomes and boundaries, not full test bodies.

## What the plan records

- **Public boundary:** the CLI, API, or public method whose behavior is observed. Prefer existing
  test boundaries and cite the tests being extended. Test behavior rather than private structure.
- **Outcomes:** concrete examples derived from acceptance criteria, including relevant failures
  and compatibility guarantees. Specify invalid-input behavior at every public boundary affected;
  upstream validation does not protect independent callers.
- **Files and command:** tests and implementation belong in the same task. Name the command and
  what it must prove. Plan approval confirms these boundaries and outcomes for later build.

## How build will execute each task

1. **Red:** write one behavioral test and run it. Confirm it fails because the requested behavior
   is absent, not because of an unrelated setup error. If it passes already, investigate what
   behavior is missing before claiming a red–green cycle.
2. **Green:** make the smallest implementation change to pass that test; rerun it.
3. **Refactor:** improve the code and tests within the approved scope while keeping tests green.
   Repeat for the next behavior. Do not write all tests first or defer all tests until after code.
4. **Done:** run the task's checks and relevant regression suite. Report actual red/green results
   and final checks at the build checkpoint. Temporary red is expected inside a task; it ends green.

For documentation-only work, use an appropriate review or command check. Where automated testing
is infeasible, explain the constraint and propose concrete verification for approval.

## Example task

**T1 Add status filtering through the CLI**
- Files: service, CLI, and their relevant tests.
- Boundary: CLI subprocess; public service method if independently supported.
- Behaviors: default/all preserve output; open/done select correctly and preserve order; empty
  matches succeed; invalid CLI values exit 2; invalid service values raise ValueError if that is
  the chosen service contract; listing neither creates nor modifies storage.
- Method: red → green → refactor for each behavior, with tests and code in this task.
- Done: the named focused tests prove these outcomes and the regression suite passes.

Documentation can follow as T2. A separate "add all tests" task cannot follow implementation.
