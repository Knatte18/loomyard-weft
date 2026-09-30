---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    location: _lyx/plan/15-batten-decision-resume.md
    summary: Card 15 renames InnerRunDeps.ReadApproval and ChildApproval but omits shedrecipe/entries_batten.go, entries_batten_test.go and battencli/wire_test.go, so the build breaks
  - id: F2
    severity: BLOCKING
    location: _lyx/plan/12-landingshed-publish-split.md
    summary: Card 12 leaves publish_verify_test.go asserting Awaiting five times and calling newApprovalFixture, which the same card deletes, so landingshed stops compiling
  - id: F3
    severity: BLOCKING
    location: _lyx/plan/10-shedrecipe-review-entries.md
    summary: Card 10 registers PRGate and PRRework without extending shedbuild's newTestEnv, so TestBuild_EveryRegisteredEngineBuilds fails for both new engines
  - id: F4
    severity: MEDIUM
    location: _lyx/plan/11-loom-recipe-review-rows.md
    summary: Card 11 fills only ReworkSpec and Rework in the loomrecipe fixtures, but testLandingDeps lacks ApprovalPath, RejectionPath and TaskHead, which NewPRGate refuses
  - id: F5
    severity: MEDIUM
    location: _lyx/plan/16-pr-fixback-docs.md
    summary: Card 16 misses three overview.md statements D2 makes false and the loom-status-spec awaiting-at-Publish comment
  - id: F6
    severity: MEDIUM
    location: _lyx/plan/06-websterengine-rebaseline.md
    summary: Card 6's runlevel_test seeds a mid-run state, not a finished Webster run, so it does not prove the stale-outcome archive and integration re-run D6 promises
  - id: F7
    severity: LOW
    location: _lyx/plan/09-loomshed-pr-rework.md
    summary: Card 9 does not say what the round directories are enumerated from, though ReadCommitted reads one file per call
  - id: F8
    severity: LOW
    location: _lyx/plan/09-loomshed-pr-rework.md
    summary: A crash between ClearRejection and the persisted Done re-enters the rework row with no record, which returns Stuck and can never be cleared, since reject refuses at the moved HEAD
  - id: F9
    severity: LOW
    location: _lyx/plan/12-landingshed-publish-split.md
    summary: Card 12's ImpactSummary restates Intent instead of naming the consumers of Publish's Awaiting verdict
  - id: F10
    severity: NIT
    location: _lyx/plan/01-landingshed-rejection-record.md
    summary: Card 1's ImpactSummary counts three WriteApproval callers; there is one production caller and several test callers
---

### [BLOCKING] Card 15 misses three users of the renamed seam

**Location:** `_lyx/plan/15-batten-decision-resume.md`

**Issue:** `grep ReadApproval|ChildApproval` finds three files that card 15 omits and no other card edits.
`internal/shedrecipe/entries_batten.go:130` is production code: `requireSeam("InnerRun", "InnerRun.ReadApproval", env.InnerRun.ReadApproval)`, and its doc comment at line 87 names the seam.
`internal/shedrecipe/entries_batten_test.go:80` zeroes `env.InnerRun.ReadApproval`, and `internal/battencli/wire_test.go:73` asserts `c.env.InnerRun.ReadApproval != nil`.
After the rename, `shedrecipe` and the `battencli` tests fail to compile.
The ImpactSummary's claim that battencli's wiring is the only production user is also false, because `entries_batten.go` is one too.

**Fix:** Add `internal/shedrecipe/entries_batten.go`, `internal/shedrecipe/entries_batten_test.go` and `internal/battencli/wire_test.go` to card 15's Edit list.
Say in Intent that the `requireSeam` call, its doc comment and both tests follow the rename, and correct the ImpactSummary.

### [BLOCKING] Card 12 leaves publish_verify_test.go broken

**Location:** `_lyx/plan/12-landingshed-publish-split.md`

**Issue:** `internal/landingshed/publish_verify_test.go`'s fixture serves the no-PR list and lets Publish create the pull request.
Five tests (`TreeChangedPass`, `UpToDateNoMarker`, the `UpToDateWithMarker` pass case and both `EmptyCommand` cases) assert `shedengine.Awaiting` after a passing verify, and card 12 turns that result into `Done`.
`TestPublishVerify_ApprovalPathRunsNoVerify` (line 257) also calls `newApprovalFixture`, which card 12 removes from `publish_test.go`, so the package stops compiling.
No card edits this file.

