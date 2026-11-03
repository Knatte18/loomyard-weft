```yaml
lens: fix-code-correctness
outcome: reported
```

No findings.

Task: check the changed code (`refuseKeptPause` in Submit, the `Watch` keep-path `recordOwned`, and the added `TestSubmit`, `TestWatch` and `TestChooseResumeOnWake` rows) for correctness.

What I traced:

- **`refuseKeptPause`, terminal request with a live owned entry:** `TenderStatus` returns the entry in `Live`, and `Scope`, `Caller` and `PausedAt.Equal` all match. Submit is refused with "lym hibernate cancel" and "lym tender resume", and the file and spawn count stay unchanged.
- **`refuseKeptPause`, other inputs:**
  - An entry already lifted, or `Owned` nil: Submit passes.
  - A `TenderStatus` error: Submit is refused with a wrapped error.
  - A later operator hub pause: it has a different `PausedAt`, so it never matches.
- **`refuseKeptPause` caller:** the real `Deps` is built in `internal/hibernatecli/deps.go:65`, and it sets `TenderStatus` at `deps.go:91`. No other code constructs `hibernateengine.Deps`, so no caller is left with a nil `TenderStatus`.
- **`Watch` keep path:**
  - `Owned` cleared during the sleep: `advance` re-reads the file, so `r.Owned` is nil and `target.Owned = kept`. `recordOwned` writes the entry, and `finish` then writes done with `Owned` set.
  - `ResumeOnWake` true: the first switch case wins, so `recordOwned` never runs.
  - `kept` nil: the second case is skipped.
  - A `recordOwned` failure: the entry stays live, and the error is joined into the return value alongside the notice error.
- **Added `TestSubmit` row:** with the defect in place Submit would succeed, so `requireRefusal` fails. The lifted-entry half also checks the spawn count.
- **Added `TestWatch` rows:**
  - "a failure before the hibernate lifts the entry of a request that keeps its pause after the wake": if `fail` followed the request's switch, the entry would stay live and the notice would carry `keptPauseOutcome`, so the row fails.
  - "a request that keeps its pause and cannot write the resuming phase leaves the entry live and the notice says so": the row turns the request file into a directory. `advance` then errors, so `failAtWake` runs with `r.Owned` taken from the earlier `advance` and keeps the entry. If that call were reverted to `fail`, the entry would be lifted and the row fails.
  - "a request that keeps its pause and lost its owned record at the wake ends done with the kept entry recorded as owned": `final.Owned` stays nil without the new `recordOwned`, so the row fails.
  - "a request that keeps its pause and owns none ends done with a notice saying it owned no pause": the pre-existing EXTERNAL hub entry means `kept` and `Owned` are both nil, so `keptOutcome` returns the owned-no-pause sentence. If it regressed to the kept sentence, the row fails.
  - "a failure at the wake of a request that keeps its pause and owns none says it owned no pause to lift": `target` is nil and `lift` is false, so the notice ends "it owned no pause to lift."
- **`TestChooseResumeOnWake` rows:** the unparsable-file row only passes if `LoadConfig` returns an error, and a gate run of both packages passed. The flag-given row would fail if the file were read, because that would return an error.

**LOW observation, not a finding.** `Submit` now calls `TenderStatus` (a hub-wide read) while holding the request lock, and only for a terminal request that owns an entry. The daemon's writes wait on that lock for that long. It is bounded and rare, and I did not treat it as a defect.

Tool calls:
1. `grep` for `TenderStatus` and `hibernateengine.Deps{` outside the engine package, plus the submit calls in `cli_integration_test.go`. I needed it to confirm that no `Deps` construction lacks `TenderStatus` and that no integration test submits twice over a kept pause. My context did not hold `deps.go` or the integration test.
