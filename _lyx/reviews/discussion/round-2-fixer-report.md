# Fixer report: discussion, round 2

All four findings were checked against the code and fixed.

## Fixed

- **F1 (MEDIUM, design): the hook's cwd drifts.**
  Confirmed: the prime orch's events file records 20 hook payloads with `cwd` set to `gate-test-scope`, and `orchcli` resolves through `lyxcwd`, with no directory flag on any orch verb.
  D11 now says `orchcli` builds the hook command through `internal/shell` as a change of directory to the prime's absolute path, chained with the verb.
  The Auto-mode assumption now says the hook inherits the session's environment, not that it runs in the session's cwd.
  `PATTERN-shell-mechanics-seam` is added to Constraints, and there is a new acceptance criterion for a session whose cwd is in another worktree.
- **F2 (MEDIUM, design): the role file is stale under a draft.**
  Confirmed: `reloadAfterCompaction` (`internal/orchengine/watcher.go:961`) and `startAutoReload` (`:738`) are the only places that render `role.md`, and both run only after the gate passes.
  D11 now has the verb render the role file through `RenderRoleFile` before it renders the pointer, in every phase but `resuming`.
  `renderFile` writes with plain `os.WriteFile`, and the verb and the watcher may write at the same moment, so D11 now requires `RenderRoleFile` to write through a temporary file and a rename.
  The verb's acceptance criterion covers the role render.
- **F3 (LOW, design): the hook's binary is unstated.**
  D11 now says the bare `lyx` resolves to the spawning binary.
  The hook inherits the pane's environment, and the `panebin.go` prelude puts that binary's directory first on `PATH` and exports `LYX_BIN`.
  The finding said a bare `lyx` reaches the production binary on `PATH`, which the prelude contradicts; that part of its premise is wrong.
  Stating the resolution was still missing, so it is fixed, not disputed.
- **F4 (LOW, design): D4's wake has no bound.**
  D4 now has a Bound covering:
  - who writes the signal file;
  - the cost of each event: one stat and a remove, with no spawn;
  - that an event finding no file runs nothing, including the event the loop's own remove raises;
  - that an apply runs only when the debounce fires, and retries only up to its cap.

  The resize-loop acceptance criterion now covers the event that finds no file.

The support log has a "Burler round 2" section and Question ledger entries Q18 and Q19.

## Disputed

None.

## Deferred

None.
