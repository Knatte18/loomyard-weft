# Parent-Align halts when its own fixer pushes the task branch, reading the fixer commits as a foreign move

`parent-align-fixer-own-push` · Note · bug, landing

## Seen

`kde-window-placement-build` halted twice at Parent-Align on 2026-11-28 (14:18 and 14:47 UTC) with
`landing stop: the remote ref origin/<slug> moved during the fixer session of Parent-Align to <sha>, which holds commits the task head <sha> the session started from lacks`.
Both times the new commits were the Parent-Align fixer's own fixes on the task branch (a guard rename, a hermetic-guard exemption, a doc comment), pushed by its session.
The orch checked the commits and resumed with `lym loom resume`; each halt cost a halt notice, an orch turn and the wait.

## Idea

Parent-Align tells its own fixer's commits from a foreign move: commits the fixer session made on the task branch, descending from the session's start head, pass without a halt.
Only a move that holds commits the session did not make stops the run.
