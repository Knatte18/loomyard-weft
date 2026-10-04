# No recovery path needs a command the agents are denied

`no-denied-recovery` · Note · enhancement, undecided, agents, permissions, webster

Seen 2026-10-03 in the `pair-lifecycle` run: Webster refused a plan fix for a failed batch.
Its only named way forward was "reset the branch to the run's start commit with git and run `lyx webster run --fresh`" (see #350).
The driver, even in bypass mode, could not run `git reset --hard`, because `~/.claude/settings.json` denies `Bash(git reset --hard:*)` and a deny rule holds in every mode.
The run stopped until the operator ran the reset by hand.
`contracts/specs/refusal-spec.md` prescribes `git reset --hard <pre-fix head>` for another halt as well.

Idea (operator): a permission refusal must never stop the flow.
Every recovery a lyx refusal names must be doable by the agent that receives it.
Either lyx performs the destructive step itself, in Go, guarded to the run's own branch and start commit (`webster run --fresh` resetting the branch, a `keep|drop` verb for the integration fix commits), or the agents' permission set is loosened for what lyx's recoveries need.
Audit every refusal and stencil for commands the deny list blocks.
Related: `orch-live-hardening` (permission modes mean what they say).