**Fix:** Add `internal/landingshed/publish_verify_test.go` to card 12's Edit list.
The five passing-verify assertions expect `Done`, and `TestPublishVerify_ApprovalPathRunsNoVerify` is removed with the other approval cases.

### [BLOCKING] Card 10 breaks shedbuild's every-engine build test

**Location:** `_lyx/plan/10-shedrecipe-review-entries.md`

**Issue:** `internal/shedbuild/build_engines_test.go`'s `TestBuild_EveryRegisteredEngineBuilds` builds every name in `shedrecipe.Names()` against `newTestEnv` in `internal/shedbuild/fixture_test.go`.
That fixture's `testLandingDeps` sets no `ApprovalPath`, `RejectionPath` or `TaskHead`, and `NewPRGate` refuses each one (card 7).
`newTestEnv` also sets neither `ReworkSpec` nor `Rework`, which `prReworkEntry` requires.
Once card 10 registers the two keys, both subtests fail.
Card 15 lists `shedbuild/fixture_test.go`, but only for the batten rename, and it lands after card 10.

**Fix:** Add `internal/shedbuild/fixture_test.go` to card 10's Edit list.
Its `testLandingDeps` fills `ApprovalPath`, `RejectionPath` and `TaskHead`, and its `newTestEnv` fills `ReworkSpec` and every `Rework` seam, as the fixture's own doc comment requires of every new registry entry.

### [MEDIUM] Card 11 leaves the loomrecipe landing deps short of the gate's required fields

**Location:** `_lyx/plan/11-loom-recipe-review-rows.md`

**Issue:** `internal/loomrecipe/fixture_test.go`'s `testLandingDeps` sets no `ApprovalPath`, `RejectionPath` or `TaskHead`.
`shape_test.go` and `buildSequenceFixture` both use it, and once the recipe carries `PR-Gate`, every `New(env, paths)` in the package reaches `NewPRGate`, which refuses all three.
Card 11 edits `fixture_test.go` but says only that the test `Env` gains `ReworkSpec` and `Rework` fakes.
Otherwise, card 11 does not change the halt `sequence_test.go` and `resume_test.go` assert at `NamePublish`.
In that fixture, Publish halts `Stuck`, and card 11 changes only `Publish.on_done`, so `on_stuck` stays empty and the run still blocks at Publish before the gate.

**Fix:** Card 11's test sentence also says `testLandingDeps` fills `ApprovalPath` and `RejectionPath` with temp-dir paths and `TaskHead` with a stub closure.

### [MEDIUM] Card 16 misses statements that D2 makes false

**Location:** `_lyx/plan/16-pr-fixback-docs.md`

**Issue:** The docs-in-the-same-commit Constraint covers `docs/overview.md`'s loom, landing and batten text, and D2 moves the PR-review hand-off from Publish to the gate.
Four live statements carry the old shape, and card 16 names none of them:
- `docs/overview.md:249`, the module tree: "landing's two general ShedProducers, Publish and Finalize". The package now also holds `PR-Gate`.
- `docs/overview.md:369`: "`Publish` returns Done on a valid approval (matching the branch's current HEAD) before any sync". Card 12 deletes that behaviour. Card 16 rewrites the landing-tail line but not this sentence.
- `docs/overview.md:411`: "(it does restart an approved `awaiting` one)". Under D8, batten also restarts a rejected one.
- `contracts/specs/loom-status-spec.md:43`: "awaiting is the planned PR-review hand-off at Publish". Under D2, that hand-off is at the gate. No card touches this spec.

**Fix:** Card 16 names each statement and its replacement.
Add `contracts/specs/loom-status-spec.md` to its `Prosa` targets.

### [MEDIUM] Card 6's resume test does not model a finished Webster run

**Location:** `_lyx/plan/06-websterengine-rebaseline.md`

**Issue:** The rework row re-baselines a Webster run that has already finished.
A finished run has every batch complete and leaves `outcome.yaml` and `summary.md` in the webster directory and an integration report in the reports directory.
`Run` archives the first two through `archiveStaleOutcome`/`ArchiveStaleSummary` and removes the third (`runlevel.go`).
The card seeds only batch records, which is an ordinary interrupted-run resume.
That test would pass even if the stale-artifact handling D6 relies on regressed, so it does not prove the acceptance criterion's "followed by the integration stage" over a finished run.

