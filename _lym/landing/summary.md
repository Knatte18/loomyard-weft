# One fixed orch role for every repository, brought up to date

The orch role is lym's instruction to every orch in every repository.
Until now it was a registered stencil with an editable board copy on Bolt, and it had gone stale against the operator's later decisions.
It sent the arbiter's hand-on question, `decision` escalations and landing-stop halts to the operator, and it told the orch to watch runs with `lym batten status`, which the operator had ruled out.
Rules meant for every orch sat in the loomyard hub's rules file and in the orch note instead.

## What changes

- The role moves from `contracts/stencils/orch/orch-template-role.md` to a new leaf package, `contracts/orchrole` (`orch-role.md`, embedded as `orchrole.Role`).
  It leaves the stencil registry, so no Bolt copy is seeded and `lym stencil` no longer lists it.
- `orchengine.RenderRoleFile(path, index)` renders those embedded bytes and no longer takes a stencils directory.
  The runtime check for the role's two markers is removed, since no hub can edit the role and `TestStencils_TopLevelMarkers` pins the markers on the shipped text.
  `prompt.go` splits reading a stencil from filling a template (`writeFilled`, `fill`), so the role and the stencils share one fill-and-write path.
- The role text:
  - The orch settles every run-level call itself and records it with `lym loom decision add --by parent`: an arbiter's hand-on question, a `decision` escalation, a review finding, a hard-cap accept, a landing stop or a false premise.
    Only a design question goes to the operator: what to build, a task's scope or a choice between designs.
    The escalation, peer-run, fault and landing-halt passages and Acting without asking now say so.
  - `lym batten status` is no longer how the orch watches runs: runs are watched through their notices, and a status read on a stop is filtered to the fields needed.
  - A new Working rules section, plus additions to the run loop, escalations, After landing and The board, carries the rules for every orch that sat in the hub rules file and the orch note.
    These cover: no `cd`, no changes in the prime, saying "worktree", a question is never a go-ahead, no priority unless the operator names one, agents reading the source themselves, Codex effort at most `high`, an IDE only on request, only your own hub, the converged-Bouncer stop, preferring decisions plus accept over a new review round, the board tidy, a board body holding the current design only, loom only for bundles, a rename running alone, explorations as darn docs, and Discussion rounds not cut for a full body.
  - Where a finding goes: a bug in the hub's own repository becomes a board note, lym's included when the hub works on LoomYard; selfreport is for lym bugs found in other hubs.
  - The orch note holds run state only and is written only at a context cycle.
    The Orch rules section names heddle, through `lym orch rules`, as the home of a repository's own orch rules.
- The stencil scans in `contracts/stencils` (skill-load, citation, edit-rule and ask-operator scans, and the Go-value table) read the role through a test-only `fixedStencils` table beside the registry, so they still cover it.
- PATTERN: `PATTERN-stencil-ownership` and its background name the orch role as the one exception to "never embedded".
- Docs: the `orchengine` and `orchcli` package docs describe the fixed role.
  Their notes on runtime marker checks and on code fixes pushed from the prime are removed.

## Design decisions

- The role lives in its own package, not in `orchengine` or `contracts/stencils`.
  `orchengine` importing `contracts/stencils` would cycle through `contracts/stencils`' tests, which import `gatecli`, which depends on `orchengine`.
  A leaf `contracts/orchrole` lets both the engine and the stencil tests import it.
- Choosing to take the role out of the registry, rather than overwrite an edited board copy on every reconcile, is the parent decision recorded in the decision record ("The orch role leaves the stencil registry").
- A stale `_lym/stencils/orch/orch-template-role.md` left in a Bolt checkout is no longer read and is left in place: no tidy step for stale stencil copies exists.
- The hub rules file `_lym/orch/rules.md` on Bolt has not been read since `heddle-orch-target`, so its closed #799, #849 and #885 sections leave with it.
  The loomyard-only rules (the `update-plugins.sh` production route and pushing `main` by hand) stay out of the role, and the parent carries them into heddle with `lym orch rules`.
- The old rule "never choose `lym loom circling continue` without asking the operator" becomes the orch's own call, with the cheaper path preferred: record the blocking fixes as decisions and accept, and grant a new round only for findings no decision settles.
- A code fix no longer goes to a fresh agent in the prime: it runs in a task worktree as a board note the operator releases, per the no-changes-in-the-prime rule.

## To check by hand

- Start an orch in a hub whose Bolt still holds `_lym/stencils/orch/orch-template-role.md`, and check that `.lym/orch/role.md` renders the new text, not the board copy.
- Read the rendered role once in full: it is the orch's whole procedure, and the run-level/design split is a wording change no test can judge.
