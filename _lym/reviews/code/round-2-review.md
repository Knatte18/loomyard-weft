---
verdict: APPROVED
prior:
  - id: F1
    status: resolved
    answer: RemoveBenchPairs now calls td.endOwned before the archive and td.Run, so a registered holder of the bench pair's run lock is gone when awaitOrPass waits; the row ARunLockHeldByALiveProcessOfABenchPairIsEndedNotRefused pins it, and the doc comment, pairteardown/doc.go, PATTERN-pair-remote.md and the summary state it.
  - id: F2
    status: resolved
    answer: The call moved to the Teardown.Push closure, which the producer runs between its two prime-lock holds, and the wire comment and battenshed/doc.go say so.
  - id: F3
    status: resolved
    answer: benchPairAdmitted stats shedrun.RunDir on the prime and refuses an existing directory; the add and remove rows for slug-b3 pin it, and the board-only gap is stated in fabriccli/doc.go, the benchPairAdmitted comment and the summary.
  - id: F4
    status: resolved
    answer: The door reads one-way; the spec at contracts/specs/loom-final-summary-spec.md:42 makes the blast radius one word, and the prose under it names the guard admission, the process end, the removal and the pushed tags.
  - id: F5
    status: resolved
    answer: TestWiredTeardown_RemovesTheBenchPairsBeforeTheTaskPair drives the wired row for both outcomes and checks both pairs and the Stuck reason.
  - id: F6
    status: resolved
    answer: warnHalfRemovedBenchPair logs at Warn for both skip cases with the slug, the reason and `lym fabric prune --apply`.
  - id: F7
    status: resolved
    answer: benchPairVerb returns the read error, and sandboxedRoleRefusal appends it in place of the way forward; the UndecodableRunRecord row checks both.
  - id: F8
    status: resolved
    answer: The UndecodableRunRecord row is added; the two-argument case cannot reach the guard past cobra's arity check, and NoSlug pins the reachable branch.
  - id: F9
    status: resolved
    answer: The error path now asserts wantRemoved, and dotgit.ListWorktrees sorts linked worktrees by path, so task-b1 goes before the refused task-b2.
  - id: F10
    status: resolved
    answer: isBenchPairPath now runs benchPairAdmitted on the writer's own worktree, and the WriterOnAnotherProfile and BenchSlugABattenRunNames push rows pin it.
  - id: F11
    status: resolved
    answer: The new Pair teardown bullet names the route, the process end, the archive, the skipped driver checks and the ordering exception.
---

This round found no new finding, and every prior finding is resolved.

## Focus questions

- **Bench pair with a live driver or a held run lock.**
  `RemoveBenchPairs` first ends the pair's registered processes through `td.endOwned`.
  That end spares the calling batten process and its ancestors, teardown children and the orch.
  The ended verdict then skips the quiet wait, and `awaitOrPass` finds the run lock free.
  A holder the registry does not reach is still refused after `runLockWait`, which is the existing rule for unregistered holders.
  The doc comment, `internal/pairteardown/doc.go`, `pattern/PATTERN-pair-remote.md` and the summary all say a live process is ended rather than refused, and the pattern bullet names the ordering exception correctly.
- **Prunable or unresolvable bench worktree, and partial failure.**
  Both skips log a Warn naming `lym fabric prune --apply` and return nothing.
  A failure returns the slugs removed before it, and the refused-archive row pins that.
  No test row drives the prunable or unresolvable case, and the wire drops the partial list on error, logging only the wrapped error.
  This incremental round raises new findings only at BLOCKING, and neither is a BLOCKING defect, so both are recorded under Rejected.
- **Locks and resume of the wired row.**
  The bench removal runs inside `Push`, between the producer's two prime-lock holds.
  Each archive takes the slug's teardown lock, and the removal takes fabric's own locks.
  A failure returns before `teardownPush`, the producer reports Stuck with "the push step failed and the pair is kept", and the task pair stays.
  On resume the shutdown is re-run over a session already down, and the archive reuses the tag names its tips give.
  `TestWiredTeardown_RemovesTheBenchPairsBeforeTheTaskPair` pins the order and the failure.
- **Guard admission.**
  `benchPairVerb` admits only `add` and `remove` with one bench-form argument of the writer's own slug.
  The verb must run from the writer's own worktree or its records sibling in its own hub, with no `--parent` or `--adopt-local` on `add`.
  The run record must name darn and docs, and no batten run directory may name the slug.
  A writer on another recipe or profile, or with no record, gets the bench way forward.
  An undecodable record or a batten-named slug gets the reason instead.
  A slug only a board entry names passes, which the doc and the summary state.
  `roleguard_test.go` has a row for each refusal branch.
- **Merge danger.**
  The one-way door and the prose under the one-word radius name the guard admission, the process end, the removal and the tags pushed to origin.

## Rejected

- No test row drives a prunable or unresolvable bench worktree in `TestRemoveBenchPairs`.
  It is a coverage gap in a log-only path, not BLOCKING, so this incremental round does not raise it.
- On a failure, the wired `Push` closure in `internal/battencli/wire.go` does not log the bench pairs removed before the failure.
  Each removal goes through `td.Run`, which logs its own session end, so this is not BLOCKING.
- `isBenchPairPath` places the writer's task worktree as `filepath.Join(pair.HubPath, name.Slug)` rather than through `lymcwd.NamedWorktree`.
  The result is checked by `ownPairLocation` right after, so a layout change refuses rather than admits; not BLOCKING.
- `RemoveBenchPairs` runs for every run's teardown, whatever its recipe or profile.
  A non-docs run's `<slug>-b<N>` pair that no board entry or batten run names is an orphan the dobby's sweep would also take, archived first; not BLOCKING.
