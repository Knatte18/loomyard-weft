# board-lock-running: run-held board entries keep their scope

Batch `01-run-lock` was the only batch in the run's card list.
It was recorded `done` at head `ee1b75a965f1ff34dd0c7c9da521d5d82bbbecb3` with no reported deviations.

The fork committed three cards, because its prompt covered all three plan cards:

- `74f8aa054` (card 1): the run-status form and the scope lock in the board's write critical section.
- `6b7804f39` (card 2): a shared hub-board opener and a run-status write used by loom.
- `ee1b75a96` (card 3): batten's Worktree-Create row claims the board entry before it creates the pair.

`record-batch` returned only informational `scope-outside-plan` warnings.
They name symbols touched outside the batch's own target glyphs, in `internal/boardengine` (`runlock.go`, `runlock_test.go`, `board_test.go`), `internal/battencli`, `internal/boardcli` and `internal/loomcli` tests.

The fork reported that each card's gate was green when committed.
It did not run the plan-level verify tiers; the verify gate runs them after this turn ends.
