# slackcli: prompt for the Slack token on a terminal with echo off

`lym slack login` run in a terminal printed nothing and seemed to hang.
It read the token with `io.ReadAll` on standard input, so it waited for end of input (Ctrl+D) with no prompt,
and the pasted token showed on screen.

On a terminal, `login` now prints a prompt naming the Bot User OAuth Token to paste, reads one line with echo off,
and goes on at Enter.
Input that is not a terminal is read to its end as before, so a script can still pipe the token in.

## Design decisions

- The terminal read is a new seam on `slackCLI`, `readHiddenLine`, filled in `wiring.go` by `readHiddenTerminalLine`.
  It uses `golang.org/x/term` and reports `onTerminal`, so `login` takes the one-line path on a terminal and the read-to-end path otherwise.
  Tests drive the seam through the harness; the production function is tested for input that is not a terminal.
- The prompt goes to standard error, so it never mixes into the verb's result on standard output.
- An interrupt or termination signal during the hidden read restores the terminal's echo and ends the read with an error,
  so the operator's shell is not left with echo off.
  A read that fails this way is refused with a message that nothing was stored and to run `login` again.
- An empty token is refused on both paths, before any call to Slack.
  The refusal messages name the way forward for the path taken: paste at the prompt, or pipe the token in.
- The `login` help text and the package doc in `doc.go` describe the two input paths.

## To check by hand

The terminal path needs a real terminal, which the tests cannot provide.

- Run `lym slack login` in a terminal: the prompt shows, the pasted token does not echo, and Enter continues to the Slack check.
- Press Ctrl+C at the prompt: the command ends with an error, nothing is stored, and typing in the shell afterwards echoes.
- Run `echo xoxb-… | lym slack login`: no prompt is printed and the token is stored as before.
