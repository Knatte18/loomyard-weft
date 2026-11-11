```yaml
lens: fix-affected-set
outcome: reported
```

**Findings**

- LOW. `internal/adoptcli/steps.go:175` (`stepPause` now loops `inputs.Paused`). Every listed slug is restated as `AdoptPaused`, `Run = pausedRun` and `PauseStanding = "...lym tender resume X lifts it"`, including one whose entry an operator already lifted. The test `PausedSlugWhoseEntryTheOperatorLiftedIsNotPausedAgain` (`worker_test.go:127`) checks only `w.record.Live` and does not see this. This matches how `stepBeam` (`steps.go:274`) and `resumeSlug` (`steps.go:236`) already treat `Inputs.Paused`, so I read it as consistent. A reader might still notice the standing text claims a pause that no longer stands.

**Set searches, no gap found**

- F5 (all `OwnerCaller` sites, non-test): `beamcli/dobbydeps.go:501` (lift, pinned by the new F6 test), `pauserecord/hold.go:306` (`OwnerEntryExempt`), `pauserecord/record.go:242` (`OwnerHeld`, which drives the parent-lock release), `hibernateengine/wake.go:113`, `tendercli/deps.go:238`, and `adoptcli/pause.go:115,129` (the conversion). The new `pauserecord/doc.go` text names the exemption and the release, which are the two sites the review flagged. The lift and the wake were already covered by the existing sentence.
- F2 (sites that print or parse the held-back command): `beamcli/adoptsection.go` reads `record.WayForward` and needs no change. The `beamcli/adoptsection_test.go` rows build their own records. `adopt_integration_test.go:183-186` asserts the per-slug `WayForward` and the two flag names, not the run-level command, so the changed run-level command does not touch it. The one unit test that pinned the old text (`adopt_test.go:433`, two slugs) was updated. The other `wantErr` rows (447, 460, 848, 966) are single-subject, so their text is unchanged.
- F4 (consumers of `namedEntries` and `Subjects`): `namedEntries` is now called only by `validateRequest` (`worker.go:115`) and by the fresh path (`worker.go:222`). `stepCheck`, `refuseBeforeSteps` and the envelope all use `run.Slugs` or `record.Subjects`, which now agree on resume.
- F1 (consumers of `Inputs.Paused`): `steps.go:26` keeps `Paused` across a restart, and `steprun.go:169` seeds outcomes from it. Both are consistent with the restate loop.

**Calls made**
1. grep of non-test `OwnerCaller` sites across `internal`, to rerun F5's search; my context had only the sites named in the review.
2. grep for `held-back`, `HeldBack` and `--keep-local` outside the files I edited, to rerun F2's search; my context lacked the other tests and callers that name the command.
3. grep of `namedEntries` and `Inputs.Paused` consumers, to rerun F4's and F1's searches; my context lacked the full caller list.
