```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, covering what the diff's changes force outside the lines they touch.

**Per change, the set found:**

1. **`AnnounceHead` removed from `LandingPullRequest`.**
   - Covered: no `AnnounceHead` reference is left outside `_lym` and `.lym`. The producers (`beamcli/pr.go`, `pawlengine/retiredmove.go`) and the tests (`pr_test.go`, `landing_test.go`, `retiredmove_test.go`, `pawlwiring_integration_test.go`) are updated.
   - `beamcli/doc.go:166` is updated. `ReadLanding` decodes through `state.ReadJSON`, which is tolerant, so an old `announce_head` key is not refused.

2. **`announce.go`, `AnnouncePath`, `AnnounceFileName` and the `LandingPullRequest`/`BeamLandingsSince` seams removed.**
   - Covered: a repo-wide grep finds no `BeamLandingsSince`, `landingsPast`, `AnnouncePath`, `AnnounceFileName` or `announced-head` outside `_lym`, `.lym` and the research dumps.
   - `beamLandingsSince` and `landingsPast` are gone from `beamcli`, with the `fabricengine` import dropped from `dobby_test.go`. The `announceRead` and `announcePost` episodes are gone from `loop.go`.
   - `fabricengine.BeamLineCommits` is still used by `beamcli` (`doc.go:140`) and `fabricengine/beamline.go:104`, so nothing is orphaned.
   - `hubgeom/dobbygeom.go`, `dobbygeom_test.go` and `hubgeom/doc.go` are updated.
   - The `dobbyengine/doc.go` "member landing announcement" section is removed and replaced inside "The landing notice". The `beamcli/doc.go:454-458` seam list is updated.
   - The `contracts/orchrole/role-beams.md:50` stencil wording is updated and matches the amended entry.

3. **New `PullRequestRecorded` and `SetNotifyOnLand` seams in `Deps`, wired in `hubDobbyDeps`.**
   - Covered: `loop.go` carries the field docs, `beamcli/doc.go` the seam list, and `boardengine/doc.go:411` the `SetNotifyOnLand` paragraph.
   - `landNotices` runs at `loop.go:669`, before `startReady` at `loop.go:671`. That matches the "flagged in the tick that starts it" claim in `dobbyengine/doc.go`.
   - `TaskView.Project` and `Kind` are "read for every entry" (`starts.go:41`), so `flagMembers` sees the data it needs.

4. **`landNoticeLine`, `rememberRuns` and `takeDoneEntries` signatures changed.**
   - Covered: there is no other caller outside `landnotice.go`. The grep found no other definition or use.

5. **Repo rule documents and PATTERN entries.**
   - No PATTERN change is declared in this round's directive, and none is in the diff.
   - I found no `pattern/` entry the diff contradicts. `PATTERN-dobby-runs` constrains starting runs and `front`/`on_land` writes, not `notify_on_land`. `PATTERN-tolerant-records` is met by the tolerant read.
   - Commit messages carry no `Pattern-Departure:` trailer, and none is needed.
   - Prose that still describes `notify_on_land` as set only by hand, but is not wrong: `boardcli/cli.go:588`, `plugins/ly/skills/board/SKILL.md:237`, `contracts/orchrole/orch-role.md:235` and `docs/overview.md:1659`. None claims the flag is hand-set only.

**Findings:** none.

**Tool calls:**
1. A grep for stale announcement, landing-notice and `notify_on_land` mentions across code and docs. My context lacked a repo-wide sweep of non-test files.
2. A grep for `beam describe` with notice wording in docs and code, plus the old `dobbyengine/doc.go` announce lines and the new landing-notice doc section. This was to check that no other doc still describes two mechanisms.
