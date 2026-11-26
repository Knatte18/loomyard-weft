# Card 4 — run-watch-wiring

**Edit:**
- `internal/beamcli#hubDobbyDepsStarting`
- `internal/beamcli#TestHubDobbyDeps_RunNoticePostCarriesTheRunsAutonomy`
- `internal/beamcli#TestHubDobbyDeps_LandNoticeReadsTheRunsWorktreeLayer`
- `contracts/orchrole/orch-role.md`

**Uses:**
- `internal/dobbyengine#RunWatchDeps`
- `internal/dobbyengine#LookupRunWatch`
- `internal/beamcli/doc.go`

**Intent:**
Wire the run watch of card 3 into the hub's dobby, and name its lines to the orch (D5, D7).

`hubDobbyDepsStarting` sets `RunWatch` on the `Deps` it builds:
`Level` builds the chain with `hubgeom.TaskSettingChain(prime, slug)` and returns `dobbyengine.LookupRunWatch(chain)`, as `LandNotice` does;
`Post` returns `orchcli.NotifyRunFrom(worktree, geom.Slug, dobbyProgram, creelengine.PostTags{}, orchcli.WithRunAutonomy(worktree, slug, line))` unwrapped, with no topic and no run tag, so no watch line replaces another unread one and a run's teardown drops none.
It does not go through `requirePosted`, since the watch tells a missing orch strand from a failed post by `posted`.

`contracts/orchrole/orch-role.md`'s Notices section opens with the rule that a mail whose text starts with `[batten notice]` or `[dobby notice]` is a notice; that rule names `[run watch]` beside the two prefixes.
The section also names `[run watch]` lines, from `<shortname>:DOBBY` or `<shortname>:<slug>:DOBBY`, among the notices: a stop line is a stop and is handled under Investigating a stop; a later `[batten notice]` or driver line for the same stop adds its detail and opens no second fork; a `rows` progress line asks for nothing.

Tests:
`TestHubDobbyDeps_RunNoticePostCarriesTheRunsAutonomy` also posts one line through `deps.RunWatch.Post` for `task1` from each dobby and wants it among the posted mails with the same sender and autonomy stamp, after the row's run notice; it asserts `posted` is true.
It asserts the watch line's mail carries an empty topic and an empty run tag, so neither a newer post nor a run's teardown can drop it.
It then asserts that no log entry is `undelivered` with reason `creelengine.SupersededReason`, so the watch lines and the run notices all stand unread.
The test's doc comment and its file comment, `dobbynoticestamp_integration_test.go`'s, name the watch line beside the run notice.
`TestHubDobbyDeps_LandNoticeReadsTheRunsWorktreeLayer` also reads `deps.RunWatch.Level("b")` as `stops` with no layer setting it, and as `rows` once the beam worktree's beam.yaml sets `run_watch: rows`.

**ImpactSummary:** One new seam pair in the hub's dobby wiring and one paragraph in the orch role; existing seams and notices are untouched.

**Commit:** 4: Post the run watch's lines from the hub's dobby and name them to the orch
