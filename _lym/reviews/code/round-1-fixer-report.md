---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed-differently
    reason: "Took the review's second option, a walk of the live worktrees by the beam reinterpretation. It writes through configengine.Set rather than Seed-Child's writer, since that writer replaces a module's whole local file and the carry must keep every key the worktree already sets. The hub reconcile runs it once, with the new Reinterpretation.RunsWithoutFile flag so a hub with no beam.yaml still gets the carry. The test pins the carry into the local layer and that a key the worktree already sets is kept; it does not run a landing."
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
  - id: F13
    disposition: fixed
  - id: F14
    disposition: fixed
  - id: F15
    disposition: fixed
  - id: F16
    disposition: fixed
  - id: F17
    disposition: fixed
  - id: F18
    disposition: fixed
  - id: F19
    disposition: fixed
  - id: F20
    disposition: fixed
  - id: F21
    disposition: fixed
  - id: F22
    disposition: fixed
  - id: F23
    disposition: fixed
  - id: F24
    disposition: fixed
  - id: F25
    disposition: fixed
---

# Fixer-report, round 1, code review

F12 is a `decision`-class finding that went to the parent, so this report leaves it out and its code is unchanged.
`lym gate test --upto 1` over the touched packages passes at tier 0 and tier 1.

## What changed

- **F1.** `ReinterpretPerTaskSettings` no longer claims the required-to-on rewrite keeps `required`'s meaning.
  It flags each rewritten `required`, in the hub file and in any worktree file, with what `on` no longer does and the way forward.
  The way forward names GitHub branch rules and the `--layer default` setting of `merge`, and says that `merge` squash-merges at once with no review.
  `retiredMeaning` now says the rewrite only lets beam.yaml load.
  Test rows in `TestReinterpretPerTaskSettings` assert the flag.
- **F2.** The new leaf package `internal/beamsetting` declares the beam module name, the two keys and the values once.
  `dobbyengine/config.go` aliases them, `landingshed/pullrequestmode.go` and `beamcli/pr.go` and `land.go` drop their copies,
  and `boardengine/store.go` takes its hint literals from it.
  `landingshed` and `dobbyengine` each gained the needed entries in their import allowlist tests.
  The overview label `dobbyLandNotice` in `battencli` now takes its name from the key.
- **F3.** `ownTaskSetting` in `dobbyengine/tasksetting.go` is the one reader of a place's own value.
  The lookup and the reinterpretation's flag step both call it.
- **F4.** `TaskSettingChain` builds each worktree's `Location` through `lymcwd.NamedWorktree`.
- **F5.** The beam reinterpretation carries the board's retired keys into live worktrees.
  `boardengine.LegacyBeamSettings` reads the retired keys of board.json as they are on disk.
  The reinterpretation writes each setting into the local layer of the worktree named after the entry, unless that worktree's own files set the key.
  Tests: `TestReinterpretPerTaskSettings_HubWorktrees` and `TestLegacyBeamSettings`.
- **F6.** The reinterpretation reads the own beam.yaml files of each worktree beside the prime, skipping the Bolt checkout.
  It flags each retired value with its path and `lym config beam --set` in that worktree.
  One more line says that a worktree file's `pull_request` now decides that worktree's own landing and is inherited below.
- **F7.** The beam module registers `dobbyengine.CheckLocalValues`, so upsert refuses a bad `pull_request` or `land_notice`.
  The allowed values come from `beamsetting`.
  Rows in `TestPrepareRunSettings` pin a refused `required`, a refused `per-task` and a pass.
- **F8.** `LandNoticeState` records whether `land_notice` was read.
  A remembered run past Land that was never read is read again each tick, and its notice stays undecided until a read succeeds.
  A memory an older binary wrote with a request counts as read.
- **F9.** A legacy `pull_request` of `off` is dropped by the fold.
  The reconcile flags each such entry with the `lym config beam --set pull_request=off` that keeps its old effect.
- **F10.** `ParseBoardTasks` folds the legacy keys as `Load` does.
  Two `TestFabricReplay` rows pin a commit that sets only `notify_on_land` and the removal of an unchanged entry that carries a legacy `pull_request`.
- **F11.** `TestLookup_BeamCarriesThePerTaskSettingsReinterpretation` pins the registration.
- **F13.** A beam project's `notify_on_land` is carried into its worktree's local layer by the same reconcile step.
  The `foldLegacyBeamFields` comment, the `Load` comment and the `boardengine` doc now say what happens to a project's key.
- **F14.** A prime file that does not read is a flagged line, and the hub file is still rewritten.
  A test row has an invalid prime local file.
- **F15.** `LandNoticeState.ClearOwed` decodes the old memory read only, and `rememberRuns` forgets such a run without a line.
  A test row seeds that state.
