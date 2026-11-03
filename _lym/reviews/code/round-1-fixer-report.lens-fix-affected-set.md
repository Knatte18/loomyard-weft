```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I reran the set searches for F1, F2 and F4/F5. The one gap is uncovered optional sites, so the lens has one LOW finding.

**Searches**
- **F1** (`PruneUnknown` sites): the non-test callers of `ReconcileAll` and `ReconcileHubWideAt` are `hubwide.go:187/206/215` and `configcli.go:560`. The hubwide lines are the default-layer path, which the fixer's loom and landing rows cover. `configcli.go:560` is the hub worktree-file path, which the new `reconcile --apply` subtest on `codeWorktreePath` covers. No operator-side site is missed.
- **F2** (automatic callers):
  - `walk.go:211` is covered by the `newerHubWideKey` assertion, and `walk.go:285` by the new `logger.yaml` assertion on the pair.
  - `fabric.go:748` is covered by the new `ReconcileKeepsAKeyTheTemplateLacks` step.
  - `clone.go:58` and `clone.go:122` have no test. The review called a clone row optional.
- **F4/F5** (doc wording): no stale statement remains in `PATTERN.md:21`, `pattern/PATTERN-config-layers.md:50`, `hubreconcile/doc.go:25` or the `ReconcileAll` comment. `configcli/doc.go:84` says reconcile prunes keys the template lacks. That is correct, since it describes `lym config reconcile`. `configcli/doc.go:62` concerns a different verb's `preserved` field, so I left it unchecked.

**Findings**
- **LOW** — `clone.go:58` and `clone.go:122` are automatic callers with `PruneUnknown` unset, and no test pins them. Adding `PruneUnknown: true` to either would fail nothing. The review marked this optional, so it is a coverage gap, not a defect in the fix.

**Tool calls**
1. One `grep` over non-test Go files for `ReconcileAll`, `ReconcileHubWideAt` and `ReconcileHubWide` callers, to rerun F1 and F2's site sets. My context had only the sites the reviewer and fixer named.
2. The same call also ran one `grep` over `.md` and `doc.go` files for the unknown-key wording, to rerun F4 and F5's set. My context had only the files the review named.
