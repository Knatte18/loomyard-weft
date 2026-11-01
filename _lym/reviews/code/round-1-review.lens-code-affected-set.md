```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set. For each change the diff makes, I looked for what reads, calls or asserts it outside the touched lines, and whether the diff left it wrong.

**List: per change, the set found, covered or not**

1. `removeRunRecords` now commits only while `runRecordsTracked` is true.
   - Callers: `cancelcmd.go:305` (`RemoveRecords`) and `cleanup.go:217` (`cleanupSlug`). Both pass `fabricCleanupDeps`, or the test stubs, which set `Committed` (`cleanup.go:76`; `cleanup_test.go:56`). Covered.
   - Nothing builds a `cleanupDeps` without `Committed`, so the new `deps.Committed` call cannot hit a nil function.
   - `cleanupSlug` judges through `judgeRunDirectory`, which reads the same two paths via `deps.Committed` (`cleanup.go:325`). A candidate with the directory present but untracked used to commit; now `removeRunRecords` skips the commit, and `report.Removed` still lists the slug. That matches the new doc text. Covered.
   - The cleanup fixture change at `cleanup_test.go:89` (seed marked tracked) keeps the existing `TestCleanup` rows on the commit path. Covered.
2. `RecordsPresent` refactored onto `runRecordsTracked` (`cancelcmd.go:241`). This is the same two-path read as before, now shared. Covered.
3. `cli.go` doc line on `--remove` after a plain cancel. The cancel verb's `Long` help text in `cancelcmd.go:53-81` was not updated, so it does not mention an already-removed records directory counting as removed. LOW at most; it is not a finding on its own, because the help text only says it "removes the run's records".
4. `HoldRemoteRefs` now admits the process through `pauserecord.Admit` (`remoterefslock.go:30`).
   - Callers: `withRemoteRefs`, which every fetch, pull and push goes through, and `fabricengine/originreconcile.go:761`, which holds two locks in a loop. Each caller calls `Admit` once per lock, so `admissions` counts up to 2 and is released in the deferred order. Covered.
   - Docs: `gitrepo/doc.go`, `pauserecord/doc.go`, the `PATTERN.md` entry and the `HoldRemoteRefs` comment all landed in the diff. `fabricengine/doc.go:70` describes the reconcile holding the locks and says nothing of the gate, so it is not wrong.
   - `PATTERN-remote-refs-lock` has no background file, so there is none to update. The new entry text matches the code.
   - Readers of the in-flight markers: `orchengine/pause.go:201` and `tendercli/deps.go:124`. They read `NetOp.Op` or `Strand` and match on neither the op name nor its count. The new `remote-refs` marker, written at `Admit`, now lasts from before the lock is taken until its release. Before, an in-flight marker lasted only for one git call.
   - The consequence is that a pause started during a held lock sees an in-flight network operation for the whole hold, including `ls-remote` retries and their sleeps (`originreconcile.go:753-758`). The orch pause logic (`orchNetworkOperationInFlight`) and the tender's `NetOps` seam read this as busy, and their docs say "in flight" for a single call. `pauserecord/doc.go:33` was not updated to say a marker can span a lock hold. Neither reader can mistake it for a hang, but the effect is wider than the diff's docs state. I found no test or doc that pins the old duration. LOW, `consistency`.
5. `Network` with `admissions > 0` skips the gate for every call in the process. Callers: `gitexec.go:232` and `githubclient/transport.go:47`. A GitHub call on another goroutine of the same process also passes while an admission stands. `pauserecord/doc.go:39` says "every network call of the process", which matches. No caller depends on the gate holding a particular call, so the set is covered. The effect on unrelated goroutines is the handler's question, not this lens's.
6. Declared PATTERN changes. The directive declares none (no declared-changes list appears in the constraints section). The `PATTERN-remote-refs-lock` text was changed in the diff regardless, and it is consistent with the code. Nothing to raise.
7. Commit messages. Neither carries a `Pattern-Departure:` trailer. The only one needed would be for the answer key's first idea (a cancel never waits on its own pause step), and the diff takes the other route (narrowing the lock holder path), so it needs no trailer. Whether that route meets the Goal is the handler's scope question.

**Findings**

- LOW, `consistency`: `internal/pauserecord/doc.go:33-39` and `hold.go:124`. The `NetOp` in-flight marker now also spans a whole remote-refs lock hold, readable by `orchengine/pause.go:201` and `tendercli/deps.go:124`. The docs still describe it as one operation in flight, and `Admit`'s comment mentions "reads its calls in flight" only. Evidence: `remoterefslock.go:30-40`, `hold.go:424-431`.

No other finding.

**Tool calls**
1. `grep` for `ListNetOps` readers and for docs mentioning `remote-refs`: my context lacked the consumers of the in-flight markers.
2. `Read` of `internal/orchengine/pause.go:185-235`: to see how the orch reads the markers.
3. `sed` of `internal/tendercli/deps.go:110-140` plus `grep` of `Op` in `hold.go`: to see the tender's `NetOps` seam and the `NetOp` field use.