- **F16, F17, F18.** The board skill, the overview and `PATTERN-config-layers.md` carry the replacement text.
- **F19.** `TestTaskSettingChain` gained a parent-cycle row, and `TestTaskSettingChain_Errors` pins the errors naming the slug for an unusable anchor and an unreadable board.
- **F20.** `MergeWithTheSameMissingFactRefused` asserts "the beam's pull_request is merge".
- **F21.** The three `//testtiming:keep` lines are in place.
- **F22.** `beamModeUnreadableReason` names the worktree command, the `--layer default` command for the hub's default file, and says a `run_settings` change applies from the next run.
  The reason assertions of `TestFinalize_PullRequest` and `TestPRReview_Call` follow, and `checkTaskSetting`'s way forward says "for its next run".
- **F23.** `lookupTaskSetting` expands an envsource marker in the winning value through `envsource.Build` and `yamlengine.Resolve`.
  `TestLookupTaskSettings_ExpandsAnEnvMarkerAsLoadConfigDoes` pins a resolved and a refused marker.
- **F24.** `_lym/landing/summary.md` carries the widened blast radius, the `run_settings` note for an older binary, and the updated open risks.
  The file lives under `_lym`, so it is edited and not committed.
- **F25.** `parentPullRequestMode` is `pullRequestModeSeam`, and the test fields are `mode` and `modeErr`.

## Gate repair

The comment-format gate refused the round before any test ran.
It named 17 comment sites that wrapped mid-sentence or reached 120 characters:
in `beamsetting.go`, `boardengine/doc.go` and `store.go`, `configreg_test.go`, `dobbyengine/config.go`, `landnotice.go`,
`reinterpret.go`, `reinterpret_test.go`, `seam_enforcement_test.go` and `tasksetting.go`, and `hubreconcile/reinterpret.go`.
A second gate run named one line my first repair left at 120 characters, the `LegacyBeamSettings` comment in `boardengine/store.go`, and it is split too.
Each is now one sentence per line under 120 characters, with no change of meaning.
`lym gate test --upto 1` over the touched packages passes again at tier 0 and tier 1.

## Disputed

- A fork finding on `CheckLocalValues` refusing an envsource marker that the lookup now expands.
  Ground `every fix worse`.
  `Module.CheckValues` takes the document bytes alone (`internal/configreg/configreg.go:77`), so expanding there needs an environment directory the upsert does not have.
  A run_settings value is data the orch writes, so refusing a marker there is the intended outcome.
- A fork finding that the sibling scan applies the prime's anchor offset to every worktree instead of its recorded anchor.
  Ground `every fix worse`.
  Reading a recorded anchor needs `internal/lymcwd`, which `internal/dobbyengine/seam_enforcement_test.go` denies.
  The offset equals the hub-wide fallback, and the code names this.
- A fork finding that a run never read before Land is read past Land through a worktree that may be gone.
  Ground `every fix worse`.
  The alternatives drop the run's notice, which is F8's premise, and the read still happens only for runs whose earlier reads all failed.

## Deferred with reason

Nothing deferred.

## Rulings

No ruling reached this round.

## Self-check

All four forks returned a report, saved beside this file as `round-1-fixer-report.lens-<lens>.md`.

- fix-code-correctness, MEDIUM: the Bolt checkout read as a worktree. Fixed; the test creates a Bolt checkout with a hub beam.yaml.
- fix-code-correctness, LOW: a read failure in the carry loop is silent. Fixed, flagged with its way forward.
- fix-code-correctness, LOW: `CheckLocalValues` and the marker expansion disagree. Disputed, every fix worse.
- fix-code-correctness, LOW: a never-read run past Land reads through a possibly torn-down worktree. Disputed, every fix worse.
- fix-consistency, MEDIUM: the landing-notice file header was stale. Fixed.
- fix-consistency, MEDIUM: `checkTaskSetting`'s way forward offered `run_settings` as an immediate fix. Fixed, "for its next run".
- fix-consistency, MEDIUM: the Told-Geometry invariant text. Fixed differently: the invariant's comment names the one exception, since passing more geometry changes the `Rewrite` signature of every reinterpretation.
- fix-consistency, LOW: the `Rewrite` doc ignored `RunsWithoutFile`. Fixed.
- fix-consistency, LOW: the `Load` doc said every retired key is folded. Fixed.
- fix-proposal, MEDIUM: the `merge` way forward drops review. Fixed, the flag says so.
- fix-proposal, MEDIUM: a dropped `off` changes a task's landing with no notice. Fixed, the reconcile flags each one.
- fix-proposal, MEDIUM: a memory holding a request but no `read` is read again past Land. Fixed, a request counts as read.
- fix-proposal, LOW: marker accepted in a worktree file and refused in run_settings. Disputed, the same finding as above.
- fix-proposal, LOW: the sibling scan approximates anchors, and scans the Bolt checkout. The Bolt part is fixed, the anchor part disputed.
- fix-affected-set, LOW: the `dobbyLandNotice` literal in `battencli`. Fixed.
