```yaml
lens: code-affected-set
outcome: reported
```

Task: run the code-affected-set lens over the diff and list, for each change, what it affects outside the lines it touches.

**Per-change affected set**

1. **`reedengine.ShowToggleOutcome` now passes `-d 10000`**, holding the message for ten seconds.
   - Callers:
     - `internal/slackcli/wiring.go:56`: covered.
     - `internal/orchcli/remotecontrol.go:203`, the RC button's `rc-on` verb: **not covered**. Its refusals now also stay up for ten seconds. Neither the diff's docs nor the summary name this, and the summary's blast radius leaves it out.
   - Tests:
     - `internal/reedengine/slacktoggle_test.go`: updated, covered.
     - `internal/orchcli/remotecontrol_test.go` checks no `display-message` argument shape (lines 151, 169, 204), so nothing there breaks.
   - Docs: `docs/research/speed/.../internal_reedengine.json` are measurement reports, which are exempt. No doc mentions `display-time`.
2. **`showToggle` moves from `wireProduction` to `productionCLI`.**
   - Constructions of `slackCLI`: `wiring.go:50` (production) and `cli_test.go:273` (harness, sets `showToggle`). Both set it. Covered.
   - The `slackCLI` struct comment at `cli.go:63` ("PersistentPreRunE fills every field its verb reads") was already imprecise before this diff, since `productionCLI` already set `clock`, `resolveLocation` and `wire`. Not a new staleness, so no finding.
3. **Toggle flag and verb names become constants**, read through `cmd.Flags()`.
   - The writer of the click line, `internal/reedengine/panebin.go:242-245`, still spells `"toggle"`, `--socket`, `--tmux` and `--client` as literals.
   - That literal is existing debt in a file the diff does not touch, and the constants are package-private to `slackcli`. Under PATTERN-code-conventions' debt clause, only a change that touches the literal must move it, so no finding.
   - Test `toggleArgs` at `toggle_test.go:99` uses literals, as test inputs should. Covered.
4. **The pre-run refusal of `toggle` now redraws the bar and logs at Warn.**
   - Readers of the pre-run path: `cli_test.go:1124` (`PreRunResolves…`) runs `status`, `relay` and `bar`, never `toggle`, so it is unaffected.
   - The new subtest covers the redraw.
   - Nothing reads or asserts the Warn message text (grep found no other site).
5. **Docs and help:** `docs/overview.md:954`, `internal/slackcli/doc.go` and the `toggle` long help are updated.
   - No other doc describes the toggle's refusal display; `docs/overview.md:397` and `:935` only list verbs. Covered.
   - Rule check: the PATTERN-code-conventions `doc.go` rule is met, since the new lines document the package.
6. **Commit messages and departures.**
   - `Root-Cause:` on 7daa081ab names the pre-run abort with no redraw and the 750 ms display with no log. The diff removes both. Covered.
   - There is no `Pattern-Departure:` or `Loosened-Check:` trailer, and none is needed.
   - No PATTERN changes are declared, so the "Declared PATTERN changes landed" check has nothing to verify.
   - No stencil changed.

**Findings**

- **MEDIUM, class `consistency`, location `_lym/landing/summary.md:50` (Merge danger, blast radius).**
  The blast radius reads only `slackcli`. But the diff's `reedengine.ShowToggleOutcome` change also changes the RC button: every `rc-on` refusal now holds for ten seconds. Nothing in the summary, docs or commit message names that change.
  - Evidence: `internal/orchcli/remotecontrol.go:203` calls `reedengine.ShowToggleOutcome`.
  - Evidence: `internal/reedengine/slacktoggle.go:24` passes `-d toggleMessageMillis` for every non-empty message.
  - Fix: name `orchcli`'s RC button in the blast radius and in the change description. The alternative is to keep the ten-second hold off the RC button's path.
- **LOW, class `consistency`, location `internal/reedengine/slacktoggle.go:11-17`.**
  The `toggleMessageMillis` comment and the `ShowToggleOutcome` doc both frame the hold as a Slack toggle's message. The same function also shows the RC button's refusal, so the comment describes less than the function's use.
  - Evidence: `internal/orchcli/remotecontrol.go:203`.

**Tool calls**
1. Grepped for `slackCLI` constructions, other readers of `ShowToggleOutcome` and `display-time`, toggle docs, and a tmux version floor. My context had no repository-wide search.
2. Grepped the `remotecontrol.go` hits, toggle-related doc lines and the `toggle` verb literals. The first search found a second caller my context lacked.
3. Read `internal/orchcli/remotecontrol.go` lines 170–215, the second caller's code and help, which my context lacked.
4. Grepped `remotecontrol_test.go` for assertions on the tmux call shape, to see whether the `-d` change breaks or is checked there.
