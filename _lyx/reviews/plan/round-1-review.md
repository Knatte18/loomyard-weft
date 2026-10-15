---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/06-config-terminal-guard.md:38
    summary: Card 6 breaks the existing TestConfigOverRealHub subtest that drives the real menu command over a strings.Reader to pin menu's hub-wide write guard, and its Intent neither rewrites that subtest nor moves the guard's coverage, which no TestMenu row carries
    fix: In card 6's Intent, after the sentence beginning Callers re-signed in this card, add that TestConfigOverRealHub's existing subtest hub-wide write guard - the menu from a task pair is refused naming the prime, which drives Command() with SetIn(strings.NewReader(...)) and so now meets the no-terminal refusal before any listing, is rewritten to call menu directly with terminal true over dirsOf of the code worktree's resolved Location, with dirs.hubWriteGuard set to realHubWriteGuard of that Location, and keeps its assertions (exit 1, a listing naming board, an error naming prime worktree, hub board.yaml unchanged), so menu's hub-write guard keeps its only coverage. The new no-terminal menu subtest through Command() stays as the card already describes it. No target changes; TestConfigOverRealHub is already on the card.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/05-config-empty-edit-abort.md:20
    summary: Card 5 has ErrAborted's comment say a pre-existing file is put back to its previous bytes, but the unchanged-invalid-YAML abort leaves the invalid bytes in a pre-existing file, which TestEdit's row unchanged invalid YAML aborts and keeps a pre-existing file pins, so the comment would contradict the code
    fix: In card 5's Intent, replace the sentence about ErrAborted's comment with one saying the comment scopes the restore to the empty-result abort - on an empty or whitespace-only result a pre-existing file is put back to its previous bytes and a scaffolded file is removed - and leaves the existing description of the other abort paths true, where a scaffolded file is removed and a pre-existing file keeps what the editor left.
  - id: F3
    severity: NIT
    class: consistency
    location: _lyx/plan/05-config-empty-edit-abort.md:29
    summary: Card 5 says editLocal's ErrAborted arm is already reached by TestEditOne's editor-failure row, but TestEditOne drives editOne over worktreeDirs only, and every local-layer row in TestDispatchLayers uses --set or --print with a never-called editor, so no test reaches editLocal's abort arm
    fix: In card 5's Intent, change the clause to say that editOne's ErrAborted arm is reached by TestEditOne's editor-failure row, and drop the claim about editLocal, or name editLocal's arm as one this card leaves uncovered since its reporting does not change.
---

### [MEDIUM] Card 6 breaks the real-hub menu write-guard subtest without saying what replaces it

**Location:** `_lyx/plan/06-config-terminal-guard.md:38`

**Issue:** `internal/configcli/configcli_integration_test.go:188` has the subtest "hub-wide write guard: the menu from a task pair is refused naming the prime".
It builds `Command()`, sets `cmd.SetIn(strings.NewReader(strconv.Itoa(boardChoice) + "\n"))` and runs `menu` through `clihelp.ExecuteIn`.
It asserts exit 1, a menu listing naming `board` before the envelope, an error naming `prime worktree`, and an unchanged hub `board.yaml`.
Under card 6, `runMenu` passes `stdinIsTerminal(in)` to `menu`.
That is false for a `strings.Reader`, and `menu` "checks it first, before printing the list or reading a line".
The subtest therefore gets the no-terminal refusal with no listing and no `prime worktree`, and fails.
The card lists `TestConfigOverRealHub` under `**Edit:**`, but its Intent only says that test's `dispatch` calls pass `true`, and it adds a new no-terminal menu subtest.
`TestMenu` has no hub-write-guard row, since `runMenuWith` calls `menu` with a nil commit and no guard.
So this subtest is the only test that reaches `menu`'s hub-write guard.
Left unspecified, the implementer will either delete the coverage or improvise its replacement.

### [LOW] ErrAborted's new comment would be false for the other abort path

**Location:** `_lyx/plan/05-config-empty-edit-abort.md:20`

**Issue:** `EditPath`'s unchanged-unparseable abort (`internal/configengine/edit.go`, the `string(preEditBytes) == string(postEditBytes)` branch) returns `ErrAborted` without restoring a pre-existing file.
`TestEdit`'s row "unchanged invalid YAML aborts and keeps a pre-existing file" pins `wantFile: ptr(invalid)`.
A sentinel comment saying a pre-existing file is put back to its previous bytes would describe only the new empty-result abort, and would be false for this one.

