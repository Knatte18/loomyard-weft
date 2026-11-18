# landingshed: one ending type, merged helpers, fewer seams

This is the pilot for `design-simplify-callers`.
It makes the design simplifications the shrink study (`docs/research/rewrite/shrink-estimate-2026-11-18/report.md`, step 3) found in `internal/landingshed`,
and measures the lines they save against the study's estimate.
Behaviour is meant to stay as it was: every verdict, Stuck reason and way forward is unchanged, except the one error text named under "Behaviour notes".

## Lines saved, per spot

Production code lines of `internal/landingshed`, counted with `go run ./tools/codestats -root internal/landingshed` (code lines only, comments and blanks apart), after each commit:

| Spot | Commit | Code lines before | Code lines after | Saved |
|---|---|---|---|---|
| One result type in place of the four-value return | `landingshed: one ending type in place of the four-value step return` | 5,418 | 5,312 | 106 |
| Near-identical helpers merged | `landingshed: merge the near-identical helpers of the landing rows` | 5,312 | 5,213 | 99 |
| Single-caller or test-only seams and adapters removed | `landingshed: drop single-caller seams and adapters` | 5,213 | 5,131 | 82 |
| Lock-freed, requeue and rebase layer around the fixer | none: kept, see below | 5,131 | 5,131 | 0 |
| Total | | 5,418 | 5,131 | 287 |

The study estimated about 600 to 800 lines saved by these spots: about 150 for the result type, about 340 for duplicated logic, 100 to 190 for seams and adapters, and about 90 for the lock layer.
The measured saving is 287 lines, 36 to 48 per cent of that estimate.
Production comment lines went from 2,568 to 2,570, since comments were out of scope.

Test code changed by one line: `prreview_wait_test.go` reads the `ending` that `PRReview.wait` now returns.
Test code lines stay 11,003.
Outside the package, `internal/shuttleparity/seats.go` names the new holder of the seat-table literal in three catalog entries.

## What changed

### One result type

Every landing step returned `(Outcome, OutputPointer, error, bool)`, or `(Outcome, OutputPointer, bool, error)` in PR-Review, and `pollStatus` returned six values.
Each step now returns `*ending`, and nil means the call goes on.
`fail`, `stuck`, `halt` and `gateEnding` build it, and `ending.result` gives back the three values a `ShedProducer` returns.
That removes the per-producer wrappers (`errorStop`, `halt`, `stuckOrCancelled`, two `gateStop`s, `alignError`, `haltStop`, `ownerHalt`, `landHalt`, `prStopped`) and the local `end`, `requeue`, `ended`, `failed`, `done` and `stuck` closures.
`landUnderLock` returns nil for a pass that freed the lock for a fixer, in place of a fourth `again` value.

### Helpers merged

- Finalize runs merge, gate and parent-side merge in one loop that retries once after a merge-in-required error, in place of a second copy of those steps.
- Finalize reads whether the parent already holds the task's work once per pass, for the owner checks, the land check and the sync, none of which moves the parent.
  The read after the merge, for the overlap, stays its own.
- Finalize and PR-Review share one GitHub-failure judgement, the task push, the pull request status read, the close of an open pull request, find-or-open, the record commit and removal, and the read of the description, regression check and credit block.
- Finalize and Parent-Align share the resolver construction, the landing gate with its failure recorder, the read before the merge (`readMergeBefore`) and the attempt-record read (`readRowAttempts`).
- The fixer and fix-review report parsers share one verdict-and-sections reader.
- The fixer, fix-review and Describe tables share one one-seat builder (`soloSession.table`), and PR-Review's session spec shares its model-spec resolution.
- The fixer and fix-review runs share one not-started check.
- The landing gate and the pr-review gate share one command read and one marked verify.

### Seams and adapters removed

- The `parentPuller` interface, which nothing used.
- PR-Review's 52-line wiring check: it now reads every `PullRequestDeps` field by reflection and names the first unset one the row needs.
  The `PullRequestDeps` doc now names all four optional fields; it had named two.
- Finalize's `handedOver` struct field, which carried a hand-over's error from one method to another; the hand-over returns its halt directly.
- The one-field `parentMerge` type and its `mergeIntoTask` adapter, the `noteFlakyPass` method and the `creditBlock` read, each inlined at its one caller.

## The lock-freed, requeue and rebase layer stays

The board entry asked to design this layer away by running the fixer outside the locked pass, if the spec's invariants allow it.
It already runs outside the lock, and the spec's rules keep each of its three parts:

- **Freeing the lock** is the rule "Free the parent lock before a fixer or a fix review starts" (`1fbac34f`, one run holding `main` for twelve minutes).
- **The requeue**, retaking the lock at the front, merging anew and re-verifying, follows from the same rule.
  The tree the fixer fixed is not the tree that lands while the parent can move, so the pass has to run again under the lock.
  `returningFromFixer` makes a resumed run queue at the front for the same reason.
- **The Baseline move** (`rebaseAfterParentMerge`) follows from the rule that the fixer's whole change is judged by `LineChange(Baseline, HEAD)` (`f269badc`).
  The re-merge puts parent commits on top of the head the change was let through at, and without the move they would count as the fixer's change.

Moving the fixer call from inside the pass up into `Call`'s loop would keep all three parts.
The fix review that follows an over-cap change also runs unlocked and can hand back to the fixer, so the same requeue and judgement would be needed there.
What could go, the fourth `again` return the requeue rode on, is counted under the result type.

## Spots left as they were

- The two capped path lists (`listShown`, 10 plain paths "and N more"; `dirtyPathList`, 20 quoted paths ", and N more") render different reason texts, and merging them would change Stuck reasons.
- The repeated `read the task head` wraps stay at their call sites: a shared reader saves no lines, since each site still checks the error.
- `mergeAndGate` stays, since the Finalize tests name it as the unit of one merge-and-gate pass.

## Behaviour notes

- A missing or malformed change description at Finalize now reads `read the change description` as at PR-Review, in place of `parse change description`.
  It is still a returned error, never Stuck.
- An unreadable earlier merge record logs one message for both rows: `earlier merge record unreadable, reading it as absent`.
- Parent-Align moves a scratch attempt record before it reads the row entry; it read the entry first.
  A nil `RowEntry` reads as entry 0 there, as it already did at Finalize.
- The land check's own halt on a failed parent-holds read is gone: that read now happens once, and its failure halts as an owner check, which already ran first and halted on the same failure.

## Verification

`go build` and `go vet` pass for `internal/landingshed` and `internal/shuttleparity`.
The package's tests are left to the row's verify gate.
