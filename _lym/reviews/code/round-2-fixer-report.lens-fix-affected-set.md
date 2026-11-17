```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

F15 is the only finding whose premise is a set (the sites that spell the flag name). I searched all `.go` and `.md` files outside `_lym` and `.lym` for `no-prime-claim`, `NoPrimeClaimFlag` and `noPrimeClaimFlag`.

- **Production literals:** the constant is declared once, at `internal/beamcli/cli.go:34`. It is used in `BoolVar` and `MarkHidden` (`cli.go:506-507`) and in machinecli's child argument (`internal/machinecli/start.go:312`). No other production spelling of the flag remains, and no `noPrimeClaimFlag` is left behind.
- **Tests:** `internal/beamcli/start_test.go:22,24` use the constant. The literal expected strings at `internal/machinecli/start_test.go:301,310` are left on purpose, as the review's fix directed, since they pin the child's command line.
- **Doc text:** `internal/machinecli/doc.go:33` still says `--no-prime-claim` in prose. That is a description, not a code literal. The review did not name it, and the rubric says a fix that only repeats a prior finding is not a finding.

Tool calls:
1. Ran one grep for all three spellings across `.go` and `.md` files. My context held only the earlier grep, which covered `.go` files only and was taken before the edits.
