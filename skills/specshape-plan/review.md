# Reviewing Initiative and Project plans

Check the design as a whole before showing it, after feedback, and before approval. Resolve
conflicts in the plan rather than deferring them to story planning. Keep implementation detail
for the story; settle product boundaries that affect several stories now.

## Follow one user flow across the plan

Trace an existing invocation and the new flow through the decisions and story acceptance criteria.
For multiple modes, state how each is selected and the default. A plan cannot say commands are
local-only in a decision and remotely routed in a story. The affected areas, context, and out-of-scope
claims must agree with the same behavior. No extra document or task breakdown is needed.

## Separate answers from assumptions

Record only what an answer establishes. For example, self-hosting does not establish who can reach
the service or how credentials are protected in transit. If this changes product scope, ask a
follow-up with a recommendation; do not silently exclude protection and call it a documented risk.

Distinguish independent policies: who creates accounts, who grants membership, and who may access
a team's data. Admin-created accounts can coexist with member-managed membership. Evaluate
alternatives on their actual consequences rather than claiming one is impossible without evidence.
Keep recommendations open until resolved; approval alone marks unchosen recommendations assumed.
An explicit "use all recommended" selects them and marks them decided.

For lifecycle choices, explain expiry and revocation separately, including how a user or operator
actually exercises them. Network trust does not justify an unlimited token lifetime by itself.
Deleting a row is not a delivered revocation feature unless the operator workflow is planned.

## Check story boundaries

Each story has a useful outcome, acceptance criteria and explicit dependencies. A walking skeleton
proves the smallest useful end-to-end path; it need not implement every administrative capability.
If several independently reviewable outcomes are bundled as a "skeleton", split or accurately name
the foundation and justify its scope. A story can depend on previous stories without being one
unfinished layer of work. Keep tasks for later story planning.

When expanding a story, reread its affected code or compare against a verified content baseline;
status and file listings cannot prove unchanged contents. Check whether prerequisite stories
exist in code. Planning ahead is allowed: state the assumed interfaces and revalidation needed
before build, and preserve that qualification in next-step suggestions. Show all task details
and new decisions during story review, following [presentation.md](presentation.md).

## Check offers against available capabilities

Check connected tracker integrations or the source ticket before claiming no tracker access. A Git
remote says where code lives, not whether ticket tools exist. Keep local ids if access is unavailable;
ask for the destination if access exists but routing is unclear. Creating tickets still needs yes.

Use the same build-skill availability finding for both the next command and any offered AGENTS.md
line. Do not offer to insert an unavailable command even when another paragraph admits it is missing.
