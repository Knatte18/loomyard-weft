# Repo-specific orch rules in one place, rendered into the orch role

`orch-hub-rules` · Note · bug, stencil, orch, config

## Seen

Found by the stencil intent audit (`docs/research/stencil-intent-audit-2026-10-15/findings.md`, F14); line numbers as of `main` on 2026-10-16.

`contracts/stencils/orch/orch-template-role.md:131` and `:135` are hub-generic but name this repository's `update-plugins.sh`, and tell the orch to run it by hand after a landing while the hub's landing `deploy` key is empty.
This repository's `CLAUDE.md` lets the orch run that script only after its own pushed fix; the lines are dormant in this hub, whose `landing.yaml` sets `deploy`.

## Decision

Operator decision (2026-10-16):

- The orch stencil carries nothing specific to one repository.
- What an orch must remember for one repository is specific to that repository: it lives in one fixed place and reaches the orch at every context cycle, on top of the orch's general role.
- `CLAUDE.md` holds only what every agent in the repository must have; instructions about the orch's behaviour go to the orch only, since giving them to every agent through `CLAUDE.md` wastes their context.

Operator view, not a decision: the orch watcher injects those rules at each cycle.

## Idea

Agent's suggestions:

- Drop `:131`'s tail and `:135` from the orch stencil.
- Operator's idea (2026-10-16): the file is an `ORCHESTRATOR.md` (or `ORCH.md`) at the repository root, plainly a system file of that repository, beside `CLAUDE.md` and `PATTERN.md`.
  Agent's suggestion on top: track it in git like those two, and have `RenderRoleFile` render the prime's copy into the role file under its own heading; the role file is re-read in full at every start, resume, reload and adopt, so no extra typing by the watcher is needed and nothing is lost to a compaction.
  Claude Code does not load the file on its own, so no other agent reads it.
- Move `CLAUDE.md`'s orch-only sentences into that file, starting with the orch's `update-plugins.sh` exception under `## Production`.
