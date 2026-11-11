```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set.** Per change, the set found and whether the diff covers it:

1. **Pause exemption (`ExemptPause` seam, `SetGateExemptAll`)**
   - Set: `callerDeps` and `workerDeps` are built only in `hubCallerDeps`, `hubWorkerDeps`, `caller_test.go:294` and `adopt_test.go:255`.
   - All of them fill the seam. The seam's own docs (`doc.go`, `caller.go`, `worker.go`) are updated. Covered.
   - Other users of `SetGateExemptAll` (`fabriccli`, `battencli`, `remoteteardown`) are untouched and need no change.
   - `docs/overview.md:787-803` (adopt entry) does not mention the exemption. See finding 1.

2. **`PushLockWait` and `waitRecordsLocks`**
   - Set: `reconcileRecordsBranch`'s only call site is `originMover.move`, which now passes `opts`. Nothing else in the repo calls `reconcileRecordsBranch`, `tryRecordsLocks` or `waitRecordsLocks`. Covered.
   - `ReconcileOptions` is constructed with `PushLockWait` only in `deps.go`. Every other construction defaults to zero, which keeps the old try-once behaviour.
   - Docs updated: the `originreconcilemoves.go` header, the `ReconcileOrigin` doc, `ReconcileOptions` and `fabricengine/doc.go:195-198`.
   - `tryRecordsLocks`'s own doc ("each by one try-acquire") stays true. Covered.
   - The `overview.md` adopt entry says nothing about the wait. See finding 1.

3. **`ownedUntrackedTrackedAt` / `pathOwnershipUntrackedTrackedAt`**
   - Set: the closed ownership set in `destroy.go` has `resolvePathOwnership` as its only switch. The diff adds the case there.
   - `TestGate_AbsentTargetIsNoOp` lists every kind, and the diff adds the new one. `fabricengine/doc.go` is updated.
   - `KindPathRemoved` already exists, and its consumers (the `DropLocal` doc and mutation recording) are updated. Covered.
   - Pattern entries: none of `PATTERN-gitrepo-client-boundary`, `PATTERN-fabric-boundary` or `PATTERN-lym-files` is touched. The new `gitexec.Run` call in `untrackedOnPaths` goes through `gitexec`, so it meets the boundary rule.

4. **Re-judge of a record stopped at `done` (`rejudges`)**
   - Set: `adoptcli/doc.go:172-178` is updated. The `# Step done` section (`doc.go:403`) still says "the standing record is resumed under either".
   - Since a done-stopped record now restarts at `reconcile`, that sentence is stale. See finding 2.
   - `docs/overview.md:789` ("a re-run of the same subjects follows or resumes it") stays roughly true.
   - Tests: `worker_test.go` adds the row. Covered.
   - The board entry offered "or names `lym adopt --discard` as the way forward when the stops no longer hold". The diff takes the re-judge route only, and the summary states this choice. This is a code-tests and holistic matter, not a set member.

5. **Pattern and commit checks**
   - `PATTERN.md` and `pattern/` are untouched, and the directive declares no PATTERN changes.
   - Commit messages carry no `Pattern-Departure:` trailer, so there is nothing to judge.
   - `PATTERN-dobby-runs` says an adopt pause "lifts only by hand". The diff exempts the adopt's own process from the pause gate and does not lift or alter any pause entry, so the pattern is not violated.

**Findings**

1. **LOW / consistency** — `docs/overview.md:787-803`. The adopt entry now lacks three behaviours the diff adds: the adopt passes any pause, the reconcile waits up to 30 s on a records push lock, and a done-stopped record restarts at `reconcile`. The entry describes the worker's steps in detail and the repository ships overview docs with behaviour changes, so the same diff should have updated it. Evidence: `docs/overview.md:787-803`, with no mention of any of the three; the changed `internal/adoptcli/doc.go:70-73` and `:172-175`.

2. **LOW / consistency** — `internal/adoptcli/doc.go:403`. The `# Step done` section says "Only the local-commit flag differs, so the standing record is resumed under either." The diff makes a record stopped at `done` restart at `reconcile` with its steps cleared, so "resumed" is stale for that case. The new bullet at `doc.go:172-175` states the restart, so the two passages now disagree. Evidence: `doc.go:403` against `doc.go:172-175` and `worker.go:174-176`.

**Tool calls (4 of 6 used)**
1. Bash batch of greps (callers, `callerDeps`/`workerDeps` constructors, ownership-kind users, `KindPathRemoved`, pattern docs, adopt/pause docs, `gitexec` imports): my context lacked the callers and docs.
2. Bash: `docs/overview.md:786-812`, plus the ownership and "closed set" references and drop-local mentions in the docs: I needed the overview text and any doc that enumerates ownership kinds.
3. Bash grep: stale "try-acquire" or "waits on no lock" prose and `ResumeStep` or done-resume claims: to find docs the diff left wrong.
4. Read `internal/adoptcli/doc.go:395-420`: to check the `# Step done` section.
