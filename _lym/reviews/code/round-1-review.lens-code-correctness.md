```yaml
lens: code-correctness
outcome: reported
```

Task: lens code-correctness, running each changed function or branch on a concrete case.

**List**
- `RenderRoleFile`, normal case (path `.lym/orch/role.md`): the five on-demand files are written to `filepath.Dir(path)`, each filled with a nil values map, then the role is written with only `decision_note_title`. Correct. The on-demand files carry no `{{` markers and `stencil.Fill` strips the leading comment, so `writeFilled` returns no unfilled-marker error.
- `RenderRoleFile`, role path under a regular file (the new `breakRoleRender`): the first on-demand `writeFilled` fails in `MkdirAll` and returns `orch: create dir for <file>`. The role is not written, so a stale role from an earlier render stays on disk. The watcher wraps the error as `role file failed to render: …` and goes idle, and `ResumeContext` returns the error and writes no mark. Correct. The map iteration order is random, but nothing depends on it.
- `RenderRoleFile`, concurrent renders (watcher plus `resume-context`): each write is an atomic temp-file-and-rename, so no reader sees a partial file. Correct.
- `ResumeContext`, resuming phase with a pending pointer: it returns the pointer verbatim and renders nothing, so the on-demand files come from the earlier render that stored the pointer. Correct. Every other phase renders the role and its files before the pointer. Correct.
- `ResumeContext`, missing state: the `found` check is unchanged and it still returns an error with no mark. Correct.
- `Watcher.renderRoleFile`: with the index source gone, the only failure is the render. The reason keeps the `role file failed to render:` prefix, which the updated tests assert. `startAutoReload` still forgets the boundary and goes idle. Correct.
- `NewWatcher`: the `index` field and parameter are removed, and the callers in `watch.go` and `watchloop_test.go` match the new signature. Correct.
- `startOrch` and `beamUp`: the index step is gone, so `RenderRoleFile(c.paths.RolePath)` is called straight before `launchOrch`, and a failure still returns before launch. Correct.
- `BeamUpOptions.Index` removal: `beamcli.start` and `smokeBeamUp` no longer pass it, and a grep shows no remaining reader of `command_index`, `indexRunner`, `runHelpIndex`, `watcherIndexSource` or `newIndexSource`. Correct.
- `resumeContextCmd`: it no longer calls `RenderIndex` on `cmd.Root()`. The engine lookup and hook-context steps are unchanged. Correct.
- `orchrole.OnDemand`: the five `//go:embed` vars are initialised before the map literal. The keys match the file names the role cites in backticks, which the new test checks. Correct.
- `fixedStencilTexts` (test helper): it builds keys `orch-role-beams` and so on, with no collision with registry names. Correct.

**Findings**: none.

The one behaviour change is that a bring-up or context cycle no longer fails when the `lym help index` source is unreadable. The summary names it, and the goal's "behaviour does not change" is the handler's to judge, outside this lens.

**Tool calls**
1. `grep -n "func writeFilled"` in `internal/orchengine/prompt.go` and `Fill` in `internal/stencil`: my context lacked their bodies, which I needed to check the nil-values fill and the error paths.
