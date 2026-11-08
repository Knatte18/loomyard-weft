```yaml
lens: fix-affected-set
outcome: reported
```

**Fix-affected-set lens: F1 and F2 (set: text that limits the machine override to hub-wide modules). F3 covers a test-row set and I found no gap there.**

Findings:

1. **LOW** — `internal/configcli/configcli.go:674`, the `--layer` flag help text.
   - Summary: The help still says "machine is a hub-wide module's never-tracked override of its machine keys under the board dir's .lym". That is the same stale premise F1 and F2 fixed, and `codex` is a per-worktree module with a machine key.
   - Evidence: the fixed wording at `pattern/PATTERN-lym-files.md:25` and `docs/overview.md:513` drops "hub-wide". `pattern/PATTERN-config-layers.md:44` says "The layer applies only for a module with a `machine` key, hub-wide or per-worktree". `internal/configcli/doc.go:37` says the same ("hub-wide or per-worktree"). The flag text contradicts both.
   - The review named only the two doc lines, so this member of the set was missed.

Checked, no finding:
- `internal/configengine/doc.go:52-60` already covers both module kinds.
- `internal/configcli/doc.go:37` is already correct.
- `PATTERN-config-layers.md:44` and `:48` are already correct.
- `docs/overview.md:513-514` and `:693` agree with each other.
- The remaining "hub-wide" hits are test comments and descriptions of hub-wide modules such as `envconfig` and `gateslot`. They are accurate.
- F3: the new `FullAccessSandboxMode` row sits alongside `prepare_test.go`'s `fullAccess` row and `command_test.go`'s `sandboxFullAccess` row. The `OpenLaunch` path is the only launch builder at `codexengine.go:183` without a full-access test, and the new row covers it. The other launch path at `codexengine.go:143` is covered by the `prepare_test.go` row.

Tool calls:
1. `grep -rniE "hub-wide"` filtered on "machine": reran F1's and F2's search across .md, .go and .yaml files, since the review named only two lines.
2. `grep -rnE "machine\.yaml|machine override|machine layer"` on .md files: a second phrasing of the same set, to catch rewordings that avoid "hub-wide".
3. Read `internal/configengine/doc.go:48-63`: to check whether its "hub-wide module's stack" line was stale; my context lacked that file.
