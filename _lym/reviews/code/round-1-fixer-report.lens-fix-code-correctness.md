```yaml
lens: fix-code-correctness
outcome: reported
```

No findings.

- `printNotRunTiers` with numbers [1, 2] prints the same line as the old loop; the empty guard is untouched, `strings` is still used by `printFailedRuns`, and the caller at `run.go:174` still holds.
- The new vet-row assertion fails with the `!run.vetStep` exclusion dropped, since the line would then start `gate test: tier2`.

Tool calls made: none; my context already held everything needed.
