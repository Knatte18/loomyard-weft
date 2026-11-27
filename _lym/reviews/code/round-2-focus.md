---
round: 2
focus:
  - "internal/battencli/cancel.go, refuseBeamCancel and its callers: which members and forked pairs refuse a beam cancel with and without --remove, in both the held and unheld branches, and does the outcome match the board entry's plain project with its members untouched? Check against how internal/boardengine/parentbeam.go and internal/battencli/beamland.go read a member once the beam flag is cleared, and whether the help text, the package doc and _lym/landing/summary.md describe what the code does."
  - "internal/boardengine/beamend.go, Board.EndBeam: it clears a beam's recipe, owner_machine and beam_color and its flag. Is each write correct and safe across machines, and do the owner-writer statements in internal/boardengine/doc.go and elsewhere agree with the code?"
  - "internal/battencli/cancel_test.go and cancel_integration_test.go: do the EndBeam and BeamProject seams of cancelFake conform to PATTERN-test-stand-ins, and does a test over a real hub drive the production hubCancelDeps wiring, the beam_ended envelope key and the board entry's resulting state?"
  - "internal/battencli/cancelcmd.go, the production StopBeam seam: is its skip on an absent or unmarked origin record covered by a test that would fail if the guard were removed?"
  - "internal/battencli/cancel.go, clearRunPhase: follow each failure between RemoveRecords, EndBeam, ClearClaim and PushRecords, and the re-run after it. Is any committed records change left unpushed, or any board state left that the re-run cannot finish?"
  - "_lym/landing/summary.md and the lym batten cancel help in internal/battencli/cancelcmd.go: does the merge-danger door and blast radius cover every board field erased and pushed, and does the help still carry every statement a user needs, including the emitted envelope keys?"
---
