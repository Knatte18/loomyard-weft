```yaml
lens: fix-code-correctness
outcome: reported
```

Line 464 runs unconditionally, so the non-idle assertion fails if that line is deleted. That settles the F3 check: no findings.
