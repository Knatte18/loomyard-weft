```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the F1 fix to the pending-findings resume subtest in `internal/landingshed/finalize_gate_test.go` (commit cc97e18b) for correctness.

no findings

Walkthrough of the changed branches:
- **`wantLand` row:** `fx.call` fails the test on a non-nil error. `requireLanded` then needs `outcome == Done`, exactly one entry in `fx.merger.calls`, and no attempt record left on disk. A Stuck outcome fails the `Done` check, so the row now fails with the original defect in place. The suite passes, so the real path lands.
- **`reviewStartErr` row:** `Call` must return an error, and `fx.merger.calls` must be empty. This matches the sibling subtest at `:601`.
- **Assertions after the branch:** the fixer-first session check, the `fix_review_path` value and the `FixReview` cleared check are unchanged and still valid for both rows. In the landed row `fx.attempts` finds no record, so the `found &&` guard skips the last check.
- **Callers:** only test code changed, with no signature changes.
- **Verification:** `lym gate test ./internal/landingshed` passed (663 targets, exit 0).

Tool calls made:
1. `grep` for `fx.call` and `requireLanded` in the `internal/landingshed` tests: my context did not show their bodies, and I needed them to confirm what each asserts.
