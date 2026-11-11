# One land notice: fold the beam-PR landing notice into notify_on_land

A beam member's landing used to send the beam orch two mails: the beam-PR landing notice, sent for every landing while the beam's pull request is recorded, and the `notify_on_land` line.
Both reported the same event, and it was unclear which did what.
Now `notify_on_land` is the only trigger for a land notice, and the one line carries the beam's follow-up.

Closes #93

## What changes

- While a beam's pull request is recorded, the beam's dobby sets `notify_on_land` on each open member of the beam.
  A member is flagged only while its run is listed ahead of its Land row, so a flag cleared by its landed line is not set again.
- A member's landed line then ends with `; add it to the change description, then run `lym beam describe <slug>`, which updates the pull request`.
  A task flagged by hand gets the same single line, not a second one.
- The separate beam-PR landing notice is removed from `dobbyengine`: `announce.go`, the `AnnouncePath` and `AnnounceFileName` geometry, the `LandingPullRequest` and `BeamLandingsSince` seams, and `beamLandingsSince` in `beamcli`.
- `LandingPullRequest.AnnounceHead` is dropped from the landing record in `pawlengine`; `lym beam pr open` and `MoveOffRetired` no longer write it.
- The orch role contract in `contracts/orchrole/role-beams.md` describes the single notice.

## Design decisions

- The flag is set by the beam's dobby from the recorded pull request, not by `lym beam pr open`.
  This also covers members added while the pull request is recorded, and needs no hook at open time.
  The seams are `Deps.PullRequestRecorded` and `Deps.SetNotifyOnLand`, backed by the new `boardengine.Board.SetNotifyOnLand`.
- `LandNoticeState` remembers whether the run's entry was a member (`Member`), so under `per-task` a remembered member still gets its line with the follow-up when its entry was never flagged.
- A failed read of the landing record decides no notice that tick and is logged once per episode; a failed flag write is retried at the next tick.
- Under `land_notice: all`, the follow-up rides on every member's line in the same way.

## To check by hand

- `PullRequestRecorded` in `internal/beamcli/dobbydeps.go` reads the beam's landing record; the integration test in `dobbydeps_integration_test.go` pins it, but check the seam against a real hub beam with a recorded pull request.
- Re-flagging after a landed line has cleared the flag: `flagMembers` in `internal/dobbyengine/landnotice.go` skips done entries and runs not listed ahead of Land, so check that no member gets a second line after landing.
- Landing record files written before this change still hold `announce_head`; check that they read without error.
