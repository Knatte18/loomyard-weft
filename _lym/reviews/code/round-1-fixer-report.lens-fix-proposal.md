```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the proposals the fixer carried out as written (F2, F3, F4, F5, F7, F8, F9, F10, F11 and F13).

**Finding 1 (MEDIUM), F13 hoist and the tripwire in `completionsignal_enforcement_test.go`.**
- Changed text: `retryAPIError` now ends with `return run.endOnAPIError(outcome)`. The audit entry `"retryAPIError [OutcomeAPIError]": 1` and its comment line were deleted to get the gate green.
- Summary: the F13 proposal made the api-error end invisible to the Completion Signal tripwire, and the fixer removed the audit instead of keeping it.
- Evidence:
  - The gate's first run failed with `retryAPIError [OutcomeAPIError]: found 0, audited 1`.
  - The scan keys on a literal negative outcome in a return statement. `outcome` is a variable, so any later change to that return reads as zero sites.
  - A future edit that passes `OutcomeAPIError` straight to `endOnAPIError`, skipping `classifyDeadlineExpiry`, would pass the tripwire.
  - Keeping the `return run.endOnAPIError(run.classifyDeadlineExpiry(OutcomeAPIError))` shape would have kept the guard. The cost is a second file read, which was the reviewer's NIT.
- The F9 edit to `PATTERN-shuttle-provider-seam.md` names `retryAPIError` as a helper that reaches `allOutputFilesExist`. No test pins that any more, so the pattern text and the test disagree again, which is what F9 was meant to remove.

**Finding 2 (LOW), F10 wording in `docs/overview.md`.**
- Changed text: "Any other turn end without every output file never ends a run".
- Summary: the sentence reads as if only an API-error turn end of an opted-in spec ends a run early.
- Evidence: the `Wait` deadline, a dead pane and a stop also end a run. The original sentence already listed them ("until the outputs exist, the pane dies or the deadline passes"), so the added "Any other" is accurate but now implies a closed set of exceptions. This is minor.

**Finding 3 (LOW), F8, `retryAPIError` doc comment and the `Fail` call.**
- Changed text: "logged once per episode".
- Summary: the doc comment is correct only if `Episode.Fail` logs the first failure and demotes repeats.
- Evidence: I did not read `internal/logger/episode.go`'s level policy. The review asked for Info on the first failure and Debug on repeats. `send.go` describes this API as "one Warn per episode", so the first failure may now log at Warn rather than the Info the old line used. That is a behaviour change nothing asserts.

Checked with no breakage found:
- F2: adding api-error to `unfinished` is consistent with `isTerminalOutcome` and `loopAgreesWithState`. The new carry-over row passes.
- F3: the `orch-role.md` list edit matches the sentence it extends.
- F4: `strings.Contains(envelope.Question, "")` is true for every other row, so the unconditional assertion is safe.
- F5 and F11: the rows pass in the gate. The covered row fails if the `run.covered` check is dropped, because `lifted` would stay false.
- F7: the fallback text reaches both settlers through `Result.APIError`, so neither the reason nor the question ends in a colon.
- F9: the pattern edit is correct apart from Finding 1.

Tool calls: none. My context held every changed file, the gate output and the rubric.
