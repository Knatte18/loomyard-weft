# Webster stencils describe a stale fork spawn and wake-up

`webster-fork-spawn-wording` · Note · bug, stencil, webster

## Seen

Found by the stencil intent audit (`docs/research/stencil-intent-audit-2026-10-15/findings.md`, F5); line numbers as of `main` on 2026-10-16.

- `contracts/stencils/webster/webster-prefix-fork.md:24` tells the fork that "Master's own `await-batch` sees it and takes over" once the report is written.
  Master no longer runs `await-batch`: it ends its turn after the spawn and is woken by the fork's completion notification (`internal/websterengine/doc.go`).
  The banner at `:3` likewise names an Agent-tool call, "Read this file and follow it exactly: <path>", that Master never sends.
- `contracts/stencils/webster/webster-template-master.md:12` tells a fork to recognise itself by that same instruction, but the fork prompt Master sends (`:73`, and the verify-fix fork at `:168`) reads "Read this file and do exactly and only what it says".

## Decision

Operator decision (2026-10-16): the stencils' account of how a fork is spawned and how Master learns it finished is wrong and is to be fixed; the operator leaves the shape of the fix to the agent.

Operator view, not a decision: Master's prompt cache is believed to stay warm while a fork runs.

## Idea

Agent's suggestion: in the fork prefix, say the fork's turn ends after the report and Master is woken by its completion notification, and drop the `await-batch` wording except the ban on running it; make the master's self-identification rule at `:12` quote the instruction Master sends, or key it on "You are an implementer fork", which both spawn prompts open with.
The cache remark needs no change here: the fix touches only wording.