### [NIT] Card 5 claims coverage of editLocal's abort arm that no test gives

**Location:** `_lyx/plan/05-config-empty-edit-abort.md:29`

**Issue:** `TestEditOne` (`internal/configcli/configcli_test.go:97`) calls `editOne(worktreeDirs(baseDir), ...)`, so its editor-failure row reaches `editOne`'s arm only.
`TestDispatchLayers`' local-layer rows use `--set` or `--print` with `makeNeverCalledEditor`.
`editLocal`'s `ErrAborted` arm is reached by no test, so the card's coverage statement is wrong.
The plan does not change that arm.

## Rejected

- Fidelity: every Decision D1–D7 and every Constraint is carried by a card.
  D1, D4 and D5's webster row are in card 1, D2 and the routed-file removal on a successful begin-batch in card 2, and D3's recipe edge, fasit text, row-neutral cap clause and D5's recipe header in card 3.
  D3's adapter verdicts and D5's three loom rows (rerouted, repeated, reroute-capped) are in card 4, D7 in card 5 and D6 in card 6.
  The constraint that a test pins the recipe's fasit text equal to the `websterengine` declaration is carried by `TestRecipe_WebsterReroutesToPlanReview` against `ConflictRecordRel()`.
- Card order: card 2 edits `plan:internal/webstercli#TestBeginBatchCmd_PlanConflict` and uses `PlanConflictError`, both from card 1.
  Card 3 uses card 2's `ConflictRecordRel`, and card 4 builds on cards 2 and 3.
  Card 6 re-signs `dispatch` after card 5 adds a `TestDispatchHubWideBoard` row, and its "every `dispatch` call" covers that row.
  No card depends on a later one.
- `ErrPlanDrifted` deletion: the repo-wide grep finds it only in `internal/websterengine/beginbatch.go`, three tests in `beginbatch_test.go` that card 1 retargets, and `beginBatchCmd`.
  `record-batch` and `recover-batch` key `plan_drifted` off `ErrFingerprintMismatch`, so they never reference the deleted sentinel.
- Reachability: the begin-batch first-step removal is reached by the `begin-batch x` row, and the plan-conflict write by card 1's two-finding row.
  Run-entry removal after run.lock is reached by the `TestRun_RefusesBeforeSpawn` and `TestRun_EntryHousekeeping` rows.
  The repeated-conflict halt is reached by card 4's equal and reordered rows and its cap-then-repeat test.
  The stale record on an auto-rebaseline refusal is reached by `TestRun_AutoRebaseline` and the `AutoRebaselineWrap` row.
  Every test drives a verb, `Run` or `Call`, never only an error constructor.
  The quarry-unavailable row's missing `WorktreeRoot` passes `batcher.DiskSizes`, which tolerates `ErrNotExist`, and is the same input `TestBeginBatch_ReResolvesPlanAtDispatch` already uses to reach `ErrQuarryUnavailable`.
- Attack surface: card 3's Intent bounds the new `on_reroute` edge by the producer's `Reroute` mark and shed's cap of three.
  Card 4's Intent bounds the routed file and the reroute by stuck outcome, record from this Webster call, differing findings and the cap.
  Card 6's Intent bounds the guard to callers with no terminal and leaves `--print`/`--set` unaffected.
- Member glyphs and re-sign arrows: every Edit card names member glyphs for named symbols.
  Card 6's re-signed `dispatch`, `menu`, `runConfig` and `runMenuWith` carry the arrow, and `Command` and `runMenu`, which keep their signatures, carry none.
- Test budget: by `isTopLevelTest`'s count (TestMain excluded), webstercli goes from 78 to 80 against 81, loomrecipe from 53 to 54 against 54, loomcli from 225 to 226 against 237, and shedadapters from 139 to 140 at most against 156.
  websterengine, configengine and configcli add rows only.
- The `TestBurlerReviewProducer_Call_RoundScan` row with a `CONVERGED` verdict is a new input class for D3's approved-episode claim, not a duplicate of the `CONTINUE` row.
- CLI shape: card 6 adds no command and changes no audience annotation; `config` and `menu` stay `AudienceOperator` leaves.
- `## verify:` runs `go test ./...` and `go test -tags integration ./...` over the single module, which covers every targeted package, including the integration-tagged `webstercli` and `configcli` tests.
  It is in the template's order.
