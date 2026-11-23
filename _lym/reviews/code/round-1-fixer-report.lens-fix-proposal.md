```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the proposals the fixer carried out as proposed (F2, F3, F4, F5, F6, F8).

**Findings**

1. **MEDIUM: F2's proposal leaves #203's refused hand pass unfixed.**
   - Changed text: `reconcileRun`, `reconcileGoneRun` and `reconcileEnded` now take `RunStartReconcileOptions()`.
   - The review's summary cites `catchup.go:66`, "origin is unreachable". That refusal comes from `catchUpFailure` for `dobbyengine.ErrOriginUnreachable`, not from the reconcile.
   - `ErrOriginUnreachable` is produced by `pullBoltForCatchUp` (`catchup.go:169`), `registerThisMachine` (`catchup.go:188`), `fetchRunForCatchUp` (`catchup.go:227`) and `hubRunVerdict`'s fetch (`arm.go:642`).
   - None of them retry, and the fix touches none. A hand pass whose catch-up fetch drops still refuses at once, and the tests don't cover it.
   - The proposal named only the reconcile sites, so the review's own premise stays unresolved.

2. **MEDIUM: F2's 2-minute bound stalls every run start while origin is down, holding the hub's remote-refs locks.**
   - Changed text: `runStartRetryBound = 2 * time.Minute`, applied in `Command()` and `StartFirstPass`.
   - `judgeAgainstOrigin`'s doc (`originreconcile.go:862-863`) says the retries and their sleeps run under both repositories' remote-refs locks, so an unreachable origin blocks every fetch and every push's tracking-ref write for that bound.
   - Before, `reconcileResume` met `report.Unreachable` after one attempt, logged a Warn and resumed.
   - Now each `lym batten run` pass (reconcile skipped only for continue passes) and each beam leftover teardown blocks 2 minutes first.
   - A run on a deliberately offline machine pays 2 minutes per pass, and concurrent runs queue behind the locks.
   - A shorter bound, or a retry outside the locks, would address the review's "a minute or more" without that cost.
   - Nothing in the fix mentions or tests this cost.

3. **LOW: F3's blanket 4xx guard also silences retryable 408 and 429.**
   - Changed text: `httpClientRefusalPhrases` (`"the requested url returned error: 4"`, `"rpc failed; http 4"`).
   - A GitHub fetch that fails with "The requested URL returned error: 429" and then "Recv failure" now returns false.
   - Before the fix, a bare 429 was already false, but a 429 followed by a disconnect phrase was true.
   - A narrower guard would exclude 408 and 429 and keep every other 4xx a refusal.
   - The review proposed the blanket form, so this is a cost of the proposal, not a slip in carrying it out.

**Checked, no finding**
- F4, F5: the rows fail with the defect in place. If `wait` ran after `recheck`, `waitMovesTo` would not be read at `recheck`, giving a re-step record instead of `SettleMoved`.
- `TestGoDriver_Wait`'s ended-context row and the Wait-error row pin the exits the review named.
- F6: both docs now say an unwired `Wait` removes no row.
- F8: the split rows each isolate one phrase.

**Tool calls:** none made. The context already held `catchup.go`, `arm.go`, `originreconcile.go` and the changed files.
