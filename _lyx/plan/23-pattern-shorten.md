# Card 23 — pattern-shorten

**Edit:**
- `PATTERN.md`

**Intent:**
Shorten nineteen `PATTERN.md` entries, as the parent's `changes` decisions of 2026-10-24 record, so the overview stays under its base size once the task's four declared lines land (`pattern-headroom`).
Each entry's line becomes the line below, verbatim, keeping its `(test)` marker and background link where it had them; no entry's rule changes, only its wording, and any detail a line drops already sits in its background file or its code.

```
- `PATTERN-command-row` — Adding side-action row to a recipe: a `Command` row over config keys, its command from template and board default only.
- `PATTERN-shed-recipe-registry` — Adding producer: one `map[string]Constructor` in `internal/shedrecipe` behind `Lookup`, bar `Implementer`'s.
- `PATTERN-trimpath-safe-root` — Locating module root in gate-built code: never from a `runtime.Caller` path alone.
- `PATTERN-role-skills-typed` — Loading spawned session's skill: spawner names it on launch spec and lyx delivers it; no stencil asks. (test)
- `PATTERN-template-overlay` — Judging config document against its template: only `configengine`'s effective template.
- `PATTERN-shell-guard-table` — Guarding shell command: only in `internal/shellguard`, every engine rendering every row.
- `PATTERN-lyxdirs-single-declarer` — Naming `_lyx` or `.lyx` in a path: only `internal/lyxdirs`, bar a `tools/` script's variable block.
- `PATTERN-treadle-runner-seam` — Importing into `internal/treadleengine`: allowlist only, never `burlerengine`, `lyxcwd` or a CLI package.
- `PATTERN-binary-change-reconcile` — Reconciling a hub's config after a binary change: only via `hubreconcile.Ensure`.
- `PATTERN-shed-producer-seam` — Importing into `internal/shedengine`: only stdlib, `state` and `lock`; paths are told.
- `PATTERN-parent-lock` — Moving, merging into or landing on a local parent branch: only in Finalize, under its parent lock.
- `PATTERN-agent-filed-issues` — Filing GitHub issue: only `lyx selfreport create` in production. (test)
- `PATTERN-fabric-write-containment` — Writing `_launchers` or workspace file in `fabricengine`: via a hub `os.Root` only.
- `PATTERN-origin-reconcile` — Acting on a run or hub after an absence: `fabricengine`'s origin reconcile first.
- `PATTERN-never-force-add` — Keeping transients unindexed: `.git/info/exclude`; never `git add -f`.
- `PATTERN-recipe-owns-failures` — A merge or test failure at Merge-In or Finalize: its fixer, a reroute or a halt to the orch.
- `PATTERN-hub-store-gc` — Hub stores run no auto gc: only `fabricengine/housekeeping.go` sets it or runs `gc`. — [background](pattern/PATTERN-hub-store-gc.md)
- `PATTERN-owner-machine` — Starting, resuming or landing a run or acting on wake: only where no other machine owns it. — [background](pattern/PATTERN-owner-machine.md)
- `PATTERN-pane-binary-resolution` — Starting `lyx` in a reed pane or window: resolve it via `panebin.go` only. (test) — [background](pattern/PATTERN-pane-binary-resolution.md)
```

**ImpactSummary:** Rewords nineteen overview lines with no rule change; no code reads them.
