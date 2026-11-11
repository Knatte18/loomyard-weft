# One land notice: a beam's pull request sets notify_on_land on members

A member landing into a beam with a recorded pull request used to send the beam's orch two mails:
the beam-PR landing announcement, which named the landing and asked for `lym beam describe`,
and, when the member carried `notify_on_land`, the dobby's landing line.
Now `notify_on_land` is the only land notice, and the beam's follow-up rides on its line.

## What changes

- **The beam's dobby flags its members.**
  While the beam's landing record holds a pull request, each tick of the beam's dobby sets `notify_on_land` on every open member that lacks it.
  A member is a task or project entry whose `project` is the beam's slug, and an open one is not done.
  The step runs in the landing-notice step, before the start step, so a member filed while the pull request is recorded is flagged in the tick that starts it.
  `lym beam pr open` records the pull request, and the next tick flags every open member.
- **The landed line carries the follow-up.**
  While the pull request is recorded, a landed line ends with
  `; add it to the change description, then run `lym beam describe <slug>`, which updates the pull request`.
  A member whose `notify_on_land` was set by hand gets that same one line.
  An "ended without landing" line carries no follow-up.
- **The separate announcement is gone.**
  `internal/dobbyengine/announce.go` and its test are removed,
  as are the `announced-head` file (`AnnounceFileName`, `Geometry.AnnouncePath`), the `LandingPullRequest` and `BeamLandingsSince` seams with their beamcli wiring (`beamLandingsSince`, `landingsPast`),
  and `LandingPullRequest.AnnounceHead` in the pawl landing record.
  The landing record reads as before: an existing `announce_head` key is ignored on read.
- **New seams.**
  `dobbyengine.Deps` gains `PullRequestRecorded`, which reads the landing record through `pawlengine.ReadLanding`, and `SetNotifyOnLand`.
  Both are wired for a beam's dobby only.
  `boardengine.Board.SetNotifyOnLand` sets the field under the write lock and writes nothing when the entry is absent or already carries it.
- The orch's `role-beams.md` now tells the orch that a member's landing notice carries the follow-up, in place of the old "dobby notice naming landings" line.
  The `dobbyengine`, `beamcli`, `boardengine` and `hubgeom` package docs follow the change.

## Design decisions

- **The dobby sets the flag; `lym beam pr open` does not write the board.**
  The operator's refinement has opening the pull request set `notify_on_land` on every member, and on every member added while it is recorded.
  One setter, the beam's dobby reading the landing record each tick, covers both cases and every route by which a member arrives (upsert, a project change, a move).
  It reaches each member before the dobby starts it.
  The trade-off: members open when the pull request opens are flagged at the dobby's next tick, not inside the verb.
- **`land_notice: off` stays authoritative.**
  Under `off` no member gets a line, so the follow-up goes too, where the old announcement went out whatever `land_notice` said.
  `per-task`, the template default, and `all` both send the member's line.
- **A failed landing-record read decides no line that tick**, as a failed `beam.yaml` read already did.
  This keeps a line from going out without the follow-up it should have carried.
- Only open members are flagged.
  Flagging a done member would bring back a line its landing already had, or one for a landing that happened before the pull request.

## Tests

- `TestLandNotices` gains rows for a beam with a recorded pull request:
  a member is flagged and its landed line carries the follow-up;
  a member flagged by hand gets the same one line and no second flag;
  a beam with no pull request flags nothing;
  an unlanded member's line carries no follow-up;
  and a failed landing-record read decides nothing until it reads.
- `TestNotifyOnLand` (boardengine) gains `SetNotifyOnLand` rows: it sets the field, and writes nothing for an entry already carrying it or an absent slug.
- `TestHubDobbyDeps_NotifyOnLandSeamsWriteTheHubBoardEntry` drives the beam dobby's set and clear seams against a hub board.
- `TestAnnounceLandings` and `TestLandingsPast` are removed with the code they pinned, and the `AnnounceHead` assertions are dropped from the pawl, beam PR and loom wiring tests.
