```yaml
lens: fix-affected-set
outcome: reported
```

My lens: fix-affected-set. I checked that each fix covers every reader in its finding's set.

Unchecked findings: none. I checked both F1 and the gate's `caller-uncovered/25` finding.

No findings.

- **F1 (`compareLockRetry`):** the constant has two readers. `internal/driftcli/cli.go:316` sits inside `compareCmd`, which card 22 deletes. `internal/driftcli/wiring.go:247` is the compare closure of `checkDeps`, which no card removes. The fix drops the Delete bullet, so the constant survives for the reader that stays. Its doc comment at `cli.go:35` still holds true for the check verb's compare. Every reader is covered.
- **Gate `caller-uncovered/25` (`testFiles`):** the function has seven references.
  - The five in `accepted.go` (lines 54, 72, 93, 118 and 127) are inside the acceptance and profile functions that card 25 deletes.
  - `capture.go:302` is inside `RunAcceptanceCapture`, which card 25 also deletes.
  - `check.go:274` is inside `skippedCases`, which `Check` calls for a `partial` scope. No plan card removes it; my earlier grep of `_lym/plan` for `skippedCases` and `ScopePartial` found nothing.

  The fix keeps `testFiles`, and the card's Intent now names `Check` as the remaining user. Every reference is covered.

Tool calls:
- One grep of `internal/` and `_lym/plan/` for `compareLockRetry` and `testFiles(`. It reran both findings' searches against the current tree to confirm the reader sets had not changed since the parent's grep.
