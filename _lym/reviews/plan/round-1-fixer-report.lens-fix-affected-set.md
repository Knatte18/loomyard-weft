```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I re-ran each set-premise finding's search and checked that the fix covers every member.

Unchecked: F1, F5, F8, F9, F13 and F15, because none of their premises is a set of sites. I checked F3 and F16 only against what my context already holds, not with a fresh search.

Findings:

- **LOW** — `00-overview.md`, the `Decision: out-of-part` sentence added for F12.
  - Summary: F12's set is the detached status pushes that stay detached, and the fix names only `Teardown.removeCancelledRunRecord`. `tendercli.HubDeps` also has detached spawns: `PushPrimeRecords` calls `fabricengine.SpawnStatusPush(prime, "")` and `PushPair` calls `SpawnPairPush`. No card moves them, and the overview does not say they stay.
  - Evidence: `internal/tendercli/deps.go:141-149`, inside `HubDeps` (`deps.go:42`). The overview says `internal/tenderengine` "need[s] no change" but names neither spawn among the pushes that stay detached.

Checked with no gap:

- **F4.** All tests that set the removed `Spawn` field are now card 8 targets. That covers the five the review named, plus `TestNewCommitStatusSeam_StoppedCommitStartsNoPush` (`commitrecords_test.go:101`), `TestCommitCancelRecord` (`cancel_test.go:1167`) and the five `statuscommit` `TestNew_*` tests, which card 8 already listed. `drivewiring_test.go:137` sets a different `Spawn`, of the loop type, which is not this field.
- **F14.** `internal/fabriccli/doc.go:93` is the only respawn mention outside the fabricengine doc. The fabricengine doc's mentions (`doc.go:1129`, `:1134`, `:1187`, `:1211`) are card 4's doc edit, which this fix did not touch.
- **F2.** The owed-class set is now updated wherever it is stated: the overview decision, card 7 (`FailedPushesOwedBy` and its row), card 8 (`IsPushOwed`, the `Owed` bullet and the `TestIsPushOwed` row) and card 3.

Tool calls:

1. Grepped the test files for `Spawn` and for `commitStatusDeps`/`statuscommit.Deps` literals, to rerun F4's search, which the context lacked.
2. Mapped the remaining `Spawn` lines to their test functions and listed every `SpawnStatusPush` site, for F4's members and F12's set.
3. Read `tendercli/deps.go:135-155` and grepped the doc files for respawn mentions, to identify the tender spawn and to rerun F14's search.
4. Found the function enclosing the tender spawn and `PushPrimeRecords`' callers, to place that spawn.
