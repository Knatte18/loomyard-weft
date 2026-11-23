```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals for F1–F4 that the fixer carried out as proposed.

No findings.

- **F1 (darn-recipe.yaml comment):** The new text matches `internal/shedrecipe/doc.go:65-66`. The hold-on-no-test sentence is kept, and the exit-5 hold wording is gone. I found no case where it is wrong.
- **F2 (overview.md:1113):** The added clause ("a hub whose scoper is not go passes the gate") agrees with the recipe and `doc.go`. The `:997` exit-5 mention, which the review rejected, is untouched and still correct.
- **F3 (new crash row):**
  - The row's command cats `gateslot.RunnerCrashLine(...)` and exits 5, so `verifytree.Verify` takes the `RunnerCrash` branch at `verify.go:440`.
  - With the `!res.RunnerCrash` conjunct deleted at `redfirstgate.go:216`, the gate would pass, so the row's `wantTerminal` would fail. The row therefore kills that mutation.
  - `redFirstFailure` sets `terminal` at `redfirstgate.go:256-258` with text starting "The red-first check's test runner crashed", which matches the row's `wantFindings` substring.
  - The log has no timeout line, so `TimedOut` stays false and the crash case is the one under test.
  - The `crashLog` file is created once, outside the subtests, and only read inside them. That is parallel-safe.
- **F4 (summary.md):** The go-hub qualifier is correct, because the non-go verb exits 5 before it looks for tests (`gatecli/redfirst.go:128-131`, per the review). The widened blast radius names the verb message and the writer stencil, as the review asked. That radius is also the surface of the diff itself.

No further tool calls were made. The context already held every file the lens needed.
