```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness of the pre-push check (`prepush.go` and its wiring). Cases run:

| Function or branch | Case run | Result |
|---|---|---|
| `guard`, skip | `skip()` errors → push unchecked, Warn logged | Correct. |
| `guard`, skip | `SkipGit`/`SkipPush`, prime, pause-covered, status at Finalize | Correct. A push at Finalize (`finalize_pr.go:215`) is skipped, so it is not judged twice. |
| `guard`, reconcile | `reconcile` returns an error → push unchecked | Correct. A reconcile that moved branches and then returned an error posts no notice. |
| `guard`, reconcile | `Unreachable` → push unchecked | Correct. |
| `guard`, reconcile | Pair worktree absent (`originreconcile.go:646` returns an error) → push unchecked | Correct. |
| `stopReason`, `EntryOwnedElsewhere` | Empty `AdoptSubject` falls back to the slug | Correct. |
| `stopReason`, `EntryEnded` | `landed()` errors → read as not landed, so the run stops | Matches the doc comment, but a transient read error pauses a healthy run. Reported as LOW below. |
| `stopReason`, `EntryEnded` | `landed` true → returns `"", false` | Correct. `guard` falls through to the moved-branches notice. |
| `stopReason`, `EntryStopped` | A diverged own branch | Correct. The branch is named with its state. |
| `stopReason`, `EntryStopped` | No own stopped branch (a shared branch stops the entry) | Correct. The "a branch every run shares" fallback is used. |
| `stopReason`, `EntryStopped` | An own branch in `needs_replay` | Incorrect, see finding. |
| `guard`, pause | `pause()` fails → notice carries the failure, push still stopped | Correct. |
| `loomCommitStatusDeps` `Spawn`, stop | Guard stops → logs and returns nil, so no push is spawned and the commits stay local | Correct. |
| `commitrecordsVerb` | `Push` returns `errPushStopped` → ok envelope with `push_stopped` | Correct, given `Push` returns the guard error unwrapped, as `wiring.go:224-230` does. |
| `PushTask` (PR-Review) | Stop error returned as the push's error | Correct. |
| Run-scope reconcile of the run's own pair | Own pair excluded from the `BranchLive` check (`originreconcilemoves.go:379`) | Correct. |
| Run-scope reconcile of the run's own pair | Dirty only when a dirty tracked path overlaps what origin touched (`ffWorktree`, line 147) | Correct. An agent working in the code worktree does not stop every push. |
| `PausePair` | A pause with a plain pair entry; an existing entry adds none | Correct. |
| `PauseOwnedElsewhere`, `pauseGrace` rename | Only the constant name changed; no remaining reference to `ownerPauseGrace` | Correct. |

## Findings

1. **LOW · design · `internal/loomcli/prepush.go:253-257` (`stopReason`):** the stopped-branch filter allows only level, ahead and fast_forward, so `BranchNeedsReplay` is also named as a stopped branch. `BranchState.stops()` (`originreconcile.go:156-162`) says that state does not stop an entry.
   - Case: the entry stops because the shared Bolt or the prime branch stops, and the run's own records branch is `needs_replay`.
   - Result: the notice reads "origin moved task (needs_replay)", blaming a branch that did not stop the run.
   - Evidence: `prepush.go:254`, `originreconcile.go:141,158`.
   - `stops()` is unexported, so it cannot be called from `loomcli`; the fix is to name only the states in `stops()`'s list.

2. **LOW · design · `internal/loomcli/prepush.go:243-250` (`stopReason`):** a failed `landed()` read counts as "not landed", so an ended entry pauses the run. The doc comment states this on purpose, but the pause is a false stop for a run whose parent holds its work. Same case: origin ended the run after its own landing and `ParentHoldsTaskWork` returns a transient error.
   - Evidence: `prepush.go:243-250`.

No MEDIUM or higher correctness defect found.

Tool calls (3 greps/reads, 2 reads):
1. grep `originreconcile.go` for live/dirty/scope handling, to see how the run scope treats the run's own pair.
2. grep `internal/fabricengine` for where `BranchLive`/`BranchDirty` are assigned.
3. grep + sed of `originreconcile.go` scope doc and mover logic (the grep matched little; I read the scope doc lines).
4. Read `originreconcilemoves.go:125-165` for `ffWorktree`'s dirty rule, which my context lacked.
5. Read `originreconcilemoves.go:360-410` for the live exclusion of the run's own slug, which my context lacked.
