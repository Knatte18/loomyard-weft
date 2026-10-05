---
round: 1
focus:
  - "Card 6 Intent, reflection stencil drop rule: D5 downgrades a deterministic Go filer to agent judgement and names a bound (evidence stays on disk, `lyx selfreport create` remains).
    Settle whether the card's own `**Intent:**` states what the drop rule can let through and what bounds it, per the attack-surface question."
  - "Card 5 Intent, voucher written by `lyx loom start`: settle that the stated bound covers every voucher-suppression case D4 lists, and that the card's tests pin the one-shot and Tier-2-off bounds rather than only the happy path."
  - "Verify coverage: for every package a card targets (`cmd/lyx`, `contracts/stencils`, `internal/friction`, `internal/shedverbs`, `internal/webstercli`, `internal/orchcli`, `internal/loomcli`, `internal/loomengine`), check the `## verify:` section runs its tests, including the build tags of the specific test files cards 2 and 4 edit (`wiring_test.go`, `smoke_*_test.go`, `smoke_live_test.go`)."
  - "Card 1 Delete list against its Intent: `internal/loomengine/loomstatus_test.go` is deleted whole while the Intent says only the removed accessors' tests go.
    Read the file and settle whether it holds tests that survive; also settle whether `awaiting_test.go` and `selfreport_test.go` keep the helpers other files use."
  - "Fidelity: every D1–D6 decision and every Constraint in the decision record is carried by some card, and no card adds unlicensed scope.
    Check in particular the three-file rename in card 3 (including `shedverbs/spec.go`) and the `RecentHistory` / `Vouched` field changes in card 4 against what the record licenses."
  - "Design, sequencing across cards 1→4: card 1 leaves `DetectCrashResume` and `ClassifyHalt` with no production caller and card 4 re-edits the same files (`anomaly.go`, `arm.go`, `halt.go`, `cli.go`).
    Settle whether each card's tree builds and passes `go test ./...` on its own, and whether the split between cards 1, 3 and 4 is one reviewable unit per card or splits and re-touches the same code needlessly."
  - "Test economy in card 4 and card 5: settle whether any planned new test duplicates an existing one (for example the voucher consume-once behavior against the existing `TestConsumeStepHandoffMatch`, or the crash-resume no-note table against existing observation tests), naming the existing test."
---

Seed focus from reading the plan, the decision record and the six cards.
No entry restates a gate-checked ID or a `Do not flag` item.
