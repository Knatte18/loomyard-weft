```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, covering what main's imported changes force outside the lines they touch.

**Per change, the set I found and whether it is covered**

1. **`shedverbs.WaitNote` replacing the `string` return of the Waiting hook, and `PrintStatusLinesOnChange` taking `(line, key)`:**
   - Set found:
     - the `reviewstatus.go` hook builders
     - `checklistPoll` and `checklist.go`
     - `arm.go:439`, which passes `reviewWaitingFor`
     - `shedverbs/status.go` and `spec.go`
     - the `status_test.go` callers
     - `contracts/specs/loom-status-spec.md`
     - the `docs/overview.md` status line
   - Covered. The beam adds no other implementer or caller of `Hooks.Waiting`, and `go build` and `go vet` pass on every package in the range.

2. **Pre-push check wired into `Commit`, `Spawn` and `Push` in `wiring.go`:**
   - Set found:
     - `newCommitStatusSeam`, which maps `errPushStopped` to "no commit made"
     - `commitStatusDepsGuardedBy`, `wireLightweight` and `commitStatusDeps`
     - `shedrun.ParentNoticeFile`, which sits under `ScratchDir` and so is ephemeral, as `PATTERN-durable-vs-ephemeral-state` requires
     - `pattern/PATTERN-owner-machine.md` and its `PATTERN.md` line, both updated and consistent
     - `docs/overview.md` and `loomcli/doc.go` (`doc.go` carries a +11 change)
   - Covered. The beam's `DefaultEngine`, `assistBounds` and `Assist` fields are all intact; `git diff c18e033a..HEAD -- wiring.go` shows only main's additions.

3. **New hub-wide `hibernate` config module:**
   - Set found:
     - the `configreg.go` row
     - `configreg_test.go`'s HubWide row and doc comment
     - `cmd/lym/configstrictness_test.go`'s degrading set
     - `pattern/PATTERN-config-layers.md`'s degrading-callers list, which matches `hibernateconfig`'s use of `LoadOrTemplate`
     - `docs/overview.md`'s package-tree row and hibernate section
     - the `orch-role.md` text
   - Covered. A grep for other module-name lists (the `sampler` references) turned up no list that should name `hibernate` and doesn't.
   - The beam's `Dirs.Records` field is read by no layer, so the hibernate load is unaffected.

4. **`ChildRecipeFile`, `ClearMotifRun` and `motif_run_cleared`:**
   - Set found:
     - the `battenRunCommitPaths` pathspec
     - the `wire.go` write after the child seed
     - `beamcli/create.go` and its deps
     - the overview text on `beam create` and `batten cleanup`
   - Covered. None of it touches the beam's changes.

5. **Dobby `drivable` and `ReachedParent`:**
   - Set found:
     - `passes.go`
     - the `orch-role.md` exception under batten pause
     - the dobbyengine tests
   - Covered.

6. **`fabricengine` `finishGitWorktreeRemoval`, and the leftover seam threaded through `removeWeftWorktree` and `removeLocalWeft`:**
   - Set found: the callers in `remove.go` and `weftwiring.go`, and `export_test.go`.
   - Covered. The beam's `ConfigDirs` change in `junctionnames.go` is separate and does not overlap.

7. **Declared PATTERN changes:**
   - The directive declares none, and the range's PATTERN diff is only main's two entries. Nothing is missing.

8. **Commit messages:**
   - Main's nine commits and the merge commit carry no `Pattern-Departure:` trailers. Nothing to judge.

9. **Stencils:**
   - The only contract text in the range is `orch-role.md` and `loom-status-spec.md`, and neither is an engine-block stencil. The summary's claim that main changed no stencil the beam moved into an engine block holds.

**Findings:** none.

**Tool calls (1 of 6):**
- `grep` for `"sampler"` and `sampler.yaml` across `internal`, `cmd`, `docs` and `contracts`, plus a `hibernate` grep in `configsync`, `configcli` and `hubreconcile`. My context had no module-name list other than the `configreg` ones, so I checked whether any other list needs a `hibernate` entry.
