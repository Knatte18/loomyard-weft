```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. I challenged F2, F3, F4, F5 and F6, which were fixed as proposed. F1 was disputed, so I did not check it.

**LOW** — `internal/gatecli/doc.go`, the gate.yaml example, tier0 command
- **Summary:** the F4 fix re-tagged the tier1 and tier2 examples but left the tier0 example `command: go test -json {targets} {args}` without `-tags tier0`.
- **Evidence:** `emptyTierFailure` (`tierrun.go:~190`) applies to every go-json tier, tier0 included, using `tag := tierName(run.number)`. Tier0 test files in this repo carry `//go:build tier0`, for example `emptytier_test.go:1` and `green_test.go:1`. That command compiles none of them, so the package ran 0 tests while holding tier0 tests. The same empty-tier failure the review's F4 cited for tier2 would fire. The fix made the tier1 and tier2 examples consistent but left this sibling stale.
- **Fix:** add `-tags tier0` to the tier0 example command.

**Challenged without a finding:**
- **F5 (`newestWholeAncestor`):**
  - **Tie order:** I built a three-commit case where A is an ancestor of B and C is unrelated to both. The loop still returns a tip, and when tips are unrelated it falls back to `Written`, as the review asked.
  - **Error as a tie:** an `isAncestor` error returns false and is treated as unrelated, so it falls to `Written` and stays sound.
  - **Cost:** the loop adds up to two extra git calls per candidate, at most about 40 per tier. That is acceptable.
- **F6 (pruning):**
  - **Other trees:** dropping other-tree entries that carry args breaks nothing, because `covered()` matches only the same tree and `newestWholeAncestor` only empty args.
  - **Same commit:** same-commit entries are no longer shadowed by the args entry, because `kept` is set only for empty-args entries.
- **F3 (empty-tier test rows):** the new rows match `isFilterFlag`'s trim and cut logic, including `-test.run=X` and `--run`.
- **F2 (`green_integration_test.go`):** the test has a positive control, and it fails if the `stillGreen` guard is removed, since the entry would then be recorded.

Tool calls: 1. I read `internal/gatecli/tierrun.go` lines 176–212, because my context did not show how the empty-tier tag is derived or whether it applies to tier0.
