---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: internal/reedengine/buttonclick_integration_test.go:20
    summary: "No real-tmux test runs the ask path, so the command-prompt template, the %%% escaping and the empty-Enter default are never exercised against tmux"
    fix: "In internal/reedengine/buttonclick_integration_test.go (the tier2 test that already drives a confirm entry through a real tmux client), add an ask entry with ask_default whose run writes {answer} to a file in the test's temp dir, and drive its command-prompt the way that file drives confirm-before. Type an answer holding a single quote, a double quote, a backslash, a dollar sign, a semicolon, a tilde, an inner space and the text '#{session_name}', press Enter, and assert the file holds the answer byte for byte. Type only Enter and assert the file holds the ask_default. Send Escape and assert the file is not written and the next click does not reuse the old answer. Keep it one table-driven test with one row per case."
    origin: lens:code-affected-set
  - id: F2
    severity: LOW
    class: design
    location: internal/reedengine/buttonrow.go:121
    summary: "The comma ban covers only the static ask text; a comma in a filled placeholder value (slug, worktree, session, beam, strand) still reaches command-prompt -p, where tmux splits it into several prompts"
    fix: "In composeButtonRowLocked (buttonrow.go), after filling the entry's Ask, skip the entry with a logger.Warn naming the label and the offending placeholder value when the filled text holds a ','. In runButtonVia (buttonclick.go), before raising the command-prompt, show 'lym: <label>: the ask text holds a ','' on the clicking client and return when the filled ask holds a ','. Add one row to TestPinButtonRowLocked (ask 'Name for {slug}:' with a slug holding a comma is left out) and one row to TestRunButtonVia (a strand value holding a comma in the ask raises no prompt)."
    origin: lens:code-correctness
  - id: F3
    severity: LOW
    class: scope
    location: internal/reedengine/buttonclick_test.go:100
    summary: No test row pins that an entry with both ask and confirm raises the ask prompt first on the first click
    fix: "In TestRunButtonVia (buttonclick_test.go), add a row AskAndConfirmEntryRaisesTheAnswerPromptFirst using the existing askingConfirming entry with answered and confirmed both false and wantPrompt: askFor(askingConfirming, 'Name: ', nil), so a click that raised confirm-before first would fail it."
    origin: lens:code-tests
  - id: F4
    severity: LOW
    class: scope
    location: internal/reedengine/buttonrow.go:110
    summary: "The render now reads {strand} and {beam} from the ask text through entry.names, and no render test has either placeholder in an ask text"
    fix: "In TestPinButtonRowLocked (buttonrow_test.go), add two rows modelled on the existing run and confirm rows for those placeholders: an ask entry with window: agents whose only {strand} is in the ask text expects Strand true in wantEntries, and an ask entry whose only {beam} is in the ask text is left out with the beam-absent Warn when the beam is not on this machine and kept when it is."
    origin: lens:code-tests
  - id: F5
    severity: LOW
    class: consistency
    location: internal/reedengine/doc.go:67
    summary: "Two sentences of the reedengine package doc still describe the entry as it was before ask and ask_default: the validity rules at line 67 and the spec contents at line 72"
    fix: "In internal/reedengine/doc.go line 67, change 'no '#', '%' or control character in a label or confirm text' to 'no '#', '%' or control character in a label, confirm text or ask text, no ',' in an ask text, ask_default and {answer} only with an ask'. In line 72, change 'its label, its run and confirm text with the static placeholders filled and whether {strand} is left for the click' to 'its label, its run, confirm and ask text with the static placeholders filled, its ask_default, and whether {strand} is left for the click'."
    origin: lens:code-affected-set
  - id: F6
    severity: NIT
    class: consistency
    location: internal/reedcli/cli_test.go:75
    summary: The doc comment of TestRunCLI_Switch_RefusesBadFlagsAndNeedsNoRepository names only a --confirmed without --hash among the button refusals
    fix: "In internal/reedcli/cli_test.go line 75, change 'a --confirmed without --hash' to 'a --confirmed or --answered without --hash', matching validateButtonFlags."
    origin: lens:code-affected-set
---

### [MEDIUM] No real-tmux test runs the ask path

**Location:** internal/reedengine/buttonclick_integration_test.go:20
**Issue:** The whole ask design rests on tmux's own behaviour: `command-prompt` runs its template, `%%%` escapes the answer for a double-quoted string, and an Enter on an empty prompt runs the template with an empty answer so `ask_default` applies.
The unit rows in `TestRunButtonVia` compare argv strings against a fake tmux, so a wrong template or a tmux that treats an empty Enter like Escape would pass every test.
The change description cites a manual check on tmux 3.6, which no file in the diff keeps.
The real-tmux test that already covers the confirm path has no ask entry.
I could not settle the focus question about metacharacters by running tmux either: my own probe hung and was removed, so my verdict on that question rests on reading the code and on the writer's claim.

