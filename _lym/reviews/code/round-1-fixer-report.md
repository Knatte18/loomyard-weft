## Fixer-report — code round 1

F1 is `decision`-class and went to the parent, so this report leaves it out.

## Changes

- **F2** (`internal/beamcli/dobbypairanchor_test.go`): added `TestDobbyRuns_ReadsWhetherAPausedRunsWorkReachedItsParent`. Rows: paused at Worktree-Teardown reads reached, paused with a child Finalize done reads reached, paused mid-run reads unreached, running at Worktree-Teardown reads unreached (the paused gate).
  Fixed differently: the proposed unreadable-judgment case is not covered. The hubforge pair's parent holds the task work, so a child Finalize running read `true`, not an error, and no cheap fixture makes the judgment fail. The Warn-and-leave path stays untested.
- **F3** (`internal/dobbyengine/doc.go`): the first "The passes" bullet now says a run gets a pass while running, or paused with its work on its parent.
- **F4** (`internal/battencli/dobbyreads.go`): the Warn goes through a package `logger.EpisodeSet` keyed by slug, as the package's repeated-failure policy does: Warn at the first failure, Debug while it repeats, Info when the judgment reads again or the run leaves paused.
- **F5** (`internal/dobbyengine/passes_test.go`): added a `drivePasses` subtest. A paused reached-parent run is covered, lifted while its driver lock is held, and gets the lift's full pass once the lock is released. It fails under the old `State != running` cleanup, which dropped the pending lift.
- **F6** (`contracts/orchrole/orch-role.md`, `internal/battencli/cli.go`): rule 3 names `lym batten goto` beside `lym batten pause`, and the `goto` help carries the same exception as the `pause` help.
- **F7** (`internal/dobbyengine/passes_test.go`): added `PausedReachedParentOwnedElsewhereProbeGivesFull` and `PausedReachedParentHeldGivesNone` (lift trigger, so it fails without the `Held` check). Removed the `OriginOwed` and `CancelMarked` paused rows, which repeat state-independent running-run rows.
- **F8** (`internal/battencli/cli.go`): the `lym batten pause` help states that a run whose work reached its parent is not held and names `lym tender pause <slug>`.
  Fixed differently: the proposed pointers in `internal/fabriccli/anchor.go:318` and `internal/pairteardown/claim.go:102` were made, then reverted after the self-check. Those messages ask for the batten process to exit or the run to leave running (`anchordeps.go` live check, `claimHeld`), which `lym tender pause` does not do (`tendercli/cli.go:93`: it holds agent and network work and interrupts turns).

Gate: `lym gate test` tier 0 over `dobbyengine`, `battencli`, `fabriccli`, `pairteardown` passes, and tier 1 over `beamcli` passes. Commits are one per finding; F5 and F7 share one commit because both edit `passes_test.go`.

## Disputed

- Premise false, fix-consistency finding on `internal/battencli/wire.go:1486`: `PauseWayForward` is the command a refused goto or run names to stop a running run at a boundary (`shedengine/goto.go:109`), which `lym batten pause` does and `lym tender pause` does not.

## Deferred

None. The F2 unreadable-judgment case is the coverage limit named above.

## Self-check

- fix-code-correctness, LOW: the held paused row passes without the `Held` check. Fix as proposed: the row uses the lift trigger.
- fix-code-correctness, LOW: F2's unreadable case is untested. Fixed differently, named above as not covered.
- fix-code-correctness, LOW: an F4 episode stays open when the run leaves paused. Fix as proposed in the proposal lens's duplicate: `Recover` runs in the not-paused branch.
- fix-consistency, LOW: `pairteardown/doc.go:78` disagrees with `claim.go:102`. Resolved by reverting the `claim.go` clause; the doc matches again.
- fix-consistency, LOW: `wire.go:1486` `PauseWayForward` names `lym batten pause`. Dispute, premise false for the finding's purpose: the text is the way to stop a running run at a boundary for a refused goto or run, which `lym batten pause` does and `lym tender pause` does not (`shedengine/goto.go:109`).
- fix-consistency, LOW: the `goto` help lacks the exception. Fix as proposed.
- fix-proposal, MEDIUM: the tender-pause clause in `anchor.go` and `claim.go` does not do what those messages ask. Fix as proposed: both clauses reverted.
- fix-proposal, LOW: the F4 episode closes only on a paused read. Merged with the correctness duplicate above.
- fix-affected-set, LOW: `pairteardown/doc.go:78` stale against `claim.go`. Resolved with the revert.
