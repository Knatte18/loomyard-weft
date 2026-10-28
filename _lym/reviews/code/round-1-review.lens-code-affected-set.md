```yaml
lens: code-affected-set
outcome: reported
```

Task restated: I'm the code-affected-set lens. For each change the diff makes, I list what calls, reads or asserts it.

**Per-change set (all covered, no member left wrong):**

1. **`acquireBranchLock` gained a `verb` parameter** (`internal/fabriccli/anchordeps.go:459`). The only callers are the anchor seam at `anchordeps.go:235`, which passes `"anchor"`, and the hub pull at `internal/fabriccli/hubpull.go`, which passes `"pull"`. Covered.
2. **`anchorLockOwnerRunID` was removed, and the run id became `"fabric-" + verb`.** No reference to the constant remains. The anchor verb's run id is still `fabric-anchor`, so its behavior is unchanged. A repo-wide grep found no test, doc or fixture asserting `fabric-anchor` as a run id. Covered.
3. **The wait notice and the release Warn text now carry the verb.** The old strings `lym fabric anchor: waiting for the parent lock` and `the parent lock of an anchor change is not released` appear nowhere else in `internal/`, `docs/`, `PATTERN.md`, `pattern/` or `contracts/`. The `fabric pull` doc text in `internal/fabriccli/doc.go` matches the new wording. Covered.
4. **New exported `Bolt.Pull`.** The only caller is `pullBoardSide` in `internal/fabricengine/hubpull.go`. The `orchcli` fakes implement only `PullThenCommitWritten`, so no interface needs a new method. Covered.
5. **New `PullHub`, `PairLock` and the `HubPull*` types.** The only consumer is `runHubPull` in `internal/fabriccli/hubpull.go`. The docs describe them in `internal/fabricengine/doc.go` and `internal/fabriccli/doc.go`, and both match the code. The fabricengine doc's description of an unreadable origin record as a stopped beam side matches `localBeamCandidates`. Covered.
6. **New `--hub` flag and help text on `fabric pull`** (`internal/fabriccli/weft_verbs.go`). The `fabric pull` docs in `docs/` and `README.md` contain no pull verb listing that needs the flag, and `docs/overview.md` is not affected because no module row changed, as the decision record says. The test seam `RunHubPullForTest` in `internal/fabriccli/export_test.go` is used by `internal/fabriccli/hubpull_integration_test.go`. Covered.
7. **Merged `internal/fabricengine/doc.go`.** The parent's workspace-file lines (1133-1134) are intact, and `# The hub pull` sits under its own heading at line 1223 with no duplicated or dropped line. Covered.

No findings.

Tool calls:
1. A grep across `internal`, `docs`, `PATTERN.md`, `pattern` and `contracts` for the removed and changed `acquireBranchLock` identifiers and strings. My context did not show every caller or asserting test.
2. `git diff` of `internal/fabriccli/export_test.go`, plus a grep for `PullHub`/`HubPull` users and for `fabric pull` in docs. I needed to see the test seam and any doc references.
3. A grep for `Pull` in `internal/orchcli/rules.go` and `rules_test.go`, to check that the board interface fakes are not forced to change.
