```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fixer's change set for stale siblings and twins left unchanged (fix-consistency lens).

Findings:

1. LOW — `internal/driftengine/steprun.go:160-161`, the doc comment on `outputTailMaxRunes`.
   - The comment still reads "bounds the quoted output". `commandReport` now cuts that output from its start with a leading ellipsis, while `readChildReport` (`spawn.go:186`) keeps cutting from the end with `boundOneLine`.
   - The two cut directions are deliberate, per the review.
   - Nothing at the constant says which direction the bound applies in.
   - This is minor: the new function docs state the direction.

2. LOW — `internal/driftengine/spawn.go:186`, the twin of the changed tail-quoting.
   - `readChildReport` still quotes a child's error text cut from its end, so a long child error loses its last words.
   - The review limits F3 to `commandReport` and says `readChildReport` keeps cutting from the end.
   - The fix follows that. I flag it only as a deliberate asymmetry.

3. No finding on the other siblings I checked.
   - `boundOneLine` is still used, so it is not dead code.
   - No doc or help text mentions the first-400-runes cut.
   - `TestRunCapture`'s header and `orchcli/doc.go` agree with the changes.

Tool calls used:
- One `grep` over Go, md and yaml files for `boundOneLine`, `outputTailMaxRunes`, `outputTailLines` and phrases like "last output" or "cut to". My context did not show where else these were used or documented.
