Task: fix-proposal lens, checking the reviewer's proposals as the fixer carried them out for F1, F2, F3, F5, F6 and F7.

**Findings**

1. **MEDIUM** — `internal/battencli/commitstatus_test.go`, the new row `RunShedRunningAfterRunShedPausedPushes` (F3 proposal 3).
   - The row does not isolate the `slices.Contains(preRunRows, marker.Producer)` conjunct at `commitstatus.go:264`.
   - After `{Run-Shed paused}` the marker state is `paused`, so the `marker.State == running` conjunct already routes the second call to `core`.
   - The row therefore passes with the producer conjunct deleted, which was the review's own premise for F3.
   - A row that kills it needs a running marker at Run-Shed or later: `{Run-Shed running}, {Seed-Child running}` with 2 commits and 2 pushes.
   - Without the conjunct that sequence would take `startCore` and push once.

2. **LOW** — `internal/battencli/arm.go`, the step-path caller of `seedFreshStatus`.
   - The reviewer proposed the seam call after seeding. A non-owed Worktree-Create push failure now returns from `seedFreshStatus` after the status is already durable.
   - The step path maps any non-refusal error to `shedverbs.KindUnseeded`, which is wrong: the run is seeded.
   - The next run finds the status and resumes, and the pending marker makes the next start row push, so the behaviour is correct.
   - Only the reported kind is misleading.

3. **LOW** — `internal/statuscommit/statuscommit.go`, F6 wording.
   - The sentence "A caller that defers a transition's publication passes a Push that does nothing" is now stated twice, once in the package doc and once in the `Deps.Push` doc.
   - That breaks the say-it-once rule.
   - `Deps.Push`'s first line still reads "pushes the module's run records synchronously".

**Checked, no finding**
- F1: seeding runs in prime before the Worktree-Create row. `PushRun(location, location, …)` pushes the prime's own pair, so it needs no task worktree. A crash between seeding and the commit leaves no marker, so Seed-Child takes `core` and pushes.
- F2: `commitMade` is reset before `core`, a merge-skipped call writes no marker, and an owed push retries until it lands before the reset write.
- F3 proposal 1 (`StartRowWritesTheMarkerSoARepeatSkips`) fails without `startCore`'s marker write, and proposal 2 (`WorktreeCreateAfterAStartRowPushes`) fails without the Worktree-Create exclusion. Proposal 4, `admits == wantPushes`, is equivalent to a literal column on every row.
- F5 and F7 are doc-only and consistent with the PATTERN-run-fsm wording.

**Tool calls**
1. Read `internal/pairteardown/handover.go` `PushRun` with grep, to check whether pushing before the task worktree exists breaks. My context lacked `PushRun`.
