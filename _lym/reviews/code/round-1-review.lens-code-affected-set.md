```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set.** For each change the diff makes, the set of callers, readers, tests and docs it reaches, and whether the diff covers each.

- **`Remove`'s second look and `StrayPath` (`remove.go`):**
  - Callers and readers found: `fabriccli/fabric.go:1163` emits `stray_path`; `battencli/cancel.go:670` is the new reader. `pairteardown.Result.Removal` is a `fabricengine.RemoveResult`, so `StrayPath` passes through unchanged.
  - Docs: `RemoveResult.StrayPath`, `removePair` and `fabricengine/doc.go:612` are updated.
  - Tests: covered.
  - Pattern fit: `removeWorktreeLeftover` stays in `destroy.go`, so the `PATTERN-fabric-git` and `PATTERN-pair-teardown` rules hold.
- **`PairWorktreeStands` replacing the `os.Stat` in boardcli `PairExists`:**
  - The only production `PairExists` wiring is `boardcli/cli.go:148`.
  - Docs: `boardcli/doc.go` and `fabricengine/doc.go` are updated. `boardengine/config.go:51` and `boardengine/doc.go:24` word the seam generically, so they stay correct.
  - Tests: covered. `TestCLI/Projects` was moved to `git worktree add`. The boardengine tests (`rename_test`, `project_test`) inject their own `PairExists`, so they are unaffected.
- **`ErrTaskPathTaken.Error` wording:**
  - The old text appears nowhere else. I grepped for "remove the directory and retry" and the old phrasing and found nothing.
  - `battencli/readoptstop.go:155` words its own way forward and does not quote the error text.
  - Tests: the one assertion in `add_branch_exists_test.go` is updated. Covered.
- **`createRefusal` mapping `ErrTaskPathTaken` to `PreconditionError`:**
  - There is a single caller, `wire.go:986`. The doc comment and the test row are updated.
  - The readopt path already handles the same error and is untouched. Covered.
- **`Prune`'s third pass:**
  - The only consumer is `fabriccli/fabric.go:941`, which passes `Entries` through as JSON. Exit code is unaffected, and `cleanupremove_integration_test.go:139` already pins that an `Unowned` entry still exits 0.
  - Existing prune tests: none assert entry counts. I grepped `len(...Entries)` in the prune tests, `livestate_verbs_test.go` and the CLI scenario and found no hits. The new pass also skips the prime and beam directories, because they are registered, and `*-weft` directories, because they fail the slug check. I did not run the existing prune tests, so none of them is confirmed to ignore the new `Unowned` entries.
  - Docs: the `prune.go` header, `PruneEntry.Unowned`, the `Prune` doc comment and `doc.go:614` are updated. Covered.
- **PATTERN and commit rules:**
  - No PATTERN declarations apply to this diff, and none are owed.
  - No `Pattern-Departure:` trailer is present and none is needed.
  - Commit messages match the diff.

**No findings.** The affected set is covered.

Tool calls (5 of 6):
1. Grepped for stale wording and `Unowned`/`PairExists` consumers. My context did not hold the repo-wide search.
2. Grepped prune consumers, `doc.go` mentions and the pattern files, to check docs and the PATTERN fit.
3. Grepped `StrayPath` in `pairteardown` and `battencli`, then read `fabric.go:935-1000`, to confirm the CLI output shape.
4. Grepped `RemovePair` and `pairteardown.Result`, to confirm `StrayPath` reaches cancel.
5. Grepped the prune tests, to find which ones call `Prune`. A final grep for entry-count assertions found none.
