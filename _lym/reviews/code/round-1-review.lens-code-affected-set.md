```yaml
lens: code-affected-set
outcome: reported
```

**List: per change, the affected set found, covered or not**

- **`shell.Shell` gained `ExitSuccess`, `DiscardOutput`, `CommandFlag` and `ChangeDirToWord`.**
  - A search for `ChangeDir` and `OrElse` methods outside `internal/shell` found no other implementer.
  - Covered.
- **Status bar went from two lines to three, `status` from 2 to 3, and `status-format[2]` was added.**
  - The `bar.go`, `windowsize.go`, `doc.go` (lines 111, 118, 711) and `follow.go` doc lines all match the new shape.
  - The remaining `status-format[1]` and `status 2` hits are either correct or in `docs/research` snapshots, which are historical.
  - Covered.
- **The decision-bearing option set grew to include `@lym_button_spec` and `@lym_button_dir`.**
  - `bar.go`, `follow.go`, `paneoptions.go` and `doc.go` (lines 101, 409) are updated.
  - No other doc claims `@lym_follow` is the only such option.
  - Covered.
- **`@lym_strand_name` is set wherever `@strand` is set.**
  - `paneoptions.go:33` is the only production setter of `@strand`.
  - Covered.
- **New verb `lym reed button`.**
  - It is in the `PersistentPreRunE` skip list in `reedcli/cli.go`, registered in `AddCommand`, and documented in `reedcli/doc.go`.
  - It carries `SkipStencilSeedAnnotation`, and `stencilseedgate_test.go` is updated.
  - Mentions of `reed follow` appear only in files the diff already touched or in `follow.go` itself, so no stencil or md file lists the reed verbs.
  - Covered.
- **New `buttons` key in reed and desktop config.**
  - It is classified in both `ConfigSpec`s, `OpenMaps` is added in `configreg.go`, and both templates (posix and windows) ship the default set.
  - `docs/overview.md` is updated for `desktophook`.
  - Covered.
- **Tmux feature floor.**
  - `toolfloor/doc.go` now lists `#{m:}` and `confirm-before -p`.
  - `run-shell -b -c` is not listed, though the decision record's Constraints section names it among the constructs used.
  - See finding 1.
- **PATTERN.md and `pattern/`.**
  - The diff touches neither, and the directive declares no PATTERN changes, so nothing is owed.
  - `PATTERN-shell-guard-table` (quoting through `internal/shell`) and `PATTERN-spawn-observability` (the spawn is logged in `runButtonScript`) are satisfied in the code I read.
  - Covered.
- **Commit messages.**
  - None carries a `Pattern-Departure:` trailer, and no difference from the decision record needs one in my set.
  - Covered.

**Findings**

1. **LOW, class `consistency`: `toolfloor/doc.go` omits `run-shell -b -c`.**
   - The doc's list of tmux features found is incomplete: the diff adds the `-c` start-directory flag of `run-shell` (`bindings.go` line 97 and `panebin.go`'s start-directory design) but does not add it to the list.
   - The decision record's Constraints section names it among the constructs used.
   - Evidence: `internal/toolfloor/doc.go:26` as changed; `bindings.go` diff, `run-shell -b "#{E:…}"`.
   - Uncertain: the pinned line may carry `-c` elsewhere, in a file the manifest does not show. The diff I read at `bindings.go:97` shows `run-shell -b` without `-c`. The `cd` is done in the shell line via `ChangeDirToWord`, not by `run-shell -c`. If so, D7's `run-shell -b -c '#{@lym_button_dir}'` is implemented differently, which is the handler's judgment to make. The floor doc is then accurate and this finding should be rejected.

**Tool calls**

1. Bash: grep for stale two-line and `status 2` text, other `Shell` implementers, `@lym_follow` "only decision-bearing" claims, and the PATTERN/CLAUDE diff stat. My context lacked a repo-wide search.
2. Bash: grep for other `@strand` setters, for `reed follow` mentions in docs, stencils and yaml, and for verb skip lists. My context lacked a repo-wide search.
