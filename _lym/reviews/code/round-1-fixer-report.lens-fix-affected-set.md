```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set self-check of the fix change set.

Findings left unchecked: none. I reran the searches for F3 (wording sites) and F2 (callers and the signature). F1 and F4 each name a single site, which my context already shows.

no findings

- **F3:** the sites the review named, and their twins in `internal/loomcli/landingdeps.go` and `_lym/landing/summary.md`, are narrowed. The remaining "own landing commit" mentions at `summary.md` lines 5, 46 and 48 describe the chain writer and the test fixtures, where the claim holds.
- **F2:** `writeLedgerLanding` has two call sites, one in `recordHeldLanding` and one in the test loop plus its surrounding test. Both pass the branch, and no old-signature caller remains.
- **Out of scope, noted only:** `internal/landingshed/deps.go:260` ("RecordLanding records commit as the run's landing commit") and the `recordLandedSize` doc say "landing commit" without "own". The fix did not touch either one.

Tool calls:
1. `grep` for landing-commit and landing-tip wording across `internal`, `docs` and the root `.md` files. My context had only the narrower "own landing commit" search, taken before the fix.
2. `grep` for callers of `writeLedgerLanding`, `recordHeldLanding`, `landLocally` and `RecordLanding`, plus the wording in `_lym/landing`. My context lacked the state after the fix across all of `internal`.
