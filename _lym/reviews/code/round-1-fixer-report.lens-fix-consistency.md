```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix-consistency lens over the change set, meaning twins and cross-references that the F1, F3, F4, F5 and F6 edits should have updated and might have missed.

No findings.

- **Removed identifiers:** `shellArgumentPattern`, `goWordPattern`, `goSubcommandPattern`, `goArgumentsPattern` and `ghMutationScanPattern` have no remaining reference in non-test Go or markdown outside `.lym`, `_lym` and `docs/research`. `ghWordClass` is kept and still used through `shellScope.ghWordChars`.
- **`bash -c` mentions:** the other mentions are in the webster, fabric and planparser scanners and in `contracts/specs/loom-plan-checks-spec.md`. They describe other tools' blind spots, not the shell guard rows, so they stay true.
- **`claudeengine/doc.go`:** the edited pass lists (raw go, refuse-raw, shell-edit, computed-rm, headless claude) now match the row comments in `rows.go`. Lines 681 to 688 also match: `bash -c` is read one level deep, and only a nested `bash -c` hides commands.
- **Twins:**
  - The heredoc-line branch is added to both script reaches, to both script branches of `redirectWritePattern`, and to every scoped row, since all rows go through `commandRowPattern`.
  - The single-quoted-script expansion alternative has no double-quoted twin. The `rawGoCommandPattern` comment states that gap, which is the escape F5's fix allowed.
- **Tests and golden:** the new rows cover each behaviour named in F1, F4, F5 and F6 plus the added gh and sleep rows. The golden regeneration matches the pattern-building change, and the tier-0 gates over claudeengine, codexengine, configcli and orchcli passed.

Outside the change set, not a finding: `claudeengine/doc.go` still says the raw go row's `-tags` check names `tmux` or `llm`, while the code names `tier2` and `tier3`. The fix did not touch or reach that line.

Tool calls made:
1. `grep` across Go, markdown and yaml for the removed identifiers and `bash -c`, because my context did not show other files that cite them.
