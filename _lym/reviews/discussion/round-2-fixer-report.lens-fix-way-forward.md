```yaml
lens: fix-way-forward
outcome: reported
```

Task: check that every way forward, step, row and test the fix added can be reached and makes progress.

**MEDIUM: the new D6 criterion puts its assertions in a package that cannot observe them.**
- Changed text: Acceptance criteria, the new line "D6: after a batch of three lands, the push ledger holds one landing per run whose tip is that run's own landing commit, the dobby's landed-commit record of each run names its own landing commit, and the next run to take the lock reads the parent as ahead by owed tips alone. Tier 1, `internal/landingshed`."
- Problem: `internal/landingshed` reaches the push ledger only through the `Deps.LandLocally` seam and the ahead-by-owed read only through the `Deps.ParentAheadByOwed` seam. The real ledger write and the `fabricengine.AheadByOnly` read are wired in `internal/loomcli`.
- Evidence: the real write is `landLocally` → `recordHeldLanding` → `runpusher.RecordLanding` (`internal/loomcli/landingdeps.go:926-975`), and the ahead-by-owed read is `fabricengine.AheadByOnly` (`internal/fabricengine/landing.go:207`). No test file under `internal/landingshed` touches `pushledger` or `AheadByOnly`, per the grep below.
- Effect: as placed, the test can only assert the arguments handed to `LandLocally` and `RecordLanding` (one call per run, each with its own landing commit). It cannot check the ledger's contents or that the next run reads the parent as level.
- Two ways out: reword the criterion to assert the seam arguments, or move the ledger and ahead-by-owed half to a test that drives `internal/loomcli`'s wiring.
- The older D6 criterion's "the push ledger holds one landing per run" (about line 234) has the same gap, but the fix did not touch it.

No other finding. The held path's steps can each be reached from the state they name:
- the held `ownerChecks` (`finalize_owner.go:41-45`);
- `handedOverStop`, whose unread owner is a transient re-step;
- `LandLocally` with the run's own commit;
- the entry removal.

The `Lym-Landing` trailer fallback rests on `fabriclayout.LandingTrailerKey`, written as "Lym-Landing: &lt;slug&gt;" (`refnames.go:113-114`). A run that finds no trailer commit falls back to the parent's head, so it is never left without a commit to record.

Tool calls:
- One grep of `internal/landingshed/*_test.go` for `pushledger` and `AheadByOnly`, plus `internal/loomcli` for `landingParentTarget` and `seedSlug`. My context did not show whether landingshed tests can reach the push ledger.
