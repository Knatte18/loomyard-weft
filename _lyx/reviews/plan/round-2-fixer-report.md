# Fixer report — plan, round 2

## Fixed

- **F1** (`09-webster-stops-through-shuttle.md`): fixed per the parent's decision (option a, from `ly:orch`).
  The quoted `PATTERN-shuttle-stop` entry now names a second exception, `orchcli`'s removal of the orch strand.
  The card states the reason: `orchcli` starts the orch session through `Runner.Start`, so it has a run record; its `stop` and relaunch remove the strand raw, which is out of scope; and no attach probe or `Wait` reads that record, because `orchcli` keeps only the strand guid.
  The Intent's opening sentence now says no other package removes a shuttle run's strand except the two named exceptions.
  The decision record's false Out-list premise about orch is resolved: the parent added the entry "Orch strand removal is a second PATTERN-shuttle-stop exception" through `lyx loom decision add`, and card 9's wording matches it.
- **F2** (`11-color-through-send.md`): `Claude.Prepare` now holds back the prompt pointer for a spec that shuttle colors (`spec.Segment != "" && !spec.ColorByCaller`), not only for one that names skills.
  Start therefore sends the color into a session that has not taken a turn yet, then sends the prompt through the verified send.
  The card now edits `Launch`'s field comment, `Claude.Prepare` and both `doc.go` sentences.
  `TestRunner_Start_TypesTheSegmentColorAfterStartup` gains a row for a colored spec without skills.
  `TestPrepare_SkillsDeferPromptPointer` is renamed `TestPrepare_DeferredPromptPointer`, and its loop gains two specs.
  The Bound says that delivering the prompt in start makes it subject to card 5's teardown.
  Part of the finding's premise is false: every production spec with a segment already names skills, burler's halves included (`burlerengine/engine.go`, `Skills: burlerSkills`).
  So no production launch stalls today; the card says so in its Bound and ImpactSummary.
- **F3**: each test instruction now names one existing test or states that it adds a new test or row.
  - Card 2: two new test functions, `TestClearInputSequence` and `TestPastePlaceholder`.
  - Card 5: `TestStartup_RunGatedMappings`, with its mechanism-failure row reworded and no row added.
  - Card 6: two new tables, `TestGate_WaitsForIdleWriter` and `TestGate_UnsentRepromptRetries`; the retry and deadline rows go in the second, and `TestGate_SendFailsMidLoop_EndsLoopWithAttemptsSoFar` is left unchanged, since it is not a table.
  - Card 8: `TestEngine_Run_RoundFailureRules`, in its existing second-half-not-started row.
  - Card 9: one line per test file, with a new `stopper_test.go` holding the shared double and a new `TestRemoveRecoveryStrands`; it moves to the Create list, and `strand_test.go` leaves card 9's Edit list.
  - Card 11: the `TypeColor` test is restructured into a table and renamed.
- **F4** (`11-color-through-send.md`): card 11 renames `TestColorSequence_TypesTheColorCommandForEveryPaletteColor` to `TestColorCommand_NamesEveryPaletteColor` and rewords `color_test.go`'s header.
  It also renames the `TypeColor` test to `TestRunner_TypeColor_SendsTheColorCommandOnALiveShuttleStrand`.
  After card 12, no test name or header still mentions `ColorSequence`.
- **F5**: card 3 creates `Run.newSendContext` and `Runner.newSendContext` and names them in its Intent; card 11's Uses names both.

## Disputed

None.

## Deferred

- **F1, `PATTERN.md`**: card 9 has already landed, so `PATTERN.md` still carries the old one-exception wording.
  `PATTERN.md` is outside this round's write surface, so the reworded entry reaches it only when the plan's card 9 is re-executed.
