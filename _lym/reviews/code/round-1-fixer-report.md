# Fixer-report, round 1

All six findings fixed as proposed; the self-check forks' findings are judged below.
Gates: `lym gate test ./internal/reedengine ./internal/reedcli` (tier 0) and `lym gate test --tier 2 ./internal/reedengine` both pass.

## Changes

- F1 (commit 459c7591): `TestRunButton_RealTmux` gains an ask entry and three rows driven through the real client: an answer holding every listed metacharacter reaches the file byte for byte, Escape runs nothing, and an empty Enter takes `ask_default`.
  The pty harness (`attachGeometryPTY`) now keeps the child's output, with `drew` and `forget`, so the test types only once the prompt is drawn.
  The answered re-entry is driven by the test itself, as the confirm path is, because the recorder stands in for the lym binary.
- F2 (commit edd09e87): `composeButtonRowLocked` leaves out an entry whose filled ask holds a `,`, with a Warn naming the label and the filled ask; `runButtonVia` shows `lym: <label>: the ask text holds a ','` and raises no prompt.
  Rows: `AskHoldingACommaFromAPlaceholderValueIsLeftOutWithAWarn` (beam anchor with a comma, since the test context's slug is fixed) and `AskTextHoldingACommaFromAStrandValueRaisesNoPrompt`.
- F3 (edd09e87): row `AskAndConfirmEntryRaisesTheAnswerPromptFirst`.
- F4 (edd09e87): rows `AskNamingStrandFlagsStrandForTheClick`, `AskNamingAnAbsentBeamIsLeftOutWithTheBeamWarn`, `AskNamingAPresentBeamIsKeptWithTheBeamFilled`.
- F5 (commit 3c226082): `doc.go` validity-rule and spec-contents sentences updated, with the comma rule from F2 and the `@lym_button_answer` entry in the decision-bearing option list.
- F6 (3c226082): the `cli_test.go` comment now names `--confirmed` or `--answered` without `--hash`.
- Self-check fixes (last commit): see below.

The F2, F3 and F4 fixes share test files, so they are one commit rather than three.

## Self-check

- fix-code-correctness, MEDIUM: Escape row's absence check can pass with the defect in place. Fixed with my own change: the row seeds `@lym_button_answer` with `stale` and asserts it is unchanged after Escape, and the empty `answerPath` stat is dropped. The `recordPath` stat stays as a secondary check.
- fix-code-correctness, LOW: `forget()` could meet a late redraw of the previous prompt. Disputed, premise false: see Disputed.
- fix-proposal, LOW: comma guard sat after the executable resolution. Fixed: it now runs right after `asking` is computed.
- fix-proposal, LOW: the skip is visible only in the log. Disputed, every fix worse: see Disputed.
- fix-proposal, LOW: Escape row's `recordPath` stat is racy. Merged with the correctness MEDIUM; fixed by the seeded option.
- fix-proposal, LOW: Escape row never performs a next click. Merged with the same fix; the seeded option plus the following empty-Enter row (which must read `dflt`, not `stale`) cover reuse.
- fix-consistency, LOW: `runButtonVia` doc omits the comma message. Fixed.
- fix-consistency, LOW: `doc.go` RunButton check order omits the comma refusal. Fixed.
- fix-affected-set, LOW: `buttons_test.go` header, `buttonclick.go` `sh` field comment, `bar.go` and `follow.go` option lists were stale. Fixed.
- fix-affected-set, LOW: no `--answered` without `--hash` row in `cli_test.go`. Fixed with a row beside the `--confirmed` one.

## Disputed

- premise false, fix-code-correctness LOW (`forget()` and a late redraw): the text `Say what:` is drawn only while the prompt is open, and a row starts after the previous click has returned, which happens when its prompt has closed; no later redraw contains the text. The tier-2 run passes.
- every fix worse, fix-proposal LOW (the comma skip is log-only): the only alternative to leaving the entry out is refusing the whole render or drawing a button that cannot prompt, and the beam-absent skip in `composeButtonRowLocked` already uses the same Warn-and-continue.

## Deferred

Nothing deferred.
