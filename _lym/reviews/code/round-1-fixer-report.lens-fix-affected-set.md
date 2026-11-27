```yaml
lens: fix-affected-set
outcome: reported
```

fix-affected-set check of the change set: did the fix cover every member of each set-shaped finding.

Left unchecked: none of F1, F2, F7, F11, F10 and F20. I did not rerun searches for the coverage-gap findings, which name single cases.

Findings:

1. **LOW**: the F11 help change (`internal/battencli/cleanup.go` `cleanupCmd` Long and `internal/battencli/cli.go` `run` Long) left sibling statements of the same done-run removal condition stale. Each still says the dobby removes the run once its landing is counted and its notice decided, with no watch-line or `beam.run_watch` hold:
   - `internal/beamcli/cli.go:852-855` (`lym beam create` help): "until the parent's dobby removes it once its landing is counted and its notice decided."
   - `docs/overview.md:835`: the same sentence.
   - `docs/overview.md:1755`: "The parent worktree's dobby removes any done run itself, with its task branch, once its landing is counted and its notice decided."

2. **LOW**: F20 added the nested-run bound to `gateslot.Holder.Marker` only. Its twin claim at `internal/gatecli/run.go:806-807`, added by this branch, still says "returns the environment entry that every process the tier run of pid starts carries". A nested `lym gate test` strips and replaces that entry (the `slices.DeleteFunc(... enclosingGateEnv+"=")` line in the same file).

3. **LOW**: F9 made `readNoticeChild` return `PairDir` with `found` false, but the seam doc at `internal/dobbyengine/runnotice.go:109` still says only "found is false while the child has no status file". The run watch now relies on `child.PairDir` being filled in that case, and the contract does not say so. The other two consumers (`runNotice`, `readWait`) return on `!found`, so no behaviour breaks.

Checked and complete:
- **F1:** no added or changed comment, doc or message outside tests' YAML documents and case names still names one of the keys bare. The "run's autonomy" wording in `dobbynoticestamp_integration_test.go` refers to the stamp, not the key. The two new key literals appear only in `keynames.go` and the struct tags.
- **F2:** no other hand-written config write was added to the `beamcli` tests.
- **F7:** every `GateHolder` field and `TaskWorktree` are asserted.

Tool calls (5):
1. Searched the branch diff's added lines for bare key names (F1's own search); my context had only the earlier comment-prefix grep, taken before the fix.
2. Searched the diff's test files for hand-written config writes (F2's search); my context lacked the other test files' hunks.
3. Grepped code and docs for the done-run removal statement (F11's set); my context had only a narrower phrase grep.
4. Read the `docs/overview.md` and `beamcli/cli.go` lines and grepped the marker claims (F11 and F20 twins); my context lacked that text.
5. Read the `ReadChild` seam doc and `runMarkerEntry` with its diff (F9 and F20 siblings); my context lacked both.
