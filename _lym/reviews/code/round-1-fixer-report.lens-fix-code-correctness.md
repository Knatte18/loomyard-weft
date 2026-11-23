```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix-code-correctness lens over the F1/F3 code change (`parentReviewPRAskFunc`, the reworked integration test, the Long text).

**Findings**

1. **MEDIUM** — `internal/loomcli/prreviewask_integration_test.go`, `TestPRReviewAsker_AskReachesTheParentSignedAsTheCaller`. The test still passes with the #200 defect in place.
   - The test calls `parentReviewPRAskFunc(pair, tt.claim)` directly with the claim it chooses itself. It never goes through `preRun`.
   - The defect is `preRun` handing the function `""` instead of the session's strand: `askPR: parentReviewPRAskFunc(target, strand)` becomes `parentReviewPRAskFunc(target, "")`.
   - With that change, every test row still passes. The "session" row supplies `session` as its own argument, and nothing else reaches `preRun`'s argument.
   - The extraction moves the gap from the closure to the `preRun` call site, so F1's stated premise ("replacing `strand` with an empty string brings issue #200 back and every test still passes") still holds.
   - The new test does catch a regression inside the function, such as `strandClaim` being ignored. It does not catch `preRun` passing the wrong claim.
   - The review's own fix has this same limit. A real pin needs a test that drives `preRun` or `parent-review ask` end to end with `LYM_STRAND_NAME` set, which means `t.Setenv` and a non-parallel test, as the review's fix allows.

No other findings:
- **`parentReviewPRAskFunc`:** it is the old closure body verbatim, so behaviour is unchanged. Failure of `prReviewAsker` (a config load error) returns a zero `ParentAsk` and the error. A failure of `asker.Ask` propagates. `parentReviewAskVerb` refuses on either, as before. The only caller is `preRun`, and the signature matches `deps.askPR`.
- **Deadline bound:** `Ask` is given `time.Now()` inside the closure, so the deadline is the call time plus 60 minutes. That value lies in `[before+60m, after+60m]`, and the `Before`/`After` comparisons are monotonic-safe. The old check used `Equal` against the same unrecorded value, so the ask is not truncated in a way the new bound would miss.
- **Test compile:** `prReviewAsker` is no longer referenced in the test. `pullrequest`, `time` and `burlerengine` are still used, and the gate compiled and passed at tier 0 and tier 1.
- **Long text and stencil:** they changed wording only, with no code path touched.

Tool calls made: none. My context already held every changed file.