### [LOW] Filled placeholder values can still split the ask prompt at a comma

**Location:** internal/reedengine/buttonrow.go:121
**Issue:** `buttonEntryProblem` refuses a comma in the static ask text because tmux splits `command-prompt -p` at commas.
`fillPromptPlaceholders` then puts `{slug}`, `{worktree}`, `{session}`, `{beam}` and `{strand}` into that text, and a comma in one of those values, such as a worktree path, reaches `command-prompt` unchecked.
The prompt then becomes several prompts and the template's `%%%` takes a different input than the one the click expects.

### [LOW] The order of ask before confirm has no test row

**Location:** internal/reedengine/buttonclick_test.go:100
**Issue:** `askingConfirming` is used only with `answered` or `confirmed` set.
The branch that decides a first click on an entry with both prompts raises `command-prompt` and not `confirm-before` (`asking` in `runButtonVia`) is pinned only for an entry with ask alone.

### [LOW] The render's use of the ask text for {strand} and {beam} is untested

**Location:** internal/reedengine/buttonrow.go:110
**Issue:** `composeButtonRowLocked` and `usesBeamPlaceholder` now call `entry.names`, which includes the ask text.
The one `{strand}`-in-ask row in `buttons_test.go` is an invalid row and never reaches the render, and the click tests build `Strand: true` by hand, so neither the `Strand` flag nor the beam-absent omission is pinned for the ask text.

### [LOW] Package doc still describes the pre-change entry

**Location:** internal/reedengine/doc.go:67
**Issue:** The diff updated the click-flow paragraphs of the doc but not the sentence listing the validity rules or the sentence describing the spec's contents.
Both omit `ask`, `ask_default`, the comma rule and the `{answer}` rule.

### [NIT] Stale test doc comment

**Location:** internal/reedcli/cli_test.go:75
**Issue:** The comment lists the button refusals the test covers and names only `--confirmed` without `--hash`; `--answered` without `--hash` is now a refusal of the same verb.

## Rejected

- Lens code-tests 4, an ask click with an unresolvable executable has no row: the branch is the same code that the confirm row `ConfirmEntryWithAnUnresolvableExecutableRaisesNothing` already runs, so a row would duplicate that coverage.
- Lens code-tests 5, the `--answer` word has no PowerShell row: quoting is `shell.Shell.Quote`, which each dialect's own tests pin, and the re-entry test already runs both dialects through `composeButtonReentryLine`.
- Lens code-tests 6, an empty answer with an empty `ask_default` has no row: it is the same `AskDefault` branch as the covered empty-answer row, and the record gives a default only the role of filling an empty answer.
- Lens code-correctness 2, `{answer}` in the ask text is accepted: the record asks only that `{answer}` be refused in an entry without ask, which holds; the text shows literally and harms nothing.
- Lens code-correctness 3, `ask_default` is not covered by the hash: the hash guards the run text that executes, and a default changed between prompt and re-entry by a re-render is not a safety gap.
- Lens code-correctness 4, `@lym_button_answer` is shared by every client of a session: the window is the time between the template's `set-option` and the re-entry's read, and the record names no concurrent-client requirement.
- Handler, the `--answered` and `--answer` flags, the `%%%` template and the comma ban are not in the record: each is the mechanism the record's design needs under tmux's prompt rules, and none adds behaviour an operator sees beyond it, so none is a scope finding.
- Handler, the shipped `agent` buttons use `ask: Agent name:`, `ask_default: agent` and `--role {answer}`, and the `term` buttons are labelled `terminal`, in both templates: they match the record and `config_test.go` pins them.
- Handler, no PATTERN entry is touched or declared changed and the single commit carries no `Pattern-Departure:` trailer, so nothing is owed there.
- Handler, `_lym/landing/summary.md` claims `TestPinButtonRowLocked` pins the filled ask and default, and it does; its other claims match the diff.

## Focus departures

- Directive: "does the command-prompt template carry an answer containing tmux metacharacters (quotes, backslash, semicolon, dollar, tilde, leading or trailing space, a literal %) intact into the @lym_button_answer option, and does the shell-quoted {answer} in the run text then reach the shell as one word?"
  I settled the shell half by reading `placeholderReplacer` and `sh.Quote`, which quote the whole value as one word.
  I could not settle the tmux half by running it: my live probe hung and was removed, so the escaping claim rests on reading the template and the writer's manual check, and F1 asks for the test that would settle it.
