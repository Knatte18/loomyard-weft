```yaml
lens: fix-affected-set
outcome: reported
```

Findings from the affected-set lens: F1 and F4 are covered, one LOW on F5.

**LOW — F5, `internal/loomcli/overlay.go:30`:** the set of terminal-writer checks still has a member that was not unified.
- Evidence: `internal/loomcli/overlay.go:30` decides terminal-ness with `info.Mode()&os.ModeCharDevice != 0`.
- `internal/configcli/configcli.go:435` has a separate `term.IsTerminal` check.
- The review named only the `shedverbs` duplicate, and the fix covers that one: `human.go:18` is now `clihelp.WriterIsTerminal`.
- The loomcli check is a second output-terminal rule with the old looser mechanism, and `/dev/null` counts as a terminal under it.
- Whether these two decide output rendering was not checked, because that needs reading both call sites.
- `tools/sandbox/suite.go:165` has a similar check but sits outside the CLI packages.

**F1 — covered.**
- Every note-key site in `summary.go` passes through `isNoteKey`: the `list` branch at line 133 and the `notes` branch at line 191.
- `item` (line 177) and `plainFields` (line 254) also use it.
- The new `list` branch sits in the plain-list path and does not touch an object list.
- A plain list under a note key nested at depth 2 or more never reaches `list`; `field` sends it to `notes` first. Both paths print one line per value.

**F4 — covered.**
- `docs/overview.md` and `tools/sandbox/SANDBOX-CREEL-SUITE.md` also mention `--envelope`.
- I did not read them to see whether they agree with the new pattern text.

**F2 and F3 — unchecked.** Their sets are test rows. The searches I ran do not check them, and their boundary rows are visible in the context.

Tool calls:
1. grep for terminal checks across the repo, to find every site of the set F5's premise belongs to.
2. grep for `summaryListLimit`, `isNoteKey` and `noteLines` use, plus the `.md` files mentioning `--envelope`, to cover the F1 and F4 sets.
