```yaml
lens: fix-affected-set
outcome: reported
```

**Task:** fix-affected-set lens. I reran the search behind each set-premise finding and found no gaps.

No findings.

Findings checked:
- **F2** (every site that treats an interrupted stop as only failed or paused): the repo-wide grep for "failed or paused", `interrupted-stop` and `AnomalyInterrupted` leaves only the already-updated `anomaly.go` doc comments (lines 29 and 74) as a restrictive statement.
  - The `failed, paused, ...` lists in `battenshed`, `loomcli/start.go` and the orch role stencil are hand-back and halt lists, unrelated to this classification.
  - `loomcli/doc.go:6` states the kind without restricting its states.
  - `ClassifyHalt` has one non-test caller, `loomcli/halt.go:65`. It passes `n.Interrupted` and does not gate on state, so a running or done interrupted halt now gets the anomaly there too.
- **F3** (callers of the changed test seam `SetStartFailuresForTest`): the only caller is `gitexec/startretry_test.go:47`, already updated to the new two-value return.
  - The new `sleepBeforeRetry` variable replaces the only `time.Sleep(delay)` in `gitexec`. The other `time.Sleep(delay)` match, in `reedengine/overlay.go:78`, is a different package and outside this change.
- **F1** (other sites that overwrite the blamed producer with an after-read): `loop.go` has no other `= after.CurrentProducer` assignment. `loopExitStop` already used `cmp.Or(before.CurrentProducer, rec.CurrentProducer)`, so it holds the same fallback rule.
- **F4–F8** are single-site coverage additions with no set premise, so I did not search them.

Tool calls:
1. Grep for the stale-wording and anomaly symbols across `.go` and `.md` files: the set behind F2.
2. Grep for after-producer assignments, `SetStartFailuresForTest` callers and `Interrupted` uses: the sets behind F1 and F3.
3. Read `loomcli/halt.go:50-90`: my context lacked the sole `ClassifyHalt` caller, to see whether it gates on state.
