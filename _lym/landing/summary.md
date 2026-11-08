# lym slack login prompts for the token on a terminal and hides it

`lym slack login` read the token with `io.ReadAll` on standard input.
On a terminal it printed nothing and waited for end of input (Ctrl-D), and the pasted token showed on screen.

## What changes

- On a terminal, `login` prints `Paste the Slack app's Bot User OAuth Token (xoxb-…) and press Enter: ` to standard error, reads one line with echo off and goes on at Enter.
- Any other standard input is read to its end as before, so `echo xoxb-… | lym slack login` keeps working.
- An empty line at the prompt is refused with its own way forward, and the piped refusal no longer tells the operator to end the input with Ctrl-D.

The terminal read is a new `slackCLI` seam, `readHiddenLine`.
Its production value, `readHiddenTerminalLine` in `internal/slackcli/wiring.go`, uses `golang.org/x/term`, which the module already depends on.

## Design decisions

- The prompt goes to `os.Stderr`, not the command's output, so the JSON envelope or readable summary on standard output stays clean, even when standard output is redirected.
- An interrupt or SIGTERM during the hidden read restores the terminal state saved before the read and ends `login` with a refusal.
  `term.ReadPassword` keeps signals enabled, so without this a Ctrl-C at the prompt would leave the operator's shell with echo off.
- The production read is not tested directly, since driving it needs a pseudo-terminal.
  The tests fake the seam and cover the prompt, the trimmed line being stored, the empty-line and interrupted-read refusals, and that piped input prints no prompt.
