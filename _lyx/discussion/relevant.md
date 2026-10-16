# Relevant files

- `contracts/stencils/orch/orch-template-role.md` — the role stencil every decision rewrites
- `internal/orchengine/prompt.go` `RenderRoleFile`, `renderFile`, `render` — renders the role file and refuses a stencil without its marker
- `internal/orchengine/watcher.go` `NewWatcher`, `Watcher.renderRoleFile` — the watcher's role-file render and its abort wording
- `internal/orchengine/resumecontext.go` `ResumeContext` — the role-file render after a compaction
- `internal/orchengine/state.go` `Paths` — the told paths the engine receives
- `internal/orchcli/paths.go` `orchPaths`, `PrimePaths` — builds the told paths from the resolved location
- `internal/orchcli/start.go` — `start`'s role-file render before launch
- `internal/orchengine/doc.go` — documents the role file, its marker and the render points
- `internal/orchengine/prompt_test.go` — existing role-render tests to extend
- `internal/lyxcwd/lyxcwd.go` `Location.AnchorPath`, `Location.WorktreePath` — anchor versus worktree root for the file's location
- `contracts/stencils/bouncer/bouncer-template-escalation.md` — the brief the orch's fork reads; gains the verdict and a new settle condition
- `internal/battenshed/doc.go` — already describes the Open-Window row cut from `CLAUDE.md`
- `contracts/stencils/loom/loom-template-arbiter.md` — holds the arbiter's per-cause rule the role defers to
- `contracts/stencils/shed/shed-template-driver.md` — relays the parent notice after the arbiter hands on, refuses, times out or returns a malformed verdict
- `internal/arbiterstop/doc.go` — a `budget` escalation is never arbiter-eligible
- `CLAUDE.md` — the orch-only sections that move and the notes section that gains the pointer
- `cmd/lyx/testdata/test-budget.yaml` — orch package test budget rows
