```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. List the test that pins each behaviour the diff adds or changes.

**Per behaviour, the test that pins it**

- **Terminal branch prompts with `tokenPrompt`, stores the trimmed hidden line, and ignores stdin.**
  - Pinned by `Login/OnATerminalPromptsAndStoresTheHiddenLine` (`cli_test.go:390`).
  - The prompt is asserted exactly, and so is the stored `xoxb-typed`, even though the piped stdin holds `xoxb-piped`.
- **Piped input is unchanged and prints no prompt.**
  - Pinned by `Login/StoresTheTokenOnlyAfterSlackAcceptsIt` (`cli_test.go:368`, with the new `len(h.prompts) != 0` assertion at `:385`).
  - This is an assertion added to the existing test, not a duplicate test.
- **Empty piped input is refused with the reworded message, with nothing stored and no Slack call.**
  - Pinned by `Login/EmptyStandardInputIsRefusedWithoutAskingSlack` (`cli_test.go:438`).
  - It asserts only the substring "no token on standard input". The new way-forward text ("or run it from a terminal and paste the token at the prompt") is not asserted.
  - The old "Ctrl-D" wording is not asserted absent. That is a message-wording check, so I do not count it as a gap.
- **Terminal empty line is refused ("no token entered at the prompt"), with nothing stored and no Slack call.**
  - Pinned by `OnATerminalAnEmptyLineIsRefusedNamingThePrompt` (`cli_test.go:411`).
  - The row also asserts that Slack was not reached and that no token file exists.
- **Terminal read error, such as an interrupt, is refused with "nothing was stored" and the way forward, with nothing stored.**
  - Pinned by `OnATerminalAnInterruptedReadIsRefusedNamingLogin` (`cli_test.go:412`).
  - The row only exercises the `readHiddenLine` error path in `readToken`, through a fake.
- **Pane check still runs before the prompt.**
  - No new test, and none needed.
  - The code order is unchanged: `paneRefusal` precedes `readToken`.
  - The existing pane-refusal rows pin the refusal. The only place a pane-then-prompt ordering could break is a refused-pane run that records a prompt, and no test asserts `h.prompts` is empty on a refused pane. That is a minor gap, and the behaviour is unchanged in structure, so I do not raise it as a finding.
- **`readHiddenTerminalLine` (`wiring.go`): the terminal check, the signal restore, the `Fprintln` and the `GetState` failure.**
  - No test.
  - It is untestable under tier0 without a pseudo-terminal, as `summary.md` states.
  - It sits behind the `readHiddenLine` seam, which is faked.
  - Non-`*os.File` and non-terminal input returning `onTerminal == false` is not directly pinned either.
  - `h.run` uses `strings.NewReader`, but the harness replaces the seam, so production's `in.(*os.File)` branch is never exercised.
  - This is a gap by the lens's literal rule.
- **`readToken` non-terminal read error ("read the token from standard input").**
  - No test. The harness uses `strings.Reader`, which cannot fail.
  - The path is pre-existing, moved unchanged, and misuse-only, so I do not count it as a finding.
- **Help text, `doc.go` and `tokenPrompt` prose.**
  - These are static prose, so they are exempt.

**Findings**

1. **LOW / design.**
   - Location: `internal/slackcli/wiring.go` `readHiddenTerminalLine`, no covering test (evidence: `cli_test.go:204-210` fakes the seam).
   - Summary: the production terminal-detection and signal-restore logic has no test, so the claim "restores the terminal on interrupt" is unpinned.
   - Reason it stays LOW: it needs a pty. Reading the pty through `creack/pty` or `term` in a tier1 test is possible if the repo already has a pty kit. If it has none, record the gap as accepted instead of adding a dependency.
   - Fix: either add a tier1 pty test of `readHiddenTerminalLine`, covering a non-terminal `*os.File` returning `onTerminal == false`, which needs no pty and can run at tier0 through `export_test.go`. Or state the gap explicitly.
   - The cheap piece is the non-terminal `*os.File` and non-`*os.File` rows, so add those.

2. **NIT / consistency.**
   - Location: `cli_test.go:441-442`.
   - Summary: the empty-pipe refusal test pins only "no token on standard input", so the changed way-forward wording is unpinned.
   - Whether it needs a row is the handler's call.
   - Fix: add `"lym slack login"` to the `wantRefused` arguments.

No test duplicates another: the terminal rows are table-driven, and the piped-prompt assertion extends an existing test.

**Tool calls (1)**
- `grep` of `cli_test.go` for the empty-pipe test and the `harness.run` definition, because my context had only the diff and the span from line 340.
