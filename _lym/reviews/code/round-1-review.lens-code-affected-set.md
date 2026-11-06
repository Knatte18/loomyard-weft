```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set. What else the diff's changes force outside the lines they touch.**

**Per change, the set found**
- **`RenderRoleFile(path)` loses `index`:**
  - Callers are `internal/orchcli/start.go:600`, `internal/orchengine/watcher.go:185`, `internal/orchengine/resumecontext.go:27` and the tests in `internal/orchengine/prompt_test.go`.
  - All are updated. **Covered.**
- **`NewWatcher` loses `index`:**
  - Callers are `internal/orchcli/watch.go:126`, `watcher_test.go:247` and `watchloop_test.go:254,283`.
  - All are updated. **Covered.**
- **`ResumeContext(p, stencilsDir, now)`:**
  - The callers are `internal/orchcli/resumecontext.go:58` and the two calls in `resumecontext_test.go`.
  - All are updated. **Covered.**
- **`BeamUpOptions.Index` removed:**
  - Callers are `internal/beamcli/cli.go:478`, `internal/loomcli/pawlwiring.go:120`, `internal/orchcli/start.go:116`, `refusal_integration_test.go:47`, `start_test.go:210` and `smoke_cycle_test.go:54`.
  - None sets `Index`. **Covered.**
- **`command_index` marker, `indexRunner`, `runHelpIndex`, `watcherIndexSource` and `indexWayForward` removed:**
  - A grep of `.go`, `.md` and `.yaml` outside `_lym` and `.lym` finds no remaining reference. **Covered.**
  - `Strand.LymBin` stays and is still used elsewhere, for example `reedengine/spawn.go:121`. **Covered.**
- **On-demand files rendered beside `RolePath`:**
  - The stencil registry and marker tests read them through `fixedStencils` (`registry_test.go:23-27`).
  - `PATTERN-stencil-ownership.md:23` is updated.
  - The `PATTERN.md` one-line entry is unchanged and still true.
  - **Covered**, except the findings below.
- **Presence line `prompt.go:29` ("the orch role's Slack rules"):** it now points at `role-slack.md`, which the role's file list sends the session to while the presence is `slack`. **Covered.**
- **Declared PATTERN changes:** none are declared in my context, so there is nothing to check.
- **Commit message and departures:** one commit, with no `Pattern-Departure:` trailer and no difference that needs one.

**Findings**

1. **LOW, consistency.** `docs/overview.md:353` says `lym help index` renders the operator index "which the orch's role file and the `ly:cli` skill carry". The role file no longer carries it; it only names `lym help index`. Fix: drop "the orch's role file and" from that sentence, or say the role points at the index.

2. **LOW, consistency.** `internal/orchroute/paths.go:31` declares `RolePath`. `internal/orchcli/paths.go:1` says every file orch writes is joined once in `internal/orchroute` and handed to `orchengine` as a told path. `internal/orchengine/prompt.go:73-76` instead builds each on-demand path from `filepath.Dir(path)` and the `OnDemand` map key. The files are therefore not in `Paths` and not in the `# The .lym/orch/ layout` list in `internal/orchengine/doc.go:341-355`. That list already omits `role.md`, so this is a minor gap, but the "declared once" claim no longer holds. Fix: either add an on-demand directory field to `orchroute.Paths` and have `RenderRoleFile` use it, or add one line for the `role-*.md` files to the layout list and soften the "declared once" wording.

3. **LOW, consistency.** `internal/loomcli/doc.go:204` says the verb is "a role verb, so the orch's command index leaves it out". The index is now only what `lym help index` prints, and the sentence stays true in meaning. I found no behaviour change, so I raise no finding. It is noted for the handler only.

**Tool calls**
1. A combined grep for `help index`, `RolePath`, section-name references to the role, and callers of `NewWatcher`, `ResumeContext` and `BeamUpOptions`. My context held no search of the doc and caller sets.
2. A grep for `clihelp` use in `beamcli/cli.go` and for `.lym/orch` or `role.md` references. I needed to confirm the import is still used and to find the path declarations.
3. A read of `internal/orchengine/doc.go` lines 338-367, for the `.lym/orch/` layout list, which my context lacked.
