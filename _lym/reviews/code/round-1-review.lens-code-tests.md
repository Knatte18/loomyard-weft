```yaml
lens: code-tests
outcome: reported
```

The lens was code-tests: for each behaviour the diff adds or changes, find the test that would fail if it broke. The behaviours mostly carry real coverage. The one real gap is that a failed head read no longer has a test proving it fails Finalize when `LandLocally` is wired.

**Per behaviour, the test that pins it**

1. **Finalize hands the head it reads after the merge to `Deps.LandLocally` as the tip** (`internal/landingshed/finalize.go:570-572, 1054`). Pinned by `TestLandingDeps_LocalLanding/MergesRecordsTheLandingAndPushesNothing`, which requires `Landings[0].Tip == f.primeHead()` (`internal/loomcli/landlocally_integration_test.go`, the `entry.Landings` check). Passing the wrong commit or an empty one fails it.
2. **One head read feeds both `Deps.RecordLanding` and `Deps.LandLocally`.** No test tells one read from two. The fake returns the same `headSHA` on every read after the merge (`finalize_test.go:83-87`). For a lone run under the lock the two reads cannot differ, so this is a refactor with no observable difference. Not a finding.
3. **With `LandLocally` nil, a failed head read is only logged and the outcome stays Done.** Pinned by `TestFinalize_RecordLandingFailureKeepsDone/HeadReadFails` (`finalize_test.go:487`). `newTestDeps` leaves `LandLocally` nil (`publish_test.go:71-103`).
4. **With `LandLocally` wired, a failed head read fails the call before the seam runs** (`finalize.go:1051-1053`). No test pins this; see finding 1.
5. **`recordLanding` records nothing when the head read fails.** Pinned by the same `HeadReadFails` row, which expects 0 calls to `RecordLandedSize`.
6. **`writeLedgerLanding` writes the slug, tip, records commit and branch it is given.** Pinned by `TestWriteLedgerLanding_AChainOfOwnTipsReadsAsOwed`'s loop over every landing (`landlocally_integration_test.go:514-518`).
7. **`writeLedgerLanding` replaces a run's standing landing.** Pinned by the same test, which requires one landing per run after the re-record (`:510-512`).
8. **A parent ahead by a chain of landing commits, each its run's own tip, reads as ahead by owed tips alone.** Pinned by `requireAheadByOwed("with every landing commit recorded", true)` (`:503`). If the writer used the parent's head as every tip, all tips would collapse to run three's commit and this check would fail.
9. **A run recorded again through `landLocally` keeps its own tip and its earlier records commit.** Pinned by `:509` and the landing loop (`:514-518`). `TestLandLocally` already pins the kept records commit. The new test adds the tip and the other runs' landings staying intact.
10. **`recordHeldLanding` writes through `writeLedgerLanding` after `oldestHeldRecords`.** Pinned by `TestLandLocally` and by `TestLandingDeps_LocalLanding`.
11. **A failed parent branch read inside `writeLedgerLanding`, now after the status-records commit.** No test; see finding 2.

**Findings**

1. **MEDIUM, design (coverage).** `internal/landingshed/finalize.go:1051-1053`
   - Nothing pins that a failed head read fails Finalize when `Deps.LandLocally` is wired.
   - The diff rewires exactly this path by passing `headErr` through, and `_lym/landing/summary.md` claims it "still fails the call when one is".
   - The one row that injects `headErr`, "a failed read of the parent's head under the lock" (`finalize_unlocked_test.go:389-393`), wires no `LandLocally` (no match in that file). It is satisfied by the earlier read at step 1b4b (`finalize.go:507-510`), which uses the same message.
   - The only test that sets `LandLocally` (`finalize_test.go:202`) never sets `headErr`.
   - If the `headErr` check were dropped, `LandLocally("")` would be called, and the empty tip would go into the ledger with no test failing.
   - Fix: a `finalize_test.go` row with `LandLocally` wired and `headErr` set. It expects an error wrapping `headErr` and no `LandLocally` call.
2. **LOW, design (coverage).** `internal/loomcli/landingdeps.go:979-982`
   - Nothing covers a failed parent branch read in `writeLedgerLanding`.
   - It now runs after `commitRunStatus` under the pair push lock, so a failure leaves a status-records commit with no ledger landing holding it back.
   - No test pins what that failure returns or leaves behind (evidence: the new test and `TestLandLocally` drive only successful branch reads, `landlocally_integration_test.go:401-518`).
3. **LOW, consistency (test economy).** `internal/loomcli/landlocally_integration_test.go:498`
   - The `requireAheadByOwed("with the last landing commit unrecorded", false)` check would also pass if the behaviour were wrong.
   - If `writeLedgerLanding` used the parent's head as the tip, runs one and two would both carry run three's commit. The parent would still read as not ahead by owed tips alone, so the row passes.
   - "An unrecorded commit reads as not owed" is already pinned at `AheadByOnly` by `internal/fabricengine/landing_integration_test.go:125-126`.
   - The acceptance criterion does not ask for this case. Drop the row, or say in the test's doc comment why it is there.

**Tool calls**
- `grep` for `AheadByOwed`/`AheadByOnly` in the loomcli and fabricengine tests, `headErr` and the head-read message in the landingshed tests, and `newTestDeps`'s `LandLocally` wiring: the inherited context had not looked for existing tests of these behaviours.
- `sed` of `finalize_unlocked_test.go:360-420` plus `grep LandLocally` in it: to see whether the head-read failure row reaches the read in `pushParent` or the earlier one.
