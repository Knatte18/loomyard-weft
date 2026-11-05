# Card 16 — member-landing-notice

**Create:**
- `internal/dobbyengine#AnnounceFileName`
- `internal/dobbyengine#daemon.announceLandings`
- `internal/dobbyengine#TestAnnounceLandings`

**Edit:**
- `internal/dobbyengine#Deps`
- `internal/dobbyengine#Geometry`
- `internal/dobbyengine#daemon.tick`
- `internal/dobbyengine/doc.go`
- `internal/hubgeom#DobbyGeometry`
- `internal/hubgeom#TestDobbyGeometry`
- `internal/hubgeom/doc.go`
- `internal/beamcli#hubDobbyDeps`
- `internal/beamcli/doc.go`

**Uses:**
- `internal/pawlengine#ReadLanding`
- `internal/loomengine#BeamLandingPath`
- `internal/loomengine#BeamLandingLockPath`
- `internal/hubgeom#DobbyBeamDir`
- `internal/fabricengine#BeamLineCommits`

**Intent:** While the landing record holds a pull request, the beam's dobby tells the beam's orch about each landing into the beam branch (D7), so the orch keeps the description and the pull request listing every member.

- `dobbyengine.Geometry` gains `AnnouncePath`, the never-tracked file under the beam worktree's `.lym/beam/` (`hubgeom.DobbyGeometry` joins `AnnounceFileName` onto `DobbyBeamDir`) holding the last beam head the dobby announced (`PATTERN-durable-vs-ephemeral-state`).
- `Deps` gains `LandingPullRequest func() (announceHead string, recorded bool, err error)`, the landing record's pull request's announce head, and `BeamLandingsSince func(from string) (head string, landed []string, err error)`, the beam branch's head and the landings on its first-parent line past from, oldest first, each as its short object name and subject; both nil on the prime.
- `daemon.announceLandings`, called by `daemon.tick` on a beam's dobby after the landing step: with no pull request recorded it does nothing. Otherwise it reads the last announced head from `AnnouncePath`, or the record's announce head when the file is missing, so a fresh machine repeats at most one notice; reads the landings past it; with none posts nothing; with some posts one line through `Deps.Notice` naming them and `lym beam describe <slug>` then `lym beam pr update <slug>`, and only after the post succeeds writes the head to `AnnouncePath`. Every landing counts, a member filed after the pull request opened included, and no entry's `notify_on_land` is read. A failed read or post is logged once per episode and retried next tick; nothing else depends on it.
- `beamcli.hubDobbyDeps` fills both seams for a beam: `pawlengine.ReadLanding` at `loomengine.BeamLandingPath`, and `fabricengine.BeamLineCommits` (card 12) over the beam pair, keeping the commits after from and dropping every records-only one, so the run's status commits and the commits of `lym beam describe` and `lym beam pr` themselves are never announced and the notice cannot feed itself; a from not on the line announces the whole line once. The head it returns is the pair's HEAD, so a records-only commit moves the announced head on without a notice.
- Bound: the notice only posts to the beam's own orch; a member that also carries `notify_on_land` gets that notice too, from `landNotices`, unchanged.

Tests (tier 0): `TestAnnounceLandings` drives the daemon through `newLoopHarness`, its `geom` set to a beam's with an `AnnouncePath` after the harness builds it, with fake seams: a recorded pull request and one landing past the announce head posts one notice naming it and both verbs and writes the head; a later tick with no new commit posts none; a second landing posts one more naming only it; a new head past only records-only commits, as the fake seam reports it, posts nothing and writes the head; a missing announced-head file announces from the record's announce head; no pull request recorded posts nothing and writes nothing; a failed post writes nothing and the next tick posts again; a failing `LandingPullRequest` or `BeamLandingsSince` read posts and writes nothing. `TestDobbyGeometry` gains `AnnouncePath` under the beam's `.lym/beam/`. The records-only filter itself is pinned by card 12's `TestBeamLineCommits`.

**ImpactSummary:** one new dobby step with two optional seams filled only on a beam's dobby, and one new geometry path; the prime's dobby and the landing notice are unchanged.

**Commit:** 16: The beam's dobby announces member landings while a pull request is open