**Fix:** Seed the finished-run artifacts: batches 1–2 complete, a done `outcome.yaml`, a `summary.md` and an integration report.
Assert that `Run` archives the outcome and summary, removes the report, and renders only batch 3 as pending, with the integration prompt written.

### [LOW] Card 9 leaves the round-directory enumeration unstated

**Location:** `_lyx/plan/09-loomshed-pr-rework.md`

**Issue:** Step 2 looks for a `round-<N>` directory committed at HEAD, and step 5 picks N from the working tree.
`ReadCommitted` reads one anchor-relative file per call and cannot list directories, and the card does not say where the candidate round numbers come from.
The declared seams can still implement D5: every committed round directory is also present in the working tree, so listing `ReworkDir` and reading each `ReworkDirRel/round-<N>/record.json` through `ReadCommitted` suffices.
But the implementer has to infer this.
Step 4 matches the other cards: its `ParsePlanFrom` closure joins `planparser.PlanDirRel()` with the bare file name, which fits both card 3's read contract and card 5's anchor-relative `relPath`.

**Fix:** State in step 2 that candidates come from listing `ReworkDir` in the working tree, and that each candidate's `record.json` is read at HEAD through `ReadCommitted(ReworkDirRel/round-<N>/record.json)`.

### [LOW] A crash after ClearRejection wedges the rework row

**Location:** `_lyx/plan/09-loomshed-pr-rework.md`

**Issue:** Step 6 deletes the pending record, and Shed persists the row's `Done` only afterwards.
If the process dies between the two, Shed re-calls the rework row, step 1 finds no record and returns `Stuck`, and the run blocks at `PR-Rework`.
`lyx loom start` repeats the `Stuck`, and `lyx loom reject` refuses because the rework commit has moved local HEAD past the PR head.
The run can then leave only through a hand edit.
D5 asks for crash re-entry to be completed idempotently, and this window is the last one it leaves open.

**Fix:** In step 1, an absent record while `ReworkDir` holds a round committed at HEAD re-runs `Rebaseline` and returns `Done`.
The main line then reaches the gate again, which awaits.
An absent record with no committed round stays `Stuck`.
Add that case to the test list.

### [LOW] Card 12's ImpactSummary restates Intent

**Location:** `_lyx/plan/12-landingshed-publish-split.md`

**Issue:** "Publish returns Done where it returned Awaiting and stops reading the approval record" restates the Intent.
It does not name who consumed the `Awaiting`: the landingshed Publish tests (`publish_test.go`, `publish_verify_test.go`, `publish_integration_test.go`, `publish_awaiting_test.go`), the status spec's awaiting comment, and in-flight runs awaiting at Publish.
Batten's awaiting watch is row-agnostic (`callAwaiting` keys on state, not producer), so the gate's `Awaiting` does not affect it.

**Fix:** Replace it with a blast-radius conclusion that names these consumers and says batten is unaffected.

### [NIT] Card 1's ImpactSummary miscounts WriteApproval's callers

**Location:** `_lyx/plan/01-landingshed-rejection-record.md`

**Issue:** `WriteApproval` has one production caller (`internal/loomcli/approve.go`) and test callers in `landingshed` and `battencli`, not "three callers".

**Fix:** Say one production caller plus test callers, all unchanged.

## Focus directive verdicts

- Card 15 reach: a defect (F1). The build breaks as written.
- Card 12 reach: `publish_verify_test.go` must change (F2). `resume_test.go` and `sequence_test.go` still hold, because Publish's `on_stuck` stays empty (F4's second half), provided F4's fixture fields are filled.
- Docs fidelity: the module-tree line, the approval sentence, the batten parenthetical and the status-spec comment are all missing (F5).
- Card 9 seams: they are sufficient, and only the enumeration source is unstated (F7). Step 4's read contract matches cards 3 and 5. Checking re-entry also turned up F8.
- Card 6: the seeded state is a mid-run resume, not a finished run (F6).
- ImpactSummaries: card 2 ("no existing path changes") and card 13 ("no other verb reads that check") each give a real conclusion. Card 12 restates Intent (F9).
