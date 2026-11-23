```yaml
lens: fix-proposal
outcome: reported
```

Findings, by reviewer proposal:

**F1 — MEDIUM** — proposal: extract `parentReviewPRAskFunc` and drive it from `TestPRReviewAsker_AskReachesTheParentSignedAsTheCaller`. The changed text is `preRun`'s `askPR: parentReviewPRAskFunc(target, strand)` and the test's `parentReviewPRAskFunc(pair, tt.claim)(rec, "head-1")`.
- The proposal does not resolve the finding's premise. The test calls the function with a claim it supplies itself.
- Change `preRun` to `parentReviewPRAskFunc(target, "")` and every test still passes. That is the regression issue #200 describes.
- The proposal's own comment concedes the gap by naming only the function. `preRun` is still reached only by `cmd/lym.TestCLITree_EveryCommand`.
- A sound alternative is a test that runs `preRun` with `LYM_STRAND_NAME` set through `t.Setenv`, which drops `t.Parallel` for that test. It would then check that `s.deps.strandName` and the claim the seam signs with are the same value. Or `preRun` could derive both from one field.

**F2 — LOW** — proposal: keep `two-way` with a justification written beside it. The changed text is the new paragraph in `_lym/landing/summary.md#Merge danger`, "The call stays two-way: the record is not tracked, the next run rewrites it, and the older binary's reading ends the ask, not corrupts it."
- "The next run rewrites it" has no evidence. The ask record is per run, at the anchor path, and a resumed or long-lived run keeps it.
- The same paragraph says an older Drift-Verify stores the rulings without the parent's answer. That behaviour does not self-heal on rollback.
- The justification can be false for exactly the rollback case it is meant to cover. Either cut "the next run rewrites it" or cite the code that rewrites the record.
- My grep of `asks.json` in the Go sources found no code that clears or rewrites the file (only unrelated `tasks.json` hits). The ask-record constant may be named differently, so this is not conclusive.

**F3** — no findings.
- The new wording ("refused or failed to deliver", "names why") agrees with `ParentAskUndelivered`'s doc and with every seat ending any `Post` error undelivered.
- The twin in the `ask` verb's Long text was changed too.

Tool calls (1 of 6):
- `grep -rn "asks.json" --include=*.go internal`: to check whether any code rewrites the ask record, which my context lacked.
