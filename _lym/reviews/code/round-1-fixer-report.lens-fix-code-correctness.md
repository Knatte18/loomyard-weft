```yaml
lens: fix-code-correctness
outcome: reported
```

No findings.

I traced these inputs by hand against the code in my context:
- **Closed key mapped to a disputed finding:** `hard` is closed with `finding: F2`, and the fixer reported only F1 fixed. `fixedIDs` holds only F1, so `hard` stays in `open`. The round escalates, and `AKeyClosedForAFindingTheFixerDisputedStaysOpen` would fail without the id check, because the round would end `Done`.
- **Partial close:** `hard` closed for F1 and `other` open gives `confirmed = {hard}` and `open = [other]`. The ledger is rewritten with only `hard` resolved, and the brief reads `(other)`, which can't match `(hard, other)`.
- **Parse refusals:**
  - An empty key, a repeated key, a `closed` entry with no finding and an unknown status each return an error. `verification` wraps it as "the verifier's file …", so the note reads "the check … failed".
  - A round-1 file on disk fails the round check and falls through to `runVerifier`. That archives the stale file and spawns once, so the row's `wantVerifierRuns: 1` fails if the refusal is removed.
- **Spawn error:** `Run` returns an error, `runVerifier` wraps it, and every key stays open with the failed-check note. The shedfake records the spec before calling `RunFn`, so the run count of 1 holds.
- **Ledger write failure:** the test creates a directory at `ledger.md.tmp`. `os.WriteFile` onto it fails with EISDIR, so the judge's ledger is untouched and `hard` stays open. The note is "could not be closed in the ledger". This works for root too, unlike a chmod.
- **Atomic rewrite:** a failed rename removes the temporary file. A failed write leaves any partial temporary file, which the next `WriteFile` overwrites, so this is harmless.
- **Changed signatures:** `parseVerification` and `verification` now return `map[string]string`. Their only callers are `verification` and `closeVerifiedBlocking`, both updated. `closed[key]` is read as `(id, ok)`.
- **Hard-cap archive:** `retired := append(judgeOutputs(...), verificationPath(...))` appends to a fresh slice, the same pattern as `rejudge`. `archiveStaleOutputs` is assumed to skip a missing file, since `rejudge` already passes the verification path when none exists.
- **Whole-ledger comparison:** `Prose: "prose"` and `Rounds: []int{2}` match what `parseLedger` returns. The gate passed.

Tool calls made: none. My context held every changed file and the review.
