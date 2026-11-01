# One fixed orch role for every repository, brought up to date

The orch role is now one text shipped in the binary, the same for every orch in every repository, with no editable board copy.
It also states the operator's later decisions, which the old role contradicted.

Closes the board task `orch-role-current`; it resolves no GitHub issue.

## What changes

- The role moves from the stencil `contracts/stencils/orch/orch-template-role.md` to `contracts/orchrole/orch-role.md`, embedded by the new package `contracts/orchrole`.
  `orch-template-role` leaves the stencil registry, so no copy is seeded to Bolt and no hub can edit the role.
- `RenderRoleFile` in `internal/orchengine/prompt.go` no longer takes a stencils directory; it renders the embedded bytes.
  The render helpers split into a read step and a fill step so the role and the remaining stencils share the fill.
  The check that an edited copy still declares its markers is gone, since the text is fixed.
- The role now says the orch settles run-level calls itself and records them with `lym loom decision add --by parent`.
  That covers the arbiter's hand-on question, a `decision` escalation, a review finding, a hard-cap accept, a landing stop and a false premise in an entry.
  Only design questions go to the operator: what to build, a change of scope, or a choice between designs the entry leaves open.
- The run loop no longer has the orch watch runs with `lym batten status`; it works from notices and mail, and reads a status only filtered, when a stop needs it.
  `orch-template-usage-limit-wake` and `orch-template-start` are reworded to match.
- Rules meant for every orch move into the role from the loomyard hub's rules file and the orch note: say "worktree", never `cd`, a question is never a go-ahead, no priority unless the operator names one, agents read the source themselves, Codex effort at most `high`, a rename runs alone, no prime edits, the board-body and exploration rules, and the rest.
- The role names heddle, through `lym orch rules`, as the home of a repository's own orch rules, and says the orch note holds run state only.
- `PATTERN-stencil-ownership` gains one exception for the orch role, in `PATTERN.md` and `pattern/PATTERN-stencil-ownership.md`.
- Tests follow: `contracts/stencils` registry tests pin that the orch role stays unregistered, and the `internal/orchengine` prompt tests render the embedded role.

## Design decisions

- **Out of the registry.**
  The alternative was to keep a seeded copy and overwrite edited ones, which would break the registry's promise that edits survive.
  The role is the one named exception to stencil ownership.
- **Stale Bolt copies.**
  A `_lym/stencils/orch/orch-template-role.md` already in a Bolt checkout is no longer read.
  No tidy step for stale stencil copies exists, so the change does not remove it.
- **Hub rules file.**
  The closed #799, #849 and #885 workaround sections are out of the orch rules, which the parent carried into heddle with `lym orch rules`.
  That lives in the Bolt checkout and heddle, so this diff holds no change for it.
- **Heddle rules add, never override.**
  The role says a repository's orch rules add to it, and that one contradicting the role is raised to the operator while the role is followed.

## To check by hand

- The closed-workaround removal is not in this diff: confirm the hub's current orch rules (`lym heddle get orch-rules`) hold no #799, #849 or #885 section.
- `contracts/stencils/orch/orch-template-usage-limit-wake.md` and `contracts/orchrole/orch-role.md` once contradicted each other on polling `lym batten status`; read the wake line and the run loop's watching item together and confirm they agree.
- Run `lym orch start` once and read the rendered role file, to see the `command_index` and `decision_note_title` markers filled.
