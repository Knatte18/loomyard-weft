```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix-code-correctness lens on the F1–F3 change set. The only code change is the new assertion in `internal/dobbyengine/tidy_test.go`. F1 and F2 touch comments only.

Findings:

1. **LOW**: the new `h.state().Tidy.ReportPath` read in `tidy_test.go`.
   - The assertion sits right after `if tidy := h.state().Tidy; tidy == nil || ... { t.Errorf(...) }`, and that check uses `Errorf`, not `Fatalf`.
   - If the daemon never spawned and `Tidy` is nil, the test reports the first error and then panics with a nil dereference on the next line.
   - A failing test would still fail, but with a panic that hides the second message. The state is not wrong, and the test still catches the defect.
   - Evidence: `tidy_test.go` lines 126–133, as edited.

The new assertion does fail with the defect in place:
- `tidy.go:169` builds `filepath.Join(d.geom.TidyDir, now.UTC().Format("20060102T150405Z")+".md")`.
- Putting the `board-tidy-` prefix back, or writing outside `TidyDir`, changes the string, so the exact-equality check fails.
- The test passed in the `lyx gate test` run, so `h.now` equals the `now` the daemon used at the spawn step. The deadline assertion in the same row relies on that same clock.

The F1 and F2 comment rewrites change no behavior, and no caller or signature changed.

Further tool calls: none. My context already held every changed file and the gate output.
