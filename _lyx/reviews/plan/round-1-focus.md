---
round: 1
focus:
  - "Card 10 Edit targets: settle whether `strand_test.go` and `zoom_integration_test.go` need member or file glyphs, since `Intent` changes `TestZoomBracket_ZoomedStrandSurvivesEveryReedStep` and appends `TestSetContentRows`, and whether `applyOpts` and `Engine.applyLayoutLockedOpts` need the re-sign arrow."
  - "Card 10 Create and Edit placement: settle whether `Engine.SetContentRows` is a Create target in `strand.go` and `TestSetContentRows` a Create target in `strand_test.go`, and whether the `Intent` names the files the glyph prefixes imply."
  - "Fidelity to `_lyx/discussion/decision-record.md`: settle whether Scope Out and D7 license restoring `SetContentRows` and `InstallPins`, whether the rework round's `findings.md` is covered, and whether any restored line exceeds that licence."
  - "Test economy: settle whether `TestSetContentRows` duplicates behaviour an existing reedengine test already covers, and whether its rows are all reached."
  - "Reachability of the refusal branches `SetContentRows` restores (`rows < 1`, unknown name, non-content strand, wrong pane): settle that each has a named input and a named test row, and that no upstream check rejects the input first."
  - "Verify coverage: settle whether the `## verify:` section and the card's `**Verify:**` run the tests of `internal/reedengine` across every tag its edited test files carry, including `tmux`, which the plan only vets."
  - "Attack-surface question: settle whether the `InstallPins` early-return and `select-pane` skip in `applyLayoutLockedOpts` state what they let through and what bounds it."
---

Single-card rework plan restoring an operation a prior fix round deleted.
