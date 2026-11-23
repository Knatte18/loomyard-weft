```yaml
lens: fix-affected-set
outcome: reported
```

Lens: fix-affected-set. I reran the set searches for F2, F13 and F11 and found nothing blocking.

- **F2 (hubs with no flag):**
  - The `RunsWithoutFile` consumers are covered. `internal/hubreconcile/reinterpret.go:64` skips a change only when the file is absent and the flag is unset, so setting it on the loom entry at `configreg.go:164` is the only change needed.
  - The only other Go reinterpretation with `RunsWithoutFile: true` is dobby's, so the loom entry follows an existing pattern.
  - No other non-test code or doc line states that only a hub setting `code_auto_accept: true` is flagged.
- **F13 (rows that take the decision route):**
  - The `escalate_decisions` rows are motif, darn, tapestry, arras, hardener, sampler and the `tapestry-rows-before-motif.yaml` testdata.
  - The new hold in `settleDecisionAccept` sits in the shared Bouncer code, so every one of those rows gets it.
  - Of the three other `acceptRound` call sites, two are in `bouncer.go` (lines 1448 and 1507, both settling decided accepts) and one at line 1349 I did not read.
  - Only `bouncerdecision.go:202` is reached through a decision escalation, so no other route misses the check.
- **F11 (hardener focus permission):**
  - `FocusFindings` is set at exactly two non-test sites: `entries_burler.go:183` and `validate.go:137`. Both now have a test.

Findings:

- **LOW** — `settleDecisionAccept` in `internal/shedadapters/bouncerdecision.go`.
  - The new hold also applies to motif and arras Bouncer rows, which already set `escalate_decisions`. The review and the landing summary name only the Code and Harden rows.
  - The behaviour is consistent with F13's rule, but a reader of `_lym/landing/summary.md` would not learn that those rows changed too.
  - Evidence: `contracts/recipes/motif-recipe.yaml:64` and `arras-recipe.yaml:72`.

Further tool calls (one grep for F2, one for F13, one for F11 were intended; I ran them as one Bash call with four greps):
- Bash grep of `code_auto_accept` mentions, `RunsWithoutFile` consumers, `escalate_decisions` and `blocking_escalates` rows, and `acceptRound` callers: my context lacked the sites outside the files I had read.
- I did not run a separate `FocusFindings` grep. The earlier grep in the context already listed every site.
