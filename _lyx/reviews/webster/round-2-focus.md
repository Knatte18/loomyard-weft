---
round: 2
focus:
  - "internal/loomcli/darnwiring.go `wireDarn`: settle whether `c.env.VerifyCommand` returns the hub's whole `verify` command for a tree-scoped diff, or the per-call scoped command from `darnVerifySource`, and whether `BaseCommand` in `internal/loomshed/verifygate.go` therefore matches `pattern/PATTERN-verified-tree.md`."
  - "internal/darnengine/config.go `TreeScopedCommand`: settle whether a path lying under nested tree keys, such as `docs` and `docs/api`, credits every tree it lies in, so each tree's target reaches `{targets}`, and whether a `TestTreeScopedCommand` row pins it."
  - "internal/impactset/guards.go `ModuleGuardTests`: settle whether a listed package directory outside the module directory is returned as an error rather than skipped, and whether guard-name deduplication in `buildCommand` and the module-root `.` guard spelling in `internal/gatecli/run.go` each have a test row that fails when the behavior is removed."
  - "internal/loomshed/verifygate.go `NewVerifyGate` doc and the opening key list of internal/darnengine/doc.go: settle whether each states the current `treeScope` parameter, the hub's whole verify command, and the `tree_scope` and `tree_scope_verify` keys."
---

Each entry names a site and the question to settle; none pre-states an outcome.
