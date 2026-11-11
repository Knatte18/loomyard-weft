```yaml
lens: fix-consistency
outcome: reported
```

The fork's lens is fix-consistency. The change set agrees with its surroundings except for two stale numbers and a stale description in the test comments.

**Findings**

1. **LOW** — `internal/gatecli/testmap_integration_test.go`, the doc comment of `TestGateTestMapPassYields`.
   - The comment says "a hand call naming the package builds it meanwhile" and "It skips the package as queued when another call holds its build lock then."
   - The new row `ResumedPackageIsBuiltAndMeasuredOnce` runs with `handCall=false`, so no hand call builds the package.
   - The comment now describes only two of the three rows.
   - Evidence: the `tests` table gained the `handCall` field and a third row, but the function comment was not touched.

2. **LOW** — `runPassYields`, the comment "The hanging test would time out after 20 seconds, so the gate run's take comes first."
   - The pass's `secondUnit` was changed from `time.Second` to `500*time.Millisecond`.
   - With `redundancy_test_timeout_sec: 20`, the timeout is now 10 seconds, so the comment's number is stale.
   - The same change also reaches the other two rows. Only the 20 s to 10 s timeout number is stale.

3. **LOW** — `internal/gatecli/testmap.go`, the new constant `testMapRegistrationLockName`; the `.lym` scratch directory is documented as a list of named entries.
   - `internal/fabricengine/doc.go` lists the scratch-directory members, for example `teardown/<slug>.lock` at line 1160 and `<HubScratchDir>/runproc` at line 1198.
   - It also lists `hub-workspace.lock`, via `hubworkspace.go:51`.
   - The new `<HubScratchDir>/testmap-upkeep.lock` is not listed anywhere.
   - I did not search whether a test enumerates the directory's entries.
   - I read nothing else of `doc.go`, so whether it lists locks exhaustively is unverified.
   - It may be fine to leave this unlisted.

**Checked and consistent**
- `audit`'s doc comment now says it reads HEAD before each package job, matching the `mapPackages` comment and the Long help's "ends once HEAD moves".
- `r.measures` and `r.built` doc text still holds: a resumed package is one entry, and `PackageMeasure`'s "time one package's measurement took" still reads correctly as a sum.
- `registerUpkeepPass`'s refusal message and return values are unchanged, so the `runTestMap` caller and the refusal tests are unaffected.
- The lock path sits under `.lym` and not under `.lym/locks`. `refuseTrackedTree` resets `inBolt` at the `.lym` segment, so it is not refused.

**Further tool calls**
1. One `grep` over `*.md` for `testmap-upkeep`, `.build.lock` and `runproc`, to see whether docs list the scratch-directory locks. My context held no docs.
2. The same call also ran `grep` for `HubScratchDir` in `*.go` to see which files name members of the scratch directory.
