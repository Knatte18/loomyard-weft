```yaml
lens: code-tests
outcome: reported
```

Lens `code-tests`: for each behaviour the diff adds or changes, the test row that pins it, or none. I made no tool calls and read nothing beyond my inherited context.

**Per behaviour**
- Ask click raises `command-prompt` with the exact argv, the `%%%` template and the trailing-space prompt: pinned by `TestRunButtonVia`/`AskEntryRaisesTheAnswerPromptAndRunsNothing`.
- Ask entry naming `{strand}` carries the strand into the answered line: `AskEntryNamingStrandCarriesTheStrandInTheAnsweredLine`.
- Answered click: stored answer trimmed and shell-quoted: `AnsweredCallRunsWithTheStoredAnswerTrimmedAndQuoted`.
- Answered click: empty answer takes `ask_default`: `AnsweredCallWithAnEmptyAnswerTakesTheAskDefault`.
- Answered click: failed answer read runs nothing and shows the message: `AnsweredCallWithAFailedAnswerReadRunsNothing`.
- Answered click: changed run text is rejected by the hash check: `AnsweredCallWithAnotherHashRunsNothing`.
- Answered `{strand}` entry uses the carried name and not the pane: `AnsweredStrandEntryRunsWithTheCarriedNameNotThePanes`. Removing `|| req.Answered` in `buttonStrand` would fail this row.
- Answered entry with confirm raises `confirm-before` carrying `--answer`, with `#` doubled in the prompt: `AnsweredConfirmEntryRaisesTheConfirmPromptCarryingTheAnswer`.
- Confirmed ask entry runs with the carried answer and reads no option: `ConfirmedAskEntryRunsWithTheCarriedAnswerAndReadsNoOption`.
- Re-entry line, both stages, strand and answer words, tmux quoting: `ReentryLineAndTmuxQuoteForBothDialects`. `tmuxString` is pinned in the same test.
- `validateButtonFlags` refuses `--answered` without `--hash`: `TestValidateButtonFlags`.
- Entry validation: `TestPlaceButtons` pins every new rule.
  - `#`, `%`, a control character (tab) and a comma in `ask`.
  - `{strand}` in `ask` without `window: agents`.
  - `{answer}` in run or confirm without `ask`.
  - `ask_default` without `ask`.
  - A valid ask entry with `{answer}` and `ask_default` is kept.
- Render fills the ask text, carries `ask_default` and leaves `{answer}` for the click: `TestPinButtonRowLocked`/`AskEntryCarriesItsFilledAskAndDefaultAndLeavesAnswerForTheClick`.
- Shipped `agent` buttons (ask, default, `--role {answer}`) and `terminal` labels: `TestLoadConfig_TemplateDefaultsResolve` and `TestPlaceButtons`/`TemplateTaskGroupComesBackWhole`. I did not read how these cover both the POSIX and Windows templates.
- Escape at the ask prompt runs nothing: no Go code, a tmux behaviour. Nothing to pin.

**Findings (all LOW, class `design`)**
1. `buttonrow.go:123` — `Strand` is set from `entry.names(buttonStrandKey)`, which now includes the ask text. No render test has an ask-only `{strand}` entry. The click tests hand-build `Strand: true`. The one ask-text `{strand}` row in `buttons_test.go` is invalid-only, so it does not reach the render. Evidence: `buttonrow_test.go` diff adds only the `{answer}` row.
2. `buttonrow.go:110` and `buttonrow.go:140` — `{beam}` in the ask text now counts for both the beam-absent omission and `usesBeamPlaceholder`. No row has `{beam}` in an ask text. Evidence: no `{beam}`-in-ask row in the `buttonrow_test.go` diff.
3. `buttonclick.go:145-158` — an entry with both `ask` and `confirm`, clicked first, must raise `command-prompt` and not `confirm-before`. `askingConfirming` is only used with `answered` or `confirmed` set, so no row pins the ask-before-confirm order. Evidence: the `buttonclick_test.go` rows.
4. `buttonclick.go:147-152` — an unresolvable executable on an ask click shows the "could not raise its prompt" message. Only the confirm entry has an `exeErr` row (`ConfirmEntryWithAnUnresolvableExecutableRaisesNothing`). The branch is shared, so a break is unlikely.
5. `panebin.go:183-187` — the `--answer` word is only tested in the POSIX dialect. The PowerShell re-entry row has no answer, so quoting of an answer containing `'` in the Pwsh dialect is unpinned.
6. `buttonclick.go:208-211` — the `ask_default` fallback is only tested with a non-empty default. A click on an entry with an empty answer and an empty default fills `{answer}` with `''` and has no row.

**Tool calls:** none.
