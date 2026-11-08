```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set. Per change, the set found and whether the diff leaves it right:**

- **New `readHiddenLine` seam on `slackCLI`**
  - The only constructors are `productionCLI` (`wiring.go`, sets it) and the test `newHarness` (`cli_test.go`, sets it).
  - A search for `slackCLI{` and `readHiddenLine` found no other construction.
  - Covered.
- **`login` reads through `readToken` instead of `io.ReadAll`**
  - The only caller is `login`, and the pane check still runs first.
  - Covered.
- **Piped empty-input message reworded**
  - The old text said to end the input with Ctrl-D, and the new text points to the terminal prompt.
  - A search of `*.md`, `*.go` and `*.yaml` outside `_lym` and `.lym` for "Ctrl-D", "Ctrl+D" and "end of input" found no other copy of the old wording.
  - No test or doc asserts the old text.
  - Covered.
- **`login` `Long` help, `doc.go` and the `wiring.go` header**
  - All three describe the new behaviour consistently: prompt on standard error, one line, echo off, and piped input read to its end.
  - `docs/overview.md` (lines 391, 866) only lists the verbs and says nothing about how login reads input.
  - No other doc or spec describes login reading standard input.
  - Covered.
- **New `golang.org/x/term` import in `slackcli`**
  - `go.mod` already requires it at v0.42.0, and `configcli`, `reedcli`, `clihelp` and `statuspane` import it, so the diff adds no dependency.
  - Searches of `pattern/` and `docs/` for `x/term`, `signal.Notify` and `slackcli` found no rule that restricts it.
  - `PATTERN-code-conventions` is met: the prompt is the named constant `tokenPrompt`.
  - `PATTERN-test-speed` is met: the new tests are subtests of `TestSlackCLI`, which is already `tiergate.Require(t, 0)` and parallel, and they spawn nothing.
  - Covered.
- **Declared PATTERN changes:** the focus directive declares none, so there is nothing to land.
- **Pattern-Departure trailers:** neither commit carries one and none is needed.
- **Commit messages:** they match the diff.

No findings.

**Tool calls (1):**
1. I ran a grep over `pattern`, `PATTERN.md` and `docs` for `slackcli`, `x/term` and `signal.Notify`, plus a grep for other `slackCLI{` constructors. My context held no search of the pattern files or of the seam's constructors.
