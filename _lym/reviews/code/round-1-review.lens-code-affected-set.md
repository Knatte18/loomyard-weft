```yaml
lens: code-affected-set
outcome: reported
```

**Sets found, per change:**
- **Renames** (`TidyShuttle`→`AgentShuttle`, `runnerTidyShuttle`→`runnerAgentShuttle`, `resolveTidyModel`→`resolveModel`), plus the removed `landNoticePullRequest` and `pullRequestFollowUp`: no stale reference is left. `internal/beamcli/pr.go:300 pullRequestFollowUp` is an unrelated function of the same name. `loop.go:141` names the field `TidyShuttle`, which keeps that name. Covered.
- **`RecordDescription` signature**: its callers in `deps.go`, `describe.go` and `land_test.go`, plus `deps_integration_test.go`, are all updated. Covered.
- **`rememberRuns`/`landNoticeLine` signatures and the dropped follow-up**: `landnotice.go` callers, `landnotice_test.go`, `dobbyengine/doc.go`, `role-beams.md` and the `Deps.PullRequestRecorded` doc are all updated. `role-autonomy.md:56` (write the description before `pr open`) is unaffected. Covered.
- **Describer stencil and role**: `stencils.go` entries and `roleOpeningStencils` are added. The shuttleparity producer is added. `agentconfig.RoleDescriber` already exists. `loom-final-summary-spec.md` exists. Covered.
- **New state keys** (`landed_since_describe`, `describe`): `state.go` and `dobbyengine/doc.go` describe them. `docs/research/...state-map` is a dated snapshot and exempt. Covered.
- **New describer agent in flight, against the existing worktree-gone exit that stops a dobby's agents**: `loop.go:505-551` (`readKeptTidy`, `rememberKeptTidy`, `exitWorktreeGone`) stops only the tidy agent. Not covered; see finding 1.
- **Dobby's spawned agents in `docs/overview.md`**: line 837 is updated, but the module listing at line 294 is not. See finding 2.
- **Commit trailers**: the range has no `Pattern-Departure:`, `Loosened-Check:` or `Root-Cause:` trailer. The rows removed from `TestLandNotices` pin the follow-up behavior that the answer key removes, so they are not a loosened check. The other edited existing tests change only a signature. Covered.
- **PATTERN rules**:
  - `config-layers`: met; landing.yaml is read through `landingshed.LoadConfig`, which calls `configengine.Load`.
  - `github-auth`: no breach; the dobby only updates a pull request, and the rule restricts open, reopen, merge and close.
  - `engine-parity`: met; a producer is added.
  - `stencil-ownership`: met; the parent and edit directives are rendered.
  - `lym-files`: the drafts directory is named through the dobby geometry, as `TidyDirName` already is.
  
  The directive declares no PATTERN changes.

**Findings:**

1. **MEDIUM · design** · `internal/dobbyengine/loop.go:533` (`exitWorktreeGone`) and `internal/dobbyengine/describe.go` (`spawnDescribe`)
   When a beam's worktree is removed, the dobby exits and stops only the board-tidy agent (`d.keptTidy`). A describer agent in flight (`State.Describe`) is not kept, read or stopped.
   - The describer's run is waited on by a goroutine inside the dobby process (`dobbydeps.go`, `runnerAgentShuttle.Start`), and that goroutine dies when the dobby exits. So the agent's pane is left running against a removed worktree, with nothing to stop it.
   - The tidy path guards exactly this case: `loop.go:320`, `505-551`, and `Deps.PrimeTidyShuttle`.
   - Fix: keep `Describe` beside `keptTidy`, and stop its strand through `PrimeTidyShuttle` in `exitWorktreeGone`.
   - Evidence: `internal/dobbyengine/loop.go:505-551`; `describe.go` `DescribeState`.

2. **LOW · consistency** · `docs/overview.md:294`
   The module listing says `internal/dobbyengine` "spawns the board-tidy agent". The diff makes dobby spawn the describer agent too, and leaves this line naming only the tidy agent.
   - Evidence: `docs/overview.md:294`; diff 004 updates only line 837 onward.

Tool calls:
- Grep for leftover old names and `pullRequestFollowUp`, `board-tidy` registrations and follow-up wording: the context lacked repo-wide references beyond the diff.
- Grep for every file that names the board-tidy role versus the describer role and stencil, and for the final-summary spec's existence: needed to find registrations the describer might lack.
- Grep of `loop.go` for `PrimeTidyShuttle` and kept-tidy use, and of `overview.md`/`state.go` for tidy mentions: the worktree-gone path was outside the manifest.
- Read of `loop.go:495-560`, the `exitWorktreeGone` and kept-tidy bodies: needed to confirm that the describer is not stopped.
