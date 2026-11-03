```yaml
lens: fix-code-correctness
outcome: reported
```

No findings that block; one MEDIUM on coverage of the told path.

Lens fix-code-correctness: check the correctness of the F1/F2/F3 change set.

- **MEDIUM — `Dirs.Weft` is filled by `fabricengine.ConfigDirs` only, so other codex launch paths get no writable root.**
  - Changed text: `internal/fabricengine/junctionnames.go` `ConfigDirs`, the `weft := RecordsWorktree(l)` block, and `codexengine.go`'s `if c.dirs.Weft != ""`.
  - Evidence: the grep in the parent's context shows `shuttleengines.New` called with hand-built Dirs at `internal/burlercli/wiring.go:211` (`configengine.Dirs{Anchor: res.StateDir}`), and with `configDirs` in `internal/loomcli/decision.go:1149`, `internal/loomcli/looparm.go:118`, `internal/pairteardown/cancel.go:242` and `internal/battencli/cancelcmd.go:448`.
  - Run with the burler wiring: `Prepare` sees `c.dirs.Weft == ""`, so the launch line carries no `writable_roots`, and a Codex seat started that way cannot write through `.lym`.
  - The review's fix said to fill the path in `shuttleengines.New` from the caller's hub geometry. Doing it in `ConfigDirs` covers only callers that go through it.
  - Whether the burler and loom-decision paths launch Codex seats is not visible from my context.
- **No findings on the rest of the change.**
  - `ConfigDirs` with an existing weft directory sets `Weft`, and with an absent weft or a stat failure it leaves it empty. `TestConfigDirs` pins both.
  - Existing Dirs literals are keyed, or are `{Anchor: …}`, so adding the field breaks no caller. The grep found no positional literal and no struct equality on Dirs.
  - `TestPrepare_Launch` `WeftTold` fails if `Prepare` ignores `Dirs.Weft`: the `DeepEqual` expects `writableRoots: [weft]` and the `Contains` check expects the setting.
  - `WeftNotTold` fails if `Prepare` adds a root without being told one.
  - `ReadiesHome` `PlainClone` runs `FindRoot` on a `.git` directory, where `GitDir == CommonDir`, so `mainRoot` is empty and only the clone is trusted. It fails if a main root is invented.
  - `RefusesBeforeWriting` `CommittedRulesUnderSubfolderAnchor` anchors at `wt/sub`, which differs from the root, so `checkCodexFolders` appends the anchor's `.codex` and refuses. It fails without that branch. The home stays unmade because the refusal comes before `prepareHome`.

Tool calls: one `grep` pair (positional `Dirs{…}` literals and Dirs equality comparisons). My context lacked the other packages' call sites of the changed `Dirs` type. The `lym gate test` run covered only five packages.
