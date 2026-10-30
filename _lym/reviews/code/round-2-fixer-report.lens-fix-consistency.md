```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix-consistency lens over the F1 and F2 change set.

**Findings**

1. LOW, `internal/beamcli/dobby_test.go:3`. The file header is stale after the fix.
   - The header says the file "pins the pieces of dobby's hub wiring that need no hub: the run-process registration of the dobby, the error text read from what a pass printed and the machine-named landings seam."
   - The fix appended `TestDurationFromCount`, which pins the recipe count-to-duration fallback behind the probe interval and the quiet window.
   - That is a fourth concern the header does not name, so a reader choosing a file from the header would not find it.
   - The prose skill's rule on file headers says the header plus filename should let a reader decide whether it is the right file.

No other findings:
- `runShedDuration` doc comment (`internal/beamcli/dobbydeps.go:227`) still matches the behaviour.
- Callers of `runShedDuration` and its signature are unchanged.
- It now returns on the first Run-Shed row instead of continuing to later rows; the batten recipe has a single such row, so nothing observable changes.
- `runnotice_test.go` header (line 3) still covers the restart rows added by the fix.
- The new `restartAfter` field is documented like its sibling `writeAfter`.
- The docs the review cites (`entries_batten.go`, battenshed) still agree, as the fix did not change that behaviour.

Tool calls made: none. My context already held the final text of every changed file.
