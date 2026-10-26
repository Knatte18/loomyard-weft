```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change the diff makes, I looked for callers, readers, docs, stencils and fixtures it leaves stale.

**Per change, the set I found:**
- **Removed verbs** (`lyx orch start`, `lyx orch cleanup`, `lyx prime`, `lyx beam promote`, `adopt --prime`, `orch-template-adopt.md`, `EndRunForPromote`, `ClearRunForPromote`): covered. A repo-wide grep outside `_lyx`, `.lyx`, `.scratch` and test files finds no way forward, refusal, stencil, skill or doc that still names them. The only hits are `docs/research/stencil-engine-audit-2026-10-24/report.md:222,223,339`, a dated audit report that records one run and is not a way forward.
- **`lyx prime` shortcut, `lyx s` added**: covered. `internal/shortcutcli/shortcuts.yaml`, `internal/shortcutcli/doc.go:1` and `docs/overview.md:236,430` all agree.
- **Task `owner_machine` removal**: covered. The `boardcli` help text at `internal/boardcli/cli.go:505,703,768` says a task reads its effective owner and only a beam project records one. The remaining `owner_machine` hits are test fixtures that deliberately write one on a beam project.
- **Orch start → `BeamUp`**: not covered, one finding. Stale comments remain (finding below). The way-forward strings I saw, such as `hibernateengine/wake.go:59,103` and `orchcli/start.go:174,192,330,468`, all name `lyx beam start`.
- **`PATTERN.md` and `pattern/`**: the diff touches `PATTERN.md`, `PATTERN-owner-machine.md` and `PATTERN-run-process-owner.md`. I did not see the diff content, so I judge nothing there.

**Findings:**

1. **LOW, consistency**, `internal/orchengine/launch.go:6`: the comment "StartAction is the branch an orch start takes" still describes a start the diff removed. The code is now `orchcli.BeamUp`. Evidence: `launch.go:6` and `internal/orchcli/start.go:88`, where the bring-up is `BeamUp`.
2. **LOW, consistency**, `internal/orchcli/start.go:64`: the doc comment on `BeamUpReport.Action` says "what the orch start did to the strand", but the verb is now `lyx beam start`. This is vocabulary drift only and is easy to miss.
3. **NIT, consistency**, `internal/orchcli/start.go:275`: the `reconcileHub` doc says "whichever orch starts it", wording left from the removed `lyx orch start`. The reconcile is now run by `lyx beam start`.

No other member of the affected set is left wrong from what my context shows.

**Tool calls (3 of 6):**
1. `grep` for removed verb names across non-test files, to find stale callers, docs and stencils; the manifest does not cover repo-wide text.
2. `grep` for `orch start`, `owner_machine`, `ResetRecordsToParent` and `"prime"`, to find leftover task-owner readers and stale verb references.
3. `cat shortcuts.yaml` and a `lyx p` grep, to confirm the shortcut table and its docs agree.
