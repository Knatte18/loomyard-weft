# A beam member gets its landed line only when its land_notice is on

A beam member under a recorded pull request got its landing notice whatever its `beam.land_notice` read.
That override broke the per-task rule that a value set lower down always applies.
Now `land_notice` off means no line for every run, beam members included.
A line that does go out under a recorded pull request still ends with the beam's follow-up, `lym beam describe <slug>`.

## What changes

- `rememberRuns` in `internal/dobbyengine/landnotice.go` decides a line only from the run's remembered `land_notice` reading.
- Membership is no longer read for the landing notice.
  The board read in `landNotices`, `isBeamMember`, `landNoticeTasks`, the daemon's `landNoticeBoard` failure episode and `LandNoticeState.Member` are gone, since nothing else read them.
  A state record holding the old `member` key still loads; the key is ignored.
- The package doc, the `landnotice.go` header, the doc comments of `landNotices`, `rememberRuns`, `pullRequestFollowUp` and `Deps.PullRequestRecorded` drop the exception.
- `contracts/orchrole/role-beams.md` tells the orch that a member's notice under a recorded pull request comes only when its `land_notice` is on.
- `TestLandNotices`: a member whose lookup reads off under a recorded pull request now expects no line.
  Rows that only exercised membership (a failed board read, a non-member entry, a done member entry, an unrequested member with no pull request) are gone.
  The follow-up rows now set `land_notice` on.

## Design decisions

- The decision record's section 2 findings (F26, F2, F27, F28, F5, F13) were already fixed in the landed `118bee8ad`, so this change leaves them.
  The reinterpretation takes the hub's code worktrees from `hubreconcile` as told geometry, and the configreg registration and `beamcli` use `beamsetting.Module`.
  The registration test carries its `//testtiming:keep` marker, and `TestPrepareRunSettings` holds no duplicate row.
  The carry runs once per machine behind a marker beside the build stamp, reading run settings as well as the retired keys.
  A project's `notify_on_land` stays in `Unknown` across a save.
- `pattern/PATTERN-landing.md` and `pattern/PATTERN-config-layers.md` held no mention of the exception, so neither changes.
- An orch that wants the beam's pull request description kept current now depends on each member's `land_notice`; keeping it current without the orch is the board entry `beam-pr-describes-itself`.

## Merge danger

**Door:** two-way
**Blast radius:** dobby
