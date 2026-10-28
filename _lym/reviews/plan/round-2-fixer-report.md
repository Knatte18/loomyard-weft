# Fixer report — plan, round 2

Every finding's premise held against the code at HEAD, and all five are fixed.
`lym loom validate-plan` answers `ok` over the changed plan.

## Changes

- **F1** (card 15, Tests): fixed differently.
  The row whose interrupt function closed the store's journal now advances the fake clock past the next UTC midnight and creates a directory at the next day file's path, so the `interrupted` append rolls to that file, `rotate` cannot open it, and `append` returns the error.
  Its assertions are kept.
  I did not take the proposed "removes the journal directory": today's day file is still open when the interrupt function runs, and Windows refuses to remove a directory that holds an open file.
  An existing directory at the file's path fails `os.OpenFile` on every OS (fix-claims self-check finding).
- **F2** (card 18): the bell assertions of the idle-wake and interrupt-and-bell scenarios now read `events.jsonl`'s `UserPromptSubmit` payloads instead of the pane.
  Idle wake wants none whose prompt holds `creelengine.BellLine`.
  Interrupt and bell wants exactly one, counted after the message has shown.
  Pane polling stays for the message text, and `plan:internal/creelengine#BellLine` is under **Uses:**.
  Fixed differently in one point: the match is "contains", not "equals".
  A bell typed over the idle-wake draft would be submitted with the draft before it, so an equality match passes on the defect the scenario exists to catch (fix-proposal self-check finding).
- **F3** (card 15): `Store.interruptTarget` moved from **Uses:** to **Edit:**, and `holdRefusal` and `unreadablePauseRefusal` were added under **Edit:**.
  A new Intent sentence says `interruptTarget` returns its three refusals as a typed error that carries the refusal's kind and today's message.
  `Interrupt` and `inject --interrupt` therefore answer as they do today, and `PostInterrupt` matches the error with `errors.As`.
  A second new sentence says that a false-with-error result that is none of the three, a failed key, is returned as `interruptTarget` words it, with nothing stored.
  The ImpactSummary names the typed refusal.
- **F4** (card 7): fixed differently.
  The guarded rows gain a too-short pane and a pane with no input box, neither showing a running turn.
  Each reads `NoTurn` false, with `Idle` false and a zero `TurnEndedAt`.
  In the New rows paragraph, the "too-short pane under the pane probe" row now points at the guarded rows instead of being a second row.
  Carrying out both halves of the proposal would have created two identical guarded too-short rows.
- **F5** (card 8, Bound): replaced as proposed.
  The unregistered-provider branch is reached only in tests, and hand-added strands reach the empty-provider branch in production.

## Self-check

- fix-consistency, LOW: card 15's new failed-key outcome had no test row.
  Fixed: `TestStore_Mail` gains a row where `SetInterruptError` returns an error other than `ErrNoRunningTurn`, and the post answers that error as `interruptTarget` returns it, with nothing stored.
- fix-consistency, LOW: no test pins the unchanged refusal messages of `Interrupt` and `inject --interrupt`.
  Fixed: the Tests paragraph names `TestStore_InjectAndHold`'s existing lifted-pause and unreadable-pause rows (`verbs_test.go:472`, `:475`, `:490`), which stay unchanged, and the test is listed under **Uses:**.
- fix-consistency, LOW: card 18's closing check, that no reminder or notice line is typed, still read the pane.
  Fixed: it now asserts that `events.jsonl` holds no `UserPromptSubmit` payload containing such a line.
- fix-proposal, MEDIUM: an equality match on `BellLine` misses a bell submitted together with a draft.
  Fixed: both scenarios match "contains" (see F2).
- fix-claims, LOW: removing the journal directory may fail on Windows while the day file is open.
  Fixed: the row creates a directory at the next day file's path instead (see F1).
- fix-affected-set: no findings.

I saved the lens reports after making these edits, not before judging as the procedure orders.
Each report is the fork's final message, verbatim.

## Disputed

None.

## Deferred

None.
