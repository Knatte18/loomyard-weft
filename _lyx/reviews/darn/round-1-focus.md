---
round: 1
focus:
  - "`lyx config reconcile --layer default --apply`: check the prime-only and no-slug-strand guard, the board write lock, the all-or-nothing dry run before writing, and the commit and push, against the guard a `--set --layer default` write uses. Does a failure leave any file written?"
  - "The refusals (reconcile from the board dir, `--layer` other than `default`): do they fire before any write, and does each name the way forward? Does the dry run stay unguarded?"
  - "`configsync.ReconcileHubWideAt` with `SeedDefaults: true` seeded from the prime's anchor: does it cover the hub-wide modules' files and each per-worktree module's repo default, and could it retire or strip files that should stay?"
  - "The `configengine` load warning: is it emitted once per file and key list, and does it leave out open-map entries and the spec's `RetiredKeys`? Does the `sync.Map` dedupe hold under concurrent loads, and does the warning leave the load itself unchanged on both `Load` and `LoadOrTemplate`?"
  - "The board entry's operator decision to remove the `preserved` old keys: does the diff and the new verb remove them from any hub still holding them, and is the claim that no hub file is written borne out by the diff?"
  - "Claims in `_lyx/landing/summary.md` (envelope with `layer` and `seed`, commit message, test rows) against the diff."
  - "Docs landing in the same commit: `doc.go` of the touched packages, `docs/overview.md` if the CLI surface changed, and `PATTERN.md` for any new invariant."
  - "Test economy: do the new `TestLoad_LayerStack` rows and the `TestConfigOverRealHub` scenario duplicate existing coverage, and is any added behavior left uncovered?"
---

Seed focus from the board entry and the change description; no round has been judged yet.
