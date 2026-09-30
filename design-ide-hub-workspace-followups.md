Follow-ups from the PR #301 review (`85902381e`), all non-blocking:
- `internal/fabricengine/hubworkspace.go` builds the prime path by hand (`filepath.Join(l.HubPath, primeName, l.AnchorRel)`) instead of `WorktreePath`.
- `internal/ideengine/spawn.go` now WARNs on every spawn whose prime resolution fails, task spawns included (silent before).
- Removing the last pair prunes `_portals/<anchor>`, so the hub workspace shows a missing folder until the next prime spawn.
- Relative-path settings spliced from the prime into the workspace file resolve against `_launchers/`, not the prime.