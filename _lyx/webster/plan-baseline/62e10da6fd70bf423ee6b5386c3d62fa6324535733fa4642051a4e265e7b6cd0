# Card 8 — kit-config-text

**Create:**
- `internal/testkit/tmuxkit#ConfigText`

**Edit:**
- `internal/testkit/tmuxkit#serverConfig`

**Intent:**
Decision D7, the kit half: the kit's tmux config text becomes an export, so a `reedengine` test can pin the shell lines reed's own session pins share with the kit's server, which the kit cannot pin itself since it cannot import `reedengine`.

`ConfigText` returns the kit's tmux config for a shell, the four lines `serverConfig` writes today: `default-shell` and `default-command` set to the shell, `exit-empty off`, and the `@lyx_test_server` marker.
`serverConfig` keeps its shell choice, `$SHELL` else `sh`, and its write-once behavior, and writes `ConfigText(shell)`; the file it writes is byte-identical to today's.
`ConfigText`'s doc comment states what each line is for: the two shell lines start panes without the login flag, `exit-empty off` keeps an empty server alive, and the marker tells reed's stale-holder probe to leave the server alone.
The package doc's sentence on reed's own server is card 9's, since what it states changes with that card.

Test: none new; the kit's existing `TestSocket_StartsHermeticNonLoginServer` pins the server the unchanged config starts, and card 9's `KitConfigSharesReedsShellPins` step is the export's consumer and its pin.
`internal/testkit/tmuxkit`'s budget row stays.

**ImpactSummary:** One unexported writer in the kit now formats through the export; the config file every kit server reads is unchanged, so no test package that runs through the kit sees a difference.
