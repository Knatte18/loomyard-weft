---
round: 1
focus:
  - "internal/slackcli/wiring.go `readHiddenTerminalLine`: after a signal the read goroutine is still blocked in `term.ReadPassword`; does the later restore or the process exit leave the terminal consistent, and can the deferred `Fprintln` or a late goroutine read misbehave?"
  - "internal/slackcli/wiring.go: the terminal state is read once with `term.GetState`; is it restored on the success and read-error paths as well as the signal path, or does it rely on `ReadPassword` restoring it?"
  - "internal/slackcli/cli.go `readToken`: on a terminal the line is read once and a non-terminal input is read to its end; does every path (terminal error, empty line, empty pipe, read error) give a refusal that stores nothing and reaches Slack with no empty token?"
  - "internal/slackcli/cli.go `login`: does the pane check still run before the prompt, and does a failed `readHiddenLine` on a terminal word its error correctly for a non-interrupt cause such as `GetState` failing?"
  - "internal/slackcli/cli_test.go: do the new tests cover the terminal branch, the piped branch, and the empty-token refusals through the public `login` verb, without duplicating an existing test's coverage? Is `readHiddenTerminalLine` itself covered or untestable here?"
  - "Scope against the board entry: terminal gets a prompt, one line, echo off, Enter ends it; piped input unchanged. Does the diff change anything else about piped behaviour, such as the empty-input message wording that a caller or doc relies on?"
  - "Doc and help text (`doc.go`, `login` Long, `tokenPrompt`, wiring.go header) against the diff: does each claim (prompt on standard error, one line, echo off, script can pipe) hold, and does any other doc or spec in the repo that describes `lym slack login` reading standard input to end of input, or Ctrl-D, still say so?"
  - "Commit message of e2ac5fdf and the repository's constraints document: does the change follow its rules for new dependencies use, comment conventions and docs that must land with the diff?"
---

The record holds only the board entry under `## Goal`, so the entry is the measuring stick.
`golang.org/x/term` is already in `go.mod`, so the diff adds no dependency.
