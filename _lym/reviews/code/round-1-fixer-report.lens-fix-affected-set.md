```yaml
lens: fix-affected-set
outcome: reported
```

Findings, one per set checked (all LOW):

1. **LOW** — `internal/fabricengine/hubworkspace.go:51`, the `RegenerateHubWorkspace` doc comment. The F1 set is every place that describes the workspace settings splice as prime-only. The fix changed only `internal/fabricengine/doc.go`. This comment still says it rewrites "a workspace file per beam from … the prime's .vscode/settings.json". That is stale for a beam file, which splices the beam's own settings when it has them. The same grep shows `internal/vscode/config.go:15` (the `BuildWorkspace` limitation note, "splices the prime's settings verbatim into the hub workspace file"). That one concerns the hub file, so it is not wrong. It does not cover the beam files, whose relative-path caveat is the same.
2. **LOW** — `internal/fabricengine/hubworkspace.go:56-57`, the same doc comment. It says "unreadable prime settings … returns an error". F3 added a new degradation: an unreadable beam settings file is now logged at Warn and falls back to the prime's, and the comment does not say so. `beamWorkspaceSettings` still documents an error return that the caller no longer propagates; I did not verify its rewritten wording.

Sets checked with no gap:
- **F4:** the `owner_machine` mentions that guide the upsert payload are `internal/boardcli/cli.go:479` (the Use line), `internal/boardcli/cli.go:704` and `internal/boardcli/cli.go:769` (the `get` help), `internal/boardengine/doc.go:118`, and `internal/boardengine/doc.go:109`. The doc.go lines already say "`beam_color` is no upsert key". The Use line and the other help text do not list `beam_color`, which fits its being output-only.
- **F2:** `beam_color` appears in docs only in `docs/overview.md`. The only code sites are the `replayFields` row, `normalizeTask`, `migrateEntries`, the owner setter and the readers, and they are now all covered by tests.
- **F5, F6, F7:** each finding names a single site, so there is no wider set to search.

Unchecked: nothing.

Tool calls (2 of 6, one `Bash` call each):
1. A grep of `.go`, `.md` and `.yaml` for `owner_machine` mentions that guide upsert/accepted keys, for F4; my context lacked a repo-wide list.
2. A grep for the prime-only settings-splice wording and for `beam_color` in docs, for F1 and F2; my context lacked the other sites that describe the splice.
