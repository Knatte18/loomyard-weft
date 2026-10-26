```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Findings I searched:
- **F1** (PATTERN entry for the stop chain): one search for "tackler, arbiter" and "rule table, tackler" across `PATTERN.md`, `pattern/`, `docs/`, `contracts/` and `internal`. The only prose statement of the chain is `PATTERN.md:34`, which the fix edited. The other hits are unrelated signatures and variable names.
- **F4** (Webster-only marking): one search for `JudgesFirst` and `arbiterJudgesFirst` across all Go files. The only production assignment is `internal/loomcli/drivewiring.go:159`, now routed through the named function and pinned by `TestArbiterJudgesFirst`. `internal/battencli/drive.go` never sets it, which is correct. The `sheddriver` doc comments already describe the seam.

Findings I left unchecked:
- **F2**: its premise is a refusal that no test reaches, not a set.
- **F3**: its premise is one test row, not a set.

Tool calls:
1. grep for the stop-chain prose across docs, contracts, PATTERN and `internal`: my context held only `PATTERN.md:34`.
2. grep for the `JudgesFirst` sites across all Go files: my context held only `drivewiring.go` and the `sheddriver` doc.
