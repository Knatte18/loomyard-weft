# Fixer report — Plan-Review round 1

All three findings are fixed.
`lyx loom validate-plan` passes over the fixed plan (`ok: true`).
Its one remaining finding is informational and was there before this round: a `.Call` name match in `internal/proc/namedjob_windows.go` whose receiver the checker could not resolve.

## Changes

- **F1 (fixed as proposed).**
  The premise holds: `roundTable` sets `Segment: segmentcolor.Review` (`internal/burlerengine/engine.go:122`), and `seatengine.Table.Segment` (`internal/seatengine/table.go:91`) has no default.
  Card 2's `fixTable` sentence now lists "the review segment color (`segmentcolor.Review`)" after the role prefix.
  The planned shape assertions in `review_test.go` and `fix_test.go` now each include the review segment color.

- **F2 (fixed differently).**
  The premise holds: card 4 listed `internal/loomcli/smoke_attachprobe_test.go` and card 10 listed `internal/shedadapters/burler_test.go` as bare paths, while their Intents delete named members of those files.
  - Card 10: the `burler_test.go` path is replaced by `**Delete:**` glyphs for `TestNewBurlerProducer_Validation` and each of the 16 `TestBurlerProducer_*` functions in that file.
    The proposed `simpleBurlerProfile` Edit glyph was not added, because card 10 does not change it.
  - The proposed `**Delete:**` glyphs on card 4 were not added, because the plan gate refuses them.
    With `probingBurlerRunner.Run`, `.ProbeRound` and `.Resume` on card 4, `delete-before-reference` matched `.Run`, `.ProbeRound` and `.Resume` as text in the Edit code of cards 5 to 10 and blocked.
    With only the type and the test on card 4, `caller-uncovered` blocked on the method bodies at `smoke_attachprobe_test.go:100`, `:105` and `:121`.
    So `probingBurlerRunner`, its three methods and `TestSmokeBurlerRound_AttachesToALiveRoundInsteadOfRespawning` are now `**Delete:**` glyphs on card 10 instead.
    That matches Shared Decision `additive-then-retire`.
    The old smoke test still compiles after card 4, since it drives `BurlerProducer`, which lives until card 10.
    Card 4 drops the file path.
    Card 4's Intent now says the new smoke test lands beside the old one until card 10.
    Card 10's Intent now names the deletion.
    Card 4's ImpactSummary now says the old symbols are left referenced only by their own tests.
  - Dropping `burler_test.go` from card 10 left `var _ BurlerRunner = (*shedfake.BurlerRunner)(nil)` (`burler_test.go:22`) uncovered, and `caller-uncovered` blocked on it.
    A blank-identifier assertion has no glyph.
    `redundant-file-target` forbids putting the file back beside card 10's new Delete glyphs.
    So card 4, which already drops the old seam's compile-time assertions in `burler.go` under the same Shared Decision, now lists `internal/shedadapters/burler_test.go` and drops this assertion too.
    Its Intent says so.
    Card 10's Intent no longer claims that deletion.

- **F3 (fixed differently).**
  The premise holds: each cited comment names a retired row and sits outside every card's targets.
  The proposed glyphs name the wrong enclosing members:
  - `parity_test.go:292` is in `TestGateParity_PlanGate`'s doc, not in `planParityCase`'s.
  - `validate_test.go:260` is in `TestValidatePlanCmd_InformationalFindingsSurfaceOnSuccess`'s doc, not in `glyphPlanFixture`'s.
  - `discussionparser/validate_test.go:246` is in `TestMissingSections_ASingleHugeLineDoesNotHideEveryHeadingBelowIt`'s doc, not in `TestValidate_DecisionRecordPathIsDirectory`'s.
  - `TestPublishVerify_FailureRecord` is the right glyph for `publish_verify_test.go:366`.

  Card 6's `**Edit:**` list now carries the correct members.
  It also lists the `overlay_seam_guard_test.go` path, as proposed.
  For the card's "none points at a row that no longer exists" claim to hold, it also covers two more uncovered comments that describe the shipped rows:
  - `TestWire_DefaultConfig`'s review-segment comment (`internal/loomcli/wiring_test.go:329`);
  - `TestRun_EpisodeResetsOnProducersOwnDone`'s loom-shape comment (`internal/shedengine/run_routing_test.go:673`).

  The Intent's covered list names all of them.
  Strings in synthetic fixtures (`goto_test.go`, `step_test.go`, `teardown_test.go`, `reviewstatus_test.go`, and the verify-gate labels) are test inputs, not references to shipped rows, and are left alone.
  Comments inside tests that card 4 already edits follow card 4's own test rewrite.

## Disputed

None.

## Deferred

None.
