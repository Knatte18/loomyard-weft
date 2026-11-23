---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed-differently
    reason: "The code comment, the doc.go lines and the stop-hold premise are corrected as proposed. The summary.md Design decisions edit is left undone: the file is under _lym, outside this fixer's write surface, so the landing writer must replace its narrower claim (see Deferred)."
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed-differently
    reason: "Did not add a synchronous records push after fabricSync. The pattern already holds an owed-push rule: a failed detached push is recorded as owed and holds the run at its transition (PATTERN-run-fsm.md, An owed push holds the run; internal/fabricengine/failedpush.go). The pattern now names reset as the one transition writer whose code push goes first, and says that guarantee covers the window until its records push lands."
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed-differently
    reason: "The proposed row made 00-overview.md unreadable through permkit.Chmod and went through the gate, but the gate's plan parse reads the overview first and returns its own error, so that row would pass without reaching overviewFrameEditedFindings; Chmod also does not bind a root run. The test calls overviewFrameEditedFindings directly with a directory in the overview's place, which fails the read with an error other than not-exist."
  - id: F11
    disposition: fixed-differently
    reason: "The subtest sits in internal/fabricengine/weftgit_unborn_warp_test.go, beside the unborn-warp fixture helper, not in commit_integration_test.go, and reaches commitWeftLocked through a new CommitWeftWithTrailersForTest export, since CommitWithTrailers needs a resolvable warp worktree layout the unborn fixture lacks."
  - id: F12
    disposition: fixed
---

# Burler fixer-report, code round 1

## What changed

Commits, one per group:

- `fabricengine: fix F1, F11` — `commitWeftLocked` forces an empty weft commit when trailers are given and the warp is born, so a reset whose `state.json` bytes do not change still lands its `Run-Transition` commit.
  `CommitWithTrailers`' doc comment and `commitWeftLocked`'s doc state it.
  `TestResetCmd_RecordsTheRunTransitionAtTheNewTip` is now a table of a `batch-start` row and a `start` row.
  It commits the seeded `state.json` to the records side first, and the `batch-start` row requires an empty records commit.
  With the force line reverted, that row fails (`reset_test.go:203` to `:210` report the seed commit as HEAD); with it restored, it passes.
  `TestCommitWithTrailers` gains a subtest for unchanged weft files, and `TestCommitWeft_UnbornWarpHEAD_WritesTheCallersTrailers` covers the unborn warp.
- `fabricengine: fix F1` (follow-up) — the package doc's `CommitWithTrailers` paragraph states the empty commit.
- `sheddriver: fix F2` — the `driveNotices` comment and `doc.go` say batten still sends its own notice once the run parks, so the parent hears both.
- `sheddriver: fix F3` — `goDriver` records `judgedKey` beside `judged`, and `handBack` sets `Judged` only when the stop in hand has that key.
  A `TestGoDriver_Settle` row hands back by rule at a second blocked stop after the arbiter judged the first, watched, and wants no notice.
- `pattern: fix F5` (two commits) — `PATTERN-run-fsm.md` names `webster reset` as a transition writer outside `statuscommit` whose code push goes first, and what holds the run until its records push lands.
- `contracts: fix F6` (two commits) — `loom-plan-checks-spec.md` lists `overview-frame-edited`.
- `loomshed: fix F7, F9, F10` — a Card-Index-only row in `TestNewPlanGate`, a test that an unreadable overview is a returned error, and a doc line leaving a record with no frame to the rebaseline.
- `webstercli: fix F8` — `TestWire_ModeHubSelectsHubMode` asserts `runID` equals the worktree name.
- `sheddriver: fix F12` — `TestDriveNotices` carries its `testtiming:keep` mark.

Verify: `lym gate test` over `internal/fabricengine`, `internal/webstercli`, `internal/sheddriver` and `internal/loomshed` passed at tier 0, and over `internal/fabricengine` and `internal/webstercli` at tier 1, before the self-check's doc-only follow-ups.

- `fabricengine, loomshed, sheddriver: fix gate findings` — five comments broke at sentence boundaries, one sentence per line, with none reaching 120 characters.
  No code changed.
- `fabricengine: fix gate reach findings` — a tier 0 test, `TestCommitWithTrailers_RefusesAnInvalidTrailerBeforeTouchingAnything`, reaches `Fabric.CommitWithTrailers` over a zero Fabric.
  `Fabric.commit` stays unreached at tier 0, since it needs real git; the tier 1 `TestCommitWithTrailers` covers it.

## Self-check

- fix-code-correctness, LOW: a reset over a dirty records index now fails in its closing sync where it used to pass as a no-op — disputed, every fix worse: the refusal is loud and its way forward is a retry, where the alternative is the stale transition F1 removes.
- fix-code-correctness, LOW: `handBackFailure` at a stop other than the judged one sends no run-id line — disputed, premise false: that is F3's intended narrowing.
- fix-consistency, MEDIUM: the pattern paragraph says the code push goes first against its own rule line — fixed, now named as the exception with its reason.
- fix-consistency, LOW: `fabricengine/doc.go` still described `CommitWithTrailers` without the empty commit — fixed.
- fix-consistency, LOW: the spec entry's "its work" wording — fixed.
- fix-consistency, LOW: `_lym/landing/summary.md` is stale — deferred (see below).
- fix-proposal, LOW: pattern ordering — merged with the consistency finding, fixed.
- fix-proposal, LOW: dirty records index — merged with the correctness finding, disputed.
- fix-proposal, LOW: spec wording — merged, fixed.
- fix-affected-set, LOW: `fabricengine/doc.go` — merged, fixed.

## Disputed

- Self-check LOW (dirty records index), ground every fix worse: a refusal that surfaces `gitrepo.ErrIndexNotEmpty` fails loudly after the reset, and the sync's own way forward covers the retry.
  Evidence: `internal/fabricengine/recordsgit.go` `commitWeftLocked`'s empty-commit path and `TestCommit_DirtyWeftIndex_UnchangedContentWithTags_SurfacesPartialCommitError` in `internal/fabricengine/commit_integration_test.go`.
- Self-check LOW (`handBackFailure` run-id line), ground premise false: a stop no agent judged was never meant to send that line.
  Evidence: `internal/sheddriver/drive.go` `handBack`.

## Deferred with reason

- F2, the landing summary: `_lym/landing/summary.md` Design decisions still says both notices go only while batten's hold ends without the arbiter's hand-on, and its Tests section lacks the new rows.
  The file is under `_lym`, which this fixer never touches, so the landing writer must replace that claim with: the driver's line and batten's notice both go at every watched blocked stop an agent judged.

## Rulings

- F4 was settled by the parent (creel #2566, decision record entry "Defect 3 covers webster reset only"): only `webster reset` commits the transition and `lym loom goto` stays unchanged.
  No code changed for it.
