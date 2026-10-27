```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: challenge F1 and F2, both fixed as proposed.

**F1 — MEDIUM** (`internal/reportengine/doc.go:131`, "A darn run's setup also holds Preflight, Drift-Baseline and Drift-Await;")
- The proposal's premise is false as far as I could find. It says `darnPhases` in `internal/reportengine/timingspans.go` maps Drift-Await to setup.
- A grep for `Await` and `darnPhases` across `internal/reportengine/`, outside `doc.go`, matched only the shed state `StateAwaiting` in `stops.go`, `timingstops.go` and two test files. Neither `darnPhases` nor "Drift-Await" appears in any non-test file there.
- The reviewer also cited `timingspans.go` and `timing_test.go` as edited by the branch. My first grep for "Drift-Await|Drift-Baseline" in `timingspans.go` printed nothing, so either that file doesn't carry the row names or the mapping isn't there.
- I couldn't confirm the mapping. If the table doesn't map Drift-Await, the doc now describes a darn setup phase the code doesn't implement, and a darn run would count Drift-Await under `other` and raise a finding.
- The loom sentence at line 129 puts Drift-Await under plan, and that edit is not part of this change set.
- The parent should confirm where the darn table lives, for example with `git show HEAD~3 --stat` or `git log -p -S"Drift-Await" -- internal/reportengine`. If the mapping is absent, the doc edit is wrong, or the code edit is missing.

**F2 — LOW** (`contracts/recipes/darn-recipe.yaml:51-52`, "…has no awaited_at, halts at Drift-Verify with no counted baseline, and is settled by hand with lym drift waive.")
- The claim is consistent with the code. `records.go:188` counts a capture only when `awaited_at` is set, and `AwaitedAt` is written only by Drift-Await or `Defer`. `driftengine/doc.go:94` says a missing counted capture is a decision halt that names `lym drift waive`.
- `doc.go:52` says Waive records a waiver only when no counted capture exists, which holds in this case.
- One imprecision remains. A run still in Drift-Baseline with a running capture is not "passed Drift-Baseline", so it isn't covered. That is probably fine, since `baseline.go:34` leaves a running capture alone.
- The header's earlier sentence says runs on removed rows are "restarted, not migrated", while this one says "settled by hand". The two are distinct cases, so I'm not raising it as a finding.

Further tool calls:
1. `grep` for `Drift-Await|Drift-Baseline` in `timingspans.go`, plus `waive` and `awaited_at` in driftengine and driftcli: to check the F2 mechanics. My context lacked the code.
2. `grep` for `Drift-Await` in non-test `reportengine` Go files: to verify the F1 premise. My context lacked the code.
3. `grep` for `Await` and `darnPhases` across `internal/reportengine/` excluding `doc.go`: to look for the darn phase table by name. My context lacked the code.
