```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set. I reran the searches for F1 (way-forward callers and texts) and F2–F4 (tests and wording sites). I did not check F5, because it is a single-site test comparison with no set premise.

1. **LOW** — `internal/websterengine/recordbatch.go:124` and `:226` (`glyphRejectedWayForward`, `deleteReferencedWayForward`), the siblings of the changed `createTargetWayForward`.
   - **Summary:** F1's dead-end premise holds for these two routes as well, and the fix leaves them as they were.
   - **Evidence:** both end in `rebaseline --card NN … then lym webster recover-batch %02d`.
     - `failCardNotDone` reaches them with the batch already terminal-failed, and `Rebaseline` skips a terminal batch (`rebaseline.go:374`, per the review).
     - After two counted recoveries, `RecoverSpawnOrAttach` therefore refuses these routes with `ErrRecoveryExhausted`.
     - Only `createTargetWayForward` gained the `exhausted` branch.
   - **Scope:** the review's F1 names only the Create route, so I rate this LOW, not MEDIUM.

2. **No finding** — F1's call sites.
   - `createTargetWayForward` has one caller, `recordbatch.go:233`, and it passes the new `exhausted` argument.
   - `exhaustedRecoveryWayForward` has two callers, `recordbatch.go:141` and `recoverbatch.go:510`, and both build the same text.
   - The `recoverbatch_test.go:710` assertion on `way forward: amend the card's Create target` still holds, because the new wording keeps that prefix.
   - `recordbatch_test.go:1619` and `:1673` assert substrings that are unchanged.

3. **No finding** — F2, F3 and F4.
   - The new test has a row for each branch of `createTargetWayForward` plus the delete, glyph and create precedence.
   - The `donecheck_test.go` rows cover Rename new side, no parameter list, another owner, another unit, and a Go target.
   - The only other `rename-not-done-new` test sites are `status_completeness_test.go:109` and `:145`, which don't concern signature mismatch.

Tool calls:
- Grep over the Go and markdown files for the way-forward helpers and the amend wording: to list every caller and text site of the changed helpers, which my context lacked repo-wide.
- Grep over the test files for signature-mismatch and Rename new-side text: to find every test site touching those findings.
