```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens, running each changed function or branch with a concrete input.

**List**
- `readHiddenTerminalLine`, `in` not an `*os.File` (cobra `SetIn`, a pipe): returns `"", false, nil` and reads nothing. Correct.
- `readHiddenTerminalLine`, stdin is a terminal and `term.GetState` fails: returns `"", true, err` before the prompt prints. The state is correct, but the wording in `readToken` is slightly off (finding F1).
- `readHiddenTerminalLine`, Enter pressed: `ReadPassword` restores the terminal itself, and the deferred `Fprintln` supplies the newline. The defer order is `Fprintln` first, then `signal.Stop`. Correct.
- `readHiddenTerminalLine`, Ctrl-D on an empty line: `ReadPassword` returns `io.EOF`. `readToken` refuses with "nothing was stored" and nothing is stored. Correct.
- `readHiddenTerminalLine`, Ctrl-C or SIGTERM: `ReadPassword` keeps ISIG on (x/term clears only ECHO), so the signal is delivered to the channel.
  - The `select` takes the signal case and `term.Restore` uses the state saved before the read, so echo returns.
  - The read goroutine is left blocked on a buffered channel, and the process exits right after, so it leaks nothing. If it ever returned late, its own restore would apply the same saved state.
  - Correct. If both channels are ready, the `select` picks one at random, and either outcome is consistent.
- `readHiddenTerminalLine`, `Restore` error: discarded with `_ =`. Acceptable, since the next step is exit.
- `readToken`, terminal with a whitespace-only line: refusal "no token entered at the prompt". Nothing is stored and Slack is not called. Correct.
- `readToken`, terminal with `" xoxb-typed "`: trimmed to `xoxb-typed`. The piped input is not read. Correct.
- `readToken`, piped empty input: refusal "no token on standard input", with the Ctrl-D advice removed. Correct.
- `readToken`, piped read error: returns the wrapped `io.ReadAll` error. Correct.
- `readToken`, `readHiddenLine` returns an error with `onTerminal` false: cannot happen in production, and the fake never does it. No finding.
- `login`, pane check: still runs before `readToken`, so the prompt never appears in a refused pane. Correct.

**Findings**
- F1, LOW, `design`, `internal/slackcli/cli.go:241`: the terminal-read error tells the operator to "paste the token at the prompt" for every cause.
  - When `term.GetState` fails (`wiring.go:61`), no prompt was printed, so the advice points at nothing.
  - The same text covers EOF and an interrupt, where it is right.
  - Fix: word the way forward without "at the prompt", or have the `GetState` failure return its own message.

No other finding.

**Tool calls:** none. The manifest already held every definition and call site this lens needed.
