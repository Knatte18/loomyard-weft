```yaml
lens: fix-code-correctness
outcome: reported
```

Task: fix-code-correctness check of the F1, F2, F3 and F5 code changes.

no findings

- **`start.go` call site:** a nil result carries on. A wrapped `errStatusStrandNotReplaced` logs a Warn and carries on, as before the fix. Any other error releases the bootstrap lock and refuses.
- **`ensureStatusStrand`:** a `ReplaceStrand` failure returns before the sidecar write, as before. The `%w … %w` wrap keeps both the sentinel and the cause.
- **Callers:** the parent's grep shows only `start.go` and the `wiring.go` closure. The closure passes the error to `restoreStatusPaneBeforeWake`, which logs it and sets `status_pane`.
- **Smoke test:** the parent's temporary `return nil` in the restore closure made the new assertion fail, and the tier 2 and `code-burler-fix` gate runs passed with the restore in place. The refactored step subtest asserts the same condition as before.
- **`resume` Short:** the text changes no behaviour.

Tool calls made: none.
