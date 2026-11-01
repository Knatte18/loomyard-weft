```yaml
lens: fix-claims
outcome: reported
```

Lens fix-claims: I checked every claim the fix added or rewrote against the code at HEAD.

**MEDIUM — the way forward for a failed removal can be refused by the board.** This concerns D5's sentence "a failed removal carries the failed step's error and names `lym board remove <slug>`", D6's "its notice carries the failed step's error and names `lym board remove <slug>`, then a re-run of `lym adopt <subject>`", and the D6 acceptance criterion "files none and its notice names `lym board remove <slug>`".
- The claim is that `lym board remove <slug>` is the way forward after any failed removal step. When the failed step is the cancel itself, the fix task is running or halted and its entry still holds a run status. The board's run lock refuses removing such an entry, so the named step cannot be carried out from that state.
- Evidence: `checkRunLocks` in `internal/boardengine/runlock.go:138-160` refuses with `RunLockedError` (or `BeamRunLockedError`) when a run-status entry of `loaded` "is absent from after". `Board.RemoveTask` (`internal/boardengine/board.go:209`) runs inside `boardCriticalSection`, which this check guards.
- Only the nothing-left case F3 named leaves a closed, draft entry, which the board removes. The old text named `lym batten cancel <slug> --remove` first, and the change dropped it for every failed step, not just that case.

**LOW — "counts as the first step done" does not cover every refusal on the never-started path.** This concerns D8's removal bullet: "cancel's nothing-left refusal (`refuseNothingLeft` …), which a released task that never started or a draft one a hand cancel left meets, counts as the first step done".
- For a task that never started, `--remove` with no task branch locally, while origin is unreachable, returns a different refusal from `refuseNothingLeft`: "origin was unreachable, so the task branch … could not be read".
- So such a task does not always meet the nothing-left refusal, and the removal then fails at step 1.
- Evidence: `internal/battencli/cancel.go:582-584`, the `endOnly` branch.

The other claims hold:
- Cancel lifts every live entry of the slug's own scope with no caller filter, the hold included (`slugPauseRefs` and `liftSlugPauses`, `internal/battencli/cancelcmd.go:454-485`).
- A create followed by a null-status release write releases the task: `applyDraftRule` skips a task that was loaded as a task and whose status is now nil (`internal/boardengine/draft.go`).
- `doneSlugRefusal` is set only for a released task whose run status reads `done` (`internal/battencli/dobbyreads.go:292`).
- `Board.EffectiveOwner` and `Board.PrimeOwner` exist (`internal/boardengine/board.go:308,317`).
- `internal/state` locks its writes (`state.WriteJSON` and `state.UpdateJSON` take a lock path).
- The run-record states include `blocked` and `failed` (`internal/shedengine/status.go`).

Tool calls:
- `cancelcmd.go:464-486`, `internal/state/state.go` signatures and the board remove and rename locations: to confirm the cancel lift, the holds-file lock and the remove path, none of which my context showed.
- `slugPauseRefs` and `Store.RemoveTask`: to confirm cancel's lift has no caller filter and to see whether removal has a refusal.
- `Board.RemoveTask`: to see the board's removal path.
- A grep for the run-lock check in `runlock.go`: to find whether the run lock covers a removal.
- `runlock.go:125-175`: to read `checkRunLocks`' rule for an absent entry.
