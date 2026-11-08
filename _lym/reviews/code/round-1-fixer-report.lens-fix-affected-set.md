```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

- F1: the terminal-read refusal at `cli.go:241` was the only error built from a `readHiddenLine` error.
  The other "at the prompt" way-forwards (`cli.go:246`, `cli.go:256`) are not reached from that path.
  Line 246 is reached only after a prompt printed, so it stays accurate.
  Line 256 is the piped, non-terminal path and tells the operator to run from a terminal, so it stays accurate.
- F2: `readHiddenTerminalLine` has one production site (`wiring.go:45`) and one test, the new `wiring_test.go`.
  The seam in `cli_test.go:204` is a fake, so no other real implementation needs a test.
  Both non-terminal branches (`!ok` and not a terminal) have a row.

Tool calls:
1. grep for "at the prompt", `readHiddenLine` and `readHiddenTerminalLine` across `.go` and `.md` files outside `_lym` and `.lym`. My context held only the two files the fixer changed, not the other sites.
