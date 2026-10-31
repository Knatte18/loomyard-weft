```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I looked for the test that would fail if it broke. Evidence is the test names the diff adds. Checks that the one-hop greps could not settle are flagged below.

**Per behaviour: pinning test, or none**

- D4 fence open/close rules:
  - `validate_test.go` pins:
    - "a line indented four spaces opens no fence"
    - "a backtick run whose info string holds a backtick opens no fence"
    - "an info-string run inside an open fence is body text"
    - "a run of the other character does not close a fence"
    - "an unclosed fence hides every later section"
  - `append_test.go` pins:
    - the four-backtick outer fence with an inner block
    - the three-space-indented fence
    - the unclosed inner fence inside a longer outer fence
  - **None** for a closing run shorter than the opening run, or a closing line followed by trailing text. The four-backtick-outer and `js`-info rows may cover shorter-run closes; I could not confirm the trailing-text case.
- `missingSections` counts a heading only outside a fence: "only Open risks line sits inside a fence" in `validate_test.go`.
- Record-Write (`TestRecordWrite_Call`):
  - The body-fence sizing rows are:
    - `EmptyBody`
    - `NoBackticks`
    - `InlineCodeOnly`
    - `SectionHeadings`
    - `InnerFencedBlock`
    - `UnclosedInnerFence`
  - Resume behaviours:
    - `ExistingPairIsLeftAsItIsAndStillCommitted`
    - `RecordWithoutSupportLogGetsTheLog`
    - `TornTempFileIsRemoved`
  - Failure paths:
    - `MissingEntryIsStuckNamingBoardUpsert`
    - `SeamErrorsAreReturned`
    - `CancelledContextWins`
  - The commit-only-when-uncommitted behaviour sits in the `Commit` closure in `loomcli/wiring.go` (`recordWriteDeps`, line 370). The tests only see `Commit` as an injected seam. **None** found for the real closure being a no-op on a clean directory; this is unconfirmed.
- Darn-Write (`darnwrite_test.go`):
  - Missing record, restore from HEAD, missing support log: pinned.
  - Stop marker after a Done session, after a gate-failed Stuck, on a respawn, and with no report file: pinned.
  - A stale marker is removed before the session: pinned.
  - A resume whose session writes the marker again halts again: pinned.
  - The guard finding halts with one spawn and no re-prompt: pinned.
  - A gate-failed Stuck is not guarded: pinned.
  - Seam-unwired and `CommittedRecord` error rows: pinned.
  - Marker read ahead of the guard: the Done-session row says it "skips the record guard".
- `decision add` (`decision_test.go`):
  - `decisionWayNoRecord` and `missingRecordRefusal` by row: no status file, Preflight, Record-Write, Drift-Baseline, Drift-Await, Darn-Write, and Darn-Write with a committed record.
  - The not-committed refusal at Record-Write and at Darn-Write.
  - The differs-from-committed refusal for an appended entry, a PATTERN bullet, and otherwise.
  - The missing-historical-entry and entry-lost-after-commit refusals.
  - `--amends-plan` refused without a Plan-Write row ("NoPlanWriteRow_Refused").
  - The refusals name no Darn-Write session. I did not confirm that an explicit negative assertion exists.
- D1 contract (`TestBuild_LoomContract`): no record row, two record rows, no build row, two build rows, build before record, no Finalize, non-terminal after Finalize, no `verify_source`, a valid recipe, and a recipe without the contract.
- D2/D3 verb scope (`arm_seed_test.go`):
  - Pinned:
    - the table in `TestArmAt_RefusesOffLoomRecipes` (run, step, status, pause and goto refuse; start and the plan-verb rows do not)
    - `ArmAtNeverRefusesAnUnservedRecipe`
    - `review_test.go`: pawl resolves, batten refused, an unknown recipe refused with the re-seed, a nil seed reads as tapestry
  - **None** found for plan verbs refusing on a darn run by the Plan-Write row, with the "does not apply" text. `noPlan` at `arm_seed_test.go:194` suggests it exists but I did not see the rows.
  - **None** found for a `beam` seed being refused by `lym loom` with the re-seed text. `review_test.go` has "a target naming no embedded recipe is refused naming the re-seed", which probably covers it.
- D3 peers: `TestHubPeersDeps_OverRealHub` has five rows (seeded darn, empty seed recipe, off-machine project entry, darn entry, plain entry), so the peers behaviour is pinned.
- D11: `seed_test.go` "a beam seed keeps the recipe it records". The `lym shed` unknown-recipe refusal is probably in `shedcli/cli_test.go`; I did not read it.
- D12/D13 config:
  - Pinned: `TestLoadTapestryConfig` covers the null fallbacks, the negative values and the hard cap below the budget.
  - Pinned: `configsync_test.go` covers the loom-to-tapestry key move, and `ensure_integration_test.go` covers the tapestry former units.
  - Pinned: `runsettings_test.go` covers a retired `loom.driver_kind` and a darn arbiter with the shed way forward, and `layers_test.go` covers the retired-key way-forward suffix.
  - **None** found for `KeyCopies` holding no `arbiter*` entry, or for `migrateShedOverrides` being gone. The first is a deletion, and `configreg_test.go` may assert it; I did not confirm.
- D9: `progress_test.go:56` pins the tail order, and `darnrecipe_test.go:149` pins darn's order. Finalize's reroute to Drift-Baseline (card 10) has no row I could confirm.
- D10:
  - Pinned: `TestPreflight_BatcherCheck`, `wiring_test.go:181` (a tapestry seed gets the batcher load), `wiring_darn_test.go:115` (none for darn), and `recipekeys_test.go` for the Tapestry-Preflight retired-row map.
  - **None** found for the D10 bound that a run on Preflight with history past Preflight, a set `start_sha` or `pause_requested` proceeds. `coherence_test.go` and `seed_test.go` were deleted, and a grep of `preflightshed` and `loomshed` tests finds no replacement. The only hit is `loomshed/seed_test.go:64`, an assertion that `PauseRequested` is false on a fresh seed.
- D7: darn's Code-Review rows name the record, with `Test…` rows in `darnrecipe_test.go`. I did not confirm a test that a darn run writes no `board-entry.md` while pawl still does.

**Findings**

1. LOW, class `design`, `internal/loomcli/wiring.go:370` (`recordWriteDeps`). The real discussion `Commit` closure, which must be a no-op on a clean directory, has no test, since `recordwrite_test.go` injects it. A regression would commit empty commits or fail on a resume with nothing to commit. Evidence: `recordwrite_test.go` "ExistingPairIsLeftAsItIsAndStillCommitted" uses the injected seam. Treat this as unconfirmed.
2. LOW, class `design`, `internal/discussionparser/append.go:fencedLines`. No row pins a closing run shorter than the opening run staying body text, or a closing run followed by trailing non-whitespace text. Evidence: the `validate_test.go` and `append_test.go` names listed above. Treat this as unconfirmed.
3. MEDIUM, class `design`, `internal/preflightshed/preflight_test.go`. The D10 acceptance behaviour is untested: a run whose current row is Preflight and whose status holds history past Preflight, a set `start_sha` or `pause_requested` proceeds. The deleted `coherence_test.go` and `tapestrypreflight_test.go` had covered the old refusals, and no new row asserts the proceeding. Evidence: grep found no such assertion in `preflightshed` or `loomshed` tests.

I flag no BLOCKING gaps.

**Tool calls (4)**
1. A `git diff` over the main test files, filtered to test and subtest names; my context held the code but not the tests.
2. A per-file diff grep over the test files for subtest names, for the same reason.
3. Greps for the Tapestry-Preflight, peers, plan-verb, `ArmAt`, key-move and batcher tests, to find tests the diff list did not show.
4. Greps for the `recordWriteDeps` uses, the `start_sha`/`PauseRequested` tests and `RetiredRows` tests, plus `arm_seed_test.go` subtest names.
