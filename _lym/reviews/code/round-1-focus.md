---
round: 1
focus:
  - "internal/dobbyengine/landnotice.go and loop.go: does a landing send exactly one notice per task, with the beam follow-up text (update the change description, run `lym beam describe <slug>`) carried on the `notify_on_land` line only for members of a beam with a recorded pull request, and no second line when the flag was also set by hand?"
  - "internal/beamcli/pr.go, dobbydeps.go and internal/boardengine/board.go (`SetNotifyOnLand`): does `lym beam pr open` flag every current member task, and is every member added while the PR is recorded flagged too (the add path, not just the open path)? Check failure paths: a member whose entry cannot be flagged, a PR recorded then cleared, an already-flagged member."
  - "internal/dobbyengine/announce.go and announce_test.go deleted: is the beam-PR landing notice gone as a separate trigger, with no remaining caller, doc reference, geometry field (`geometry.go`, `hubgeom/dobbygeom.go`) or contract text (`contracts/orchrole/role-beams.md`, package `doc.go` files) still describing two mechanisms?"
  - "internal/pawlengine/landing.go, retiredmove.go and the loomcli landing-record seams: does the landing record still carry what the retired move and drop paths need now that the notice trigger moved, and is the scratch pull request carried into the record unchanged?"
  - "Test economy: the deleted dobby and announce tests against the new `TestLandNotices`, `TestNotifyOnLand` and `TestHubDobbyDeps_NotifyOnLandSeamsWriteTheHubBoardEntry`; is any behavior the diff adds (flag on member add, one-line dedupe) left without coverage, and does any new test duplicate another's coverage?"
---

Focus covers the issue's two asks: one trigger (the flag) and the beam follow-up riding on its line.
The decision record's sections after `## Goal` are empty, so the board entry and its 2026-11-09 refinement are the measuring stick.
