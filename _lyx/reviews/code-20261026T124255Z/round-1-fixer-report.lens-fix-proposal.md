```yaml
lens: fix-proposal
outcome: reported
```

Fork task: challenge the reviewer's four proposals (F1 to F4) as the fixer applied them; no findings.

- **F1** (`pattern/PATTERN-refusal-way-forward.md`): `cmd/lyx/norecoverydeny_test.go:228` asserts `sed -i` is not denied, and `internal/shellguard/rows_integration_test.go:209` and `:210` assert that `sed -i` and `perl -pi` pass. The edit matches both. The closed list now has no form the code still denies, and the computed-path `rm` and `systemctl` clause keeps its pattern-shaped wording.
- **F2** (`internal/shellguard/rows.go:61`): `grep -rn unquotedPrefixPattern internal` returns nothing, so the reworded comment names no deleted symbol.
- **F3** (`internal/editdirective/editdirective_test.go`): The proposal drops the `sed -i` phrase assertion and the "never `sed" absence check, which is the prose pinning the review flagged. The single-statement rule stays covered elsewhere. The restored comment matches the pre-diff wording.
- **F4** (`docs/research/codex-feasibility.md:14`): The row's remaining guardrails (python, raw `go test`, computed `rm`) still exist as rows, so dropping `sed -i` leaves the row correct.

Nothing outside the review's scope broke. The remaining `sed -i` mentions in `contracts/stencils/edit/edit-directive.md:6`, `internal/editdirective/doc.go:7`, `internal/shuttleengine/claudeengine/doc.go:222` and `contracts/stencils/stencils.go:568` describe the permitted scripted edit, not a deny.

Tool calls:
1. `grep` for `sed -i`, `perl -i`, `in-place` and `unquotedPrefixPattern`: to find leftover references to the removed denies.
2. A `sed -n` plus `grep` Bash call on `rows_integration_test.go` and `stencils.go`: the hook refused it, so no output.
3. Read of `rows_integration_test.go:195-215`: to confirm the allowed-everywhere table.
4. Read of `contracts/stencils/stencils.go:555-575`: to check whether `editRulePhrases` ties a `sed -i` string to the changed text.
