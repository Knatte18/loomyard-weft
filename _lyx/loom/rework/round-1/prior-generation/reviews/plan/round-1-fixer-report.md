# Plan-Burler round 1 — fixer report

## Fixed

- **F1** (BLOCKING): added card 11 `color-through-send` and card 12 `delete-color-sequence`, and listed both in the overview's Card Index with one framing sentence.
  Card 11 adds `Engine.ColorCommand` (implemented by `Claude` and both fakes), routes start's color step and `Runner.TypeColor` through `sendVerified` after readiness, logs a failed color send without failing the launch, states the attack-surface bound, extends the existing color tests (`TestRunner_Start_TypesTheSegmentColorAfterStartup`, `TestRunner_TypeColor_PlaysTheEngineSequenceOnALiveShuttleStrand`, `TestColorSequence_TypesTheColorCommandForEveryPaletteColor`, `shuttlefake_test.go`'s assertions) and carries a `**Verify:**` of `go test -tags tmux ./internal/loomcli/... ./internal/shuttleengine/...`, the run the decision record names.
  The cards are appended after card 10 rather than inserted after card 4 so the executed cards 1–10 keep their numbers; card 11 uses only symbols cards 3 and 4 create, so the order rule holds.
  Card 12 deletes the then-uncalled `ColorSequence`, kept separate per the spec's "removing the last caller of a symbol and deleting the symbol are two cards".
  `TypeColor`'s one production caller is `orchcli`'s `runnerSession.TypeColor` (the orch watcher); the loom driver is colored by start's step, not by `TypeColor`, so the ImpactSummary names the orch watcher only.
- **F3** (NIT): card 1 names `TestLoadConfig_SubmitSettleKeys`; card 5 names the new table test `TestStartup_SendFailuresTearDown` and why no existing test covers it; card 8 states that `TestBurlerProducer_Call_ArchiveOnExit`'s `TwoConsecutiveDiedWrapsErrNotStartedOnlyWhenNotStarted` subtest already pins the attempt-2 `ErrNotStarted` wrap and that no row is added.

`lyx loom validate-plan` passes on the edited plan.

## Disputed

- **F2** (LOW): both premises are false.
  - "A deletion that belongs to the card that orphans the symbol": `loom-plan-spec.md`, "Card granularity", says "Removing the last caller of a symbol and deleting the symbol are two cards, not one … and only one final `Delete` card once all are gone."
    Card 10 is that final `Delete` card, and the fix's first option (folding the deletions into cards 8 and 9) would violate the spec.
  - "PATTERN-shuttle-stop is false when it lands": the entry says no other package *removes* a shuttle run's strand.
    At card 9's commit, card 8 has dropped every caller of `burlerengine.StrandRemover`, `NewReedStrandRemover` and `shedfake.StrandRemover`, and card 9 drops every caller of `websterengine.removeStrandIfLive` (card 10's Intent: "the removers cards 8 and 9 left without callers").
    Uncalled code removes no strand, so the entry is true at card 9's commit.

## Deferred

Nothing deferred.
