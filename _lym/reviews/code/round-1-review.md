---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/slackcli/cli.go:241
    origin: lens:code-correctness
    summary: the terminal-read error tells the operator to paste the token at the prompt even when no prompt was printed
    fix: In readToken, reword the way forward of the terminal-read error so it does not mention the prompt, for example "run `lym slack login` again", so a term.GetState failure in readHiddenTerminalLine (wiring.go) before any prompt prints is not misdirected; keep "nothing was stored" and update the two substrings the OnATerminalAnInterruptedReadIsRefusedNamingLogin row asserts only if they change.
  - id: F2
    severity: LOW
    class: design
    location: internal/slackcli/wiring.go:52
    origin: lens:code-tests
    summary: readHiddenTerminalLine has no test, so its no-terminal result for piped input and its terminal handling are unpinned
    fix: Expose readHiddenTerminalLine through internal/slackcli/export_test.go and add rows to the existing TestSlackCLI or a new tier0 table test that pass a strings.Reader and a non-terminal *os.File (an os.Pipe read end) and expect ("", false, nil) with nothing written to the input; the terminal and signal paths need a pseudo-terminal and stay untested, which the commit's change description already states.
---

### [LOW] Terminal-read refusal misdirects when no prompt was printed

**Location:** internal/slackcli/cli.go:241
**Issue:** `readToken` words every `readHiddenLine` error as "run `lym slack login` again and paste the token at the prompt".
When `term.GetState` fails (`wiring.go:61`), `readHiddenTerminalLine` returns before it prints the prompt, so the advice names a prompt the operator never saw.
EOF and an interrupt are worded correctly.

### [LOW] The production terminal read has no test

**Location:** internal/slackcli/wiring.go:52
**Issue:** The harness replaces the `readHiddenLine` seam, so `readHiddenTerminalLine` runs in no test.
The terminal and signal paths need a pseudo-terminal, and the repository has no pty dependency, so those stay untested.
The branch that returns `onTerminal == false` for input that is not a terminal is what keeps piped input working in production, and a plain `os.Pipe` pins it without a pseudo-terminal.

## Rejected

- Empty-pipe refusal test does not assert the new way-forward text (lens code-tests, NIT): the changed text is message wording, the refusal itself is pinned, and no rule asks the test to pin the wording.
- Prompt goes to standard error, so it is invisible when standard error is redirected while standard input is a terminal (handler): the board entry asks for a prompt and gives no stream, standard error keeps the structured output on standard output clean, and the case is rare.
- Signal path leaves the read goroutine blocked in `term.ReadPassword` (focus directive): the process exits straight after, the buffered channel never blocks the goroutine, and a late restore applies the same saved state, so the terminal stays consistent.
- Terminal is restored only through `ReadPassword` on success and read-error paths (focus directive): `ReadPassword` restores its own state on return, and the explicit restore covers only the signal path where the goroutine is still blocked.
- Piped behaviour changed beyond the board entry (focus directive): only the empty-input refusal wording changed, to drop the Ctrl-D advice, and no doc, test or caller relies on the old wording.
- Docs that describe `lym slack login` reading to end of input (focus directive): a search of the repository outside `_lym` and `.lym` found none besides the three the diff updates.
