```yaml
lens: fix-proposal
outcome: reported
```

Findings:

1. **MEDIUM.** Changed text: card 14, "`Refused` is the error of stdout's last non-empty line when that line parses as a JSON object with `ok` false and a non-empty `error`, which only a refusal prints". The rule no longer checks whether a tier ran, so it contradicts card 6:22, which defines `Refused` as "the refusal text when the call refused before running any tier".
   - Concrete case: tier 1 of `harden-live` runs and fails. A later step then refuses with `output.Err(e.out, …)`: a guard step error (`internal/gatecli/run.go:590`) or a tier result's refusal (`run.go:668`). The doc also allows "a refusal after a start" (`internal/gatecli/doc.go:1449`).
   - Under the new rule, `hardenLiveRun` returns `Refused`, and card 6 step 1 turns it into an error with no report. A round whose test failed is then handled as a hub config fault: no red mark and no re-prompt of the fixer.
   - The old card 14 text carried "and no tier run". The proposal dropped it.
   - Possible fix: set `Refused` only when stderr carries no started line, which `gateslot.ParseStartedLines` can check. Alternatively, restate card 6:22 to match the new rule.

2. **LOW.** Changed text: card 14, "so build output on stdout never matches", and "the summary lines, the started lines and the failure report go to stderr". A tier command's own output goes to stdout, not only build output: `runTier` writes it through `io.MultiWriter(out, &amp;logBuffer)` (`internal/gatecli/tierrun.go:82`).
   - So a test binary's output lands on stdout. A failing or timed-out test whose last printed line is a JSON `{"ok":false,"error":…}` line, as a lym CLI test can print, would be read as `Refused`.
   - "never matches" holds for build output only. The claim should say that the tier's test output also reaches stdout, and it should not assert that no output other than a refusal can match.

Tool calls:
- One grep of `internal/gatecli/tierrun.go` and `run.go`, to find where a tier command's stdout goes and where refusals halt a call after a tier ran. My context had not shown `runTier` or those halt sites.
