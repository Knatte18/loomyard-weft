```yaml
lens: fix-consistency
outcome: reported
```

No findings.

Task: check the F15 change set (the `NoPrimeClaimFlag` constant in `internal/beamcli/cli.go`, its use in `internal/machinecli/start.go`, and the `Lookup` call in `internal/beamcli/start_test.go`) for agreement with its surroundings.

- A repo-wide search of non-test files finds no remaining `no-prime-claim` literal or `noPrimeClaimFlag` use. The only literal left in a non-test file is the prose mention at `internal/machinecli/doc.go:33`.
- `internal/machinecli/start_test.go:301` and `:310` still hold the literal `--no-prime-claim` strings, as the review's `fix` key asked. They pin the child's command line.
- Both flag registrations, `BoolVar` and `MarkHidden`, now use the constant, and the `Fatalf` format has its `%s` and matching argument.

Tool calls:
1. Grep over the repo for `no-prime-claim` and `noPrimeClaimFlag` outside `_lym` and `.lym`: my context only held the hits from the fixer's earlier Go-only grep, which predated its edits and skipped non-Go files.
