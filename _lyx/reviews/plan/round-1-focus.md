---
round: 1
focus:
  - "Card 3 (`paste-stdin-buffer`): the `Edit` members that change signatures (`pasteSubmitArgv`, `fakeTmux.exec`, `TmuxCmd.execHook`) carry the re-sign arrow where the Intent changes parameters or results; settle whether `TmuxCmd.run`, `TmuxCmd.output` and `installFakeTmuxOn` need member glyphs or arrows given their Intent."
  - "Card 4 (`shortcutcli-package`): settle granularity of the Create targets (private helpers such as `parse`, `mount`, `routedWords`, `runArgvs`, fakes `fakeRootFactory` and `fakeRecorder` against the public `Load`, `Mount`, `Shortcut`) and test economy of `TestShortcut_Run` against `TestMount_File`."
  - "Card 4 and card 5 (`mount-shortcuts`): settle the attack-surface question for the new top-level shortcut verbs and the module guards that skip them: does each Intent state what the shortcut can now run or let through, and what bounds it (fresh root per argv, `--` handling, extra args)?"
  - "Every prescribed refusal or guard branch in cards 2, 4 and 5 (`parse` rejecting a malformed shape, `routeWords` on unknown words, the guard skips): settle that each names an input and a test that reaches it and that no upstream check rejects that input first."
  - "Card 1 and card 2 `Edit` cards: settle that each `ImpactSummary` is a real blast-radius conclusion, not a restatement of Intent, and that card 1's `overviewRun`, `overviewSource` and `overviewCmd` members match the Intent's changes."
  - "Card order: card 5 depends on card 4's package and card 1 changes `newOverviewSource`'s signature; settle that no card uses something a later card creates, and that card 6 (`loom-status-eighth`) has no unstated ordering need."
  - "Verify section: settle that every package a card targets (`internal/battencli`, `internal/seatengine`, `internal/reedengine`, `internal/shortcutcli`, `internal/loomcli`, `cmd/lyx`, `internal/statuspane` if edited) is run by the `go test` rows, that the `tmux` tier card 3 touches is compiled by `go vet -tags tmux`, and that the section order matches the plan template."
  - "Fidelity: settle that every Decision (D1 to D6) and Constraint in `_lyx/discussion/decision-record.md` is carried by a card, that no card adds scope the record does not license, and that card 5's docs and test-budget edits are licensed."
---

Seed focus for round 1, set from the plan's overview and card targets.
No entry restates a check the round's own gate already enforces.
