```yaml
lens: fix-consistency
outcome: reported
```

No findings.

- `printNotRunTiers` calls `tierNames` from `scope.go`, whose body is the removed loop, so the line is unchanged.
- The new vet-row assertion expects the same line as the `--gate card` row at line 514.
- F3 is left as is, and `doc.go:10`, the two tests at 514 and 522 and the `notRunSuffix` comment agree with the shorter wording.

Further tool calls: none.
