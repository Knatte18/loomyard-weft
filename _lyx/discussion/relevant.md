# Relevant files

- `ORCHESTRATOR.md` — the file this task deletes; its two rules move to the hub-wide file
- `internal/orchcli/paths.go` `orchPaths`, `PrimePaths` — joins every orch path, including `RepoRulesPath` today
- `internal/orchengine/prompt.go` `RenderRoleFile`, `readRepositoryRules` — renders the role file and reads the rules file
- `internal/orchengine/state.go` `Paths` — carries `RepoRulesPath`, replaced by two rule paths
- `internal/orchengine/resumecontext.go` — passes the rules path to `RenderRoleFile`
- `internal/orchengine/watcher.go` — renders the role file at every delivery and names `ORCHESTRATOR.md` in an abort reason
- `internal/orchcli/start.go` — passes the rules path to `RenderRoleFile` at launch
- `internal/orchcli/refusal.go` `refuseNonPrime` — the prime-only refusal the new verb runs under
- `internal/orchcli/doc.go` — documents the `## This repository` section
- `internal/orchengine/doc.go` — documents `Paths.RepoRulesPath`
- `contracts/stencils/orch/orch-template-role.md` — the role stencil's closing section, marker and two placement sentences
- `internal/configcli/hubwide.go` `newHubCommit`, `setHubWide`, `rollBack`, `hubWriteRefusal` — the Bolt commit, push and restore route the new verb follows
- `internal/fabricengine/junctionnames.go` `BoardDir` — the hub's board dir the hub-wide path is joined under
- `internal/lyxdirs/dirs.go` `LyxDirName` — the `_lyx` name the paths use
- `CLAUDE.md` — its "Notes go in git" rule names `ORCHESTRATOR.md`
