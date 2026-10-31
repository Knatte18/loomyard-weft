---
verdict: APPROVED
---

No findings.

The diff over `82c5a802..HEAD` makes the change the board entry asks for, as amended by the Option B entry in the decision record.
Each focus site was read in the code and settled as follows.

- `internal/dobbyengine/land.go` `landBeam`.
  A refusal from `deps.Land` is posted once and recorded with `Refused: true`.
  The next tick sees that record, asks `Land` again, and posts nothing while `record.Land != nil`.
  A success records the episode closed through `recordLand(false)`.
  A reading that stops passing clears it.
  A refusal notice that fails to post leaves `record.Land` nil, so the next tick retries the post.
  `TestLandBeam` pins: the refusal posted once, the retry until success with a single notice, the cleared episode, the restart, and the failed post.
- `internal/beamcli/dobbydeps.go` `beamLandReading` and `deps.Land`.
  `beamLandReading` returns an empty reading when the ready read errors or the beam is unflagged, before `ReadBeamLandBlockers`.
  `deps.Land` runs `land` itself, the sequence `lym beam land` runs, so it writes the same request and returns the same refusals.
  `TestHubDobbyDeps_LandsOnlyAFlaggedBeam` reaches both over a hubforge hub: an unflagged beam reads as not passing and `Land` refuses naming `lym beam ready b` with no request written, and a flagged beam passes and `Land` writes the request.
- `internal/beamcli/deps.go` `MarkReady`.
  A flagged beam skips the write and still commits the landing directory, so rerunning after a failed commit commits the record already written instead of reporting `already` with the flag untracked.
  `TestHubLandDeps_MarkReadyCommitsAnUncommittedFlaggedRecord` pins the commit and that the original `SetAt` is kept, and `TestMarkReady` pins the seam-level rows.
- `internal/beamcli/land_test.go` and `internal/loomcli/pawlwiring.go`.
  The `TestLand` rows pin the ready refusal after the owner refusal and before the work-remaining refusal, and the `Ready` error way forward.
  The `TestMarkReady` rows pin the `Ready` and `MarkReady` error ways forward.
  `TestPawlRecheckLines` pins the unflagged line last, after the work, beam-owner and parent-owner lines.
- `internal/dobbyengine/loop.go` `tick` and the `internal/loomcli/doc.go` re-check paragraph.
  Both match the landing step and the ready record the re-check reads.

The gate's `lym gate test` over `internal/dobbyengine`, `internal/beamcli`, `internal/loomcli` and `internal/pawlengine` carried a green record, so no tier re-ran.
No `Pattern-Departure:` trailer is present and none is needed.
`PATTERN.md` and `pattern/` are untouched and the decision record declares no PATTERN change.
