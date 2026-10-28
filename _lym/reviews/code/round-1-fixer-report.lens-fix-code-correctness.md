```yaml
lens: fix-code-correctness
outcome: reported
```

Task: fix-code-correctness lens over the F1–F5 change set. Each branch below was traced by hand, not run.

No findings.

- **Gate regex (F3).** Take `lym gate test 2>&1 --card 6-prime-weft`. The pattern consumes `2>&1` through the first alternative, and the segment then runs on to `weft`, so the card ID is blanked. The old class stopped at `&`, leaving `1 --card 6-prime-weft` for the bare-word scan, so the new row fails with the defect in place.
- **Segment bounds.** `&&`, `;`, `|`, `||` and a real newline still end the segment, because none of them is `[<>]&` or `&>` and the class excludes `&`. The existing `... --card X 2>&1 | tail -40` row still holds: the redirect is consumed and the segment ends at ` |`.
- **F1 rows.**
  - `echo lym gate other-weft` has no command-position match, so `other-weft` is scanned and the row is true.
  - Each separator row leaves its `other-weft` outside the segment, so it is true.
  - Each row would fail if the segment ran past that separator or if blanking applied outside command position.
- **F2 rows.**
  - The dirty-tree refusal text lacks `stepResetToStart`, so `acceptAuditWayForward` appends `; otherwise 1) ... reset --to start; 2) ... run`. That matches the row's expected suffix.
  - The non-descendant refusal already carries `wayForwardSteps(stepResetToStart, ...)`, so nothing is appended. The row asserts that.
  - The `headErr` row relies on `acceptAuditWayForward` returning the non-`ErrAuditNotAcceptable` error as is. If it swallowed the error, `ErrRecoveryNeedsFresh` would be returned and the row would fail.
  - The gate run passed all 2807 tier0 targets, including these rows.
- **F5 row.** The nil guard is `!ok || readOnly == nil || !readOnly(cmd)`. Without it, `readOnly(cmd)` calls a nil func and panics, so the row fails with the guard dropped. The entry is `lym fabric add x`, which is mutating, so the row doesn't show that a nil classifier rejects a read-only command. That is only a coverage nuance, not a defect.
- **F4 row.** The Verify `lym gate test --card 1-prime-weft` is blanked by the gate rule, and slug `alpha` raises no card-ID finding. The row passes through the gate exemption added earlier in this branch, not through the F4 text edits, which are comment and message wording only.
- **Callers.** No signature changed, so no caller is affected.

Further tool calls: none.
