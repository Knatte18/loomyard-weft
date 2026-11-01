# One fixed orch role for every repository, brought up to date

`orch-role-current` · Task · enhancement, orch, stencil · running · Worktree-Create

Depends on: [`heddle-orch-target`](design-heddle-orch-target.md)

## Seen (orch, 2026-11-01)

The orch role, `contracts/stencils/orch/orch-template-role.md`, is lym's instruction to every orch in every repository, yet it has gone stale against the operator's later decisions:
- It sends the arbiter's hand-on question, a review escalation of cause `decision` and a landing-stop halt to the operator, and says the orch never settles them; the operator decided the orch settles run-level calls itself and brings only real design questions.
- Its run loop says to watch a run with `lym batten status`, while the operator ruled out status polling.

Rules meant for every orch sit instead in the loomyard hub's rules file and in the orch note: say worktree, no status polling, no `cd`, a question is never a go-ahead, no priority unless the operator names one, Codex effort at most high, agents read the source themselves, a rename runs alone, the orch note only at a cycle, no changes in the prime, release only on a named go-ahead, loom only for bundles, a board body holds the current design only, explorations run as darn docs, and others.
The hub rules file also holds workaround sections for #799, #849 and #885, all three closed.
Like every stencil, the role has an editable board copy on Bolt, `_lym/stencils/orch/orch-template-role.md`, through which one hub could change it.

## Idea (operator, 2026-11-01)

- The orch role is one fixed file, the same for every orch in every repository, with no board copy a hub can edit.
- The rules for every orch move into it from the loomyard hub's rules file and the orch note.
- It states that the orch settles run-level calls itself, the arbiter's hand-on question, `decision` escalations and landing-stop halts included, and brings only design questions to the operator.
- It drops the `lym batten status` watching that contradicts no status polling.
- It names heddle, through `lym orch rules`, as the home of a repository's own orch rules, and says the orch note holds run state only.
- The closed workaround sections leave the hub rules file.
