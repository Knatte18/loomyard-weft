```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that the fix covers every member of each set the findings rest on (fix-affected-set lens).

Findings left unchecked: none.

No findings.

- **F1** (sites and tests tied to the old error): the only caller of `ClientAgentPane` outside the tests is `internal/orchcli/remotecontrol.go:195`. My search of `internal`, `cmd`, `docs`, `contracts` and the root `*.md` files found no remaining consumer or test of the `no client %q is attached` text for a short row. The rc-on verb test's fake tmux prints a four-field row, so it does not reach the new branch.
- **F2** (prefix match): `/dev/pts/30` now comes before `/dev/pts/3` in `OtherClientsRowsAreSkipped`, so a prefix match would return `%9` and fail the row.
- **F3** (CR trim): there is one trim site. `AgentPaneWithoutTheMark` now ends in `\r\n`, so removing the trim would read the mark as on and fail the row.

Tool calls:
1. Repository-wide grep for the error texts and `ClientAgentPane`: my context only had an `internal` Go search. The output was too large to use, so call 3 narrowed it.
2. Grep for `list-clients` in test files: my context did not show which other fakes feed `list-clients` rows.
3. Read part of `internal/orchcli/remotecontrol_test.go` and ran the narrowed grep: my context had neither the verb test's fake row nor the consumers of the error text outside Go code.
