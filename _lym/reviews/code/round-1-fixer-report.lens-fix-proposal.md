```yaml
lens: fix-proposal
outcome: reported
```

No findings on F1, F3, F4 or F5. F1 has one gap: the way forward leaves out a step. F2 has one case, covered below.

**Finding 1 — MEDIUM, F1 route for a non-beam run is missing the release step**
- Changed text: `errorEnvelopeBodyReason` in `internal/loomshed/recordwrite.go`, which ends "... `lym board upsert --body-file <path>`, then `lym batten run <slug>`".
- Summary: after `cancel --remove`, `lym batten run <slug>` is refused until the slug is released with `lym board set-status '{"slug":"<slug>","status":null}'`. The reason omits that step, so it still does not make progress.
- Evidence:
  - `internal/battencli/startguard.go:473` words the cancelled-run refusal as "run `lym batten cancel %s --remove` ..., then %s [release command] and `lym batten run %s`".
  - `internal/battencli/arm.go:868` names the same pair, "run %s and `lym batten run %s` for a fresh start".
  - `clearRunClaim` in `internal/battencli/cancelcmd.go:597` sets the task to draft, and the release is the null status.
- This matters because the reviewer's `fix` text told the fixer to confirm the start verb in `cancelcmd.go` and the start verb's own help. The fix was applied as proposed without that check.
- Suggested change: insert `lym board set-status '{"slug":"<slug>","status":null}'` between the upsert and `lym batten run <slug>`, and add it to the test's want list.

**Finding 2 — LOW, F2 fence toggling can hide an envelope**
- Changed text: the fence toggle in `ErrorEnvelopeLine` in `internal/output/output.go`.
- Summary: the toggle does not pair an opener with its own closer, and an unclosed fence hides everything after it.
- Concrete cases:
  - A body that opens a fence and never closes it, followed by `## From x` and an unfenced envelope, returns `"", false`. Such an unclosed-fence body is a case the `UnclosedInnerFence` row in `recordwrite_test.go` already treats as plausible. Record-Write then stores the error text as the Goal.
  - A `~~~` line inside a ``` block, or a ```` ```go ```` line inside a four-backtick block, flips the state wrongly.
- Evidence: the toggle is `if HasPrefix("```") || HasPrefix("~~~") { inFence = !inFence }`, which tracks neither the opener's character nor its length.
- Suggested change: remember the opening fence's character and length, and close only on a line that starts with the same character at least as long. Optionally treat a fence still open at the end of the text as not hiding the lines after it.

**Calls made (1 of 6)**
- One `grep`/`sed` over `internal/battencli/startguard.go` and `arm.go`: I needed the release command and what a restart after `cancel --remove` requires, and my context did not hold those.
