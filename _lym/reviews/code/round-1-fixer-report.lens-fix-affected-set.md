```yaml
lens: fix-affected-set
outcome: reported
```

Task: the fix-affected-set lens, rechecking the sets behind F1, F2 and F9.

**Unchecked findings:** F3, F4, F5, F6, F7 and F8 have no set premise. I ran no search for them.

**F1 (callers of `FollowTemplate`):** no findings. The only non-test caller is `internal/configsync/configsync.go:1046`, and the new `ErrInvalidBase` handling covers it.

**F2 (sites that commit or record the template base):** no findings. Every non-test site that uses `ChangedFilesRel()` or `WroteTemplateBase` is covered by a test:
- `internal/configcli/hubwide.go:224` is covered by the `--layer default --apply` assertion.
- `internal/fabriccli/fabric.go:719` is covered by the reconcile test.
- `internal/fabriccli/clone.go:66` is covered by the clone test.
- `internal/hubreconcile/walk.go:229` was already asserted by the walk test.

**F9 (docs saying a present file is never filled):** one MEDIUM and one LOW, below. The fix covered `hubwide.go:163` and `doc.go:92`, which are the two sites the review named. It left two others unchanged.

- **MEDIUM:** `internal/configsync/configsync.go:459`, the `ReconcileAll` doc. It says "in a hub it is sparse: a present file is migrated and pruned … nothing is added". That is correct for a per-worktree file, which `ReconcileAll` handles, so I'm only flagging it as LOW below. The real MEDIUM is `internal/hubreconcile/doc.go:47-48`.
  - It says "Every write to [a base file], by any package, holds only the units the hub tuned and never fills in a template value (PATTERN-sparse-hub-base); only this walk's one-time strip drops a unit the template holds".
  - A seed-only file such as `profiles.yaml` now gains template units and loses template-dropped units on every walk, so this sentence contradicts the fix. The docs pass was meant to remove exactly this claim.
  - The paragraph above it (lines 43-44) and the "base file" definition at line 46 don't exempt seed-only modules.
- **LOW:** `docs/overview.md:378` reads "prunes but never fills a present one, carries a template change into a seed-only board file …". It is internally consistent, but "a present one" is unscoped to non-seed-only files. Because it sits beside the seed-only clause, I consider it tolerable.
- **Not a finding:** the `configsync.go:5` and `configsync.go:941` wordings are already scoped to non-seed-only modules.

**Tool calls (3 greps, 1 combined read):**
1. Grepped non-test callers of `FollowTemplate(`. My context only held the one call site the review named.
2. Grepped non-test uses of `ChangedFilesRel()`, `WroteTemplateBase` and `ReconcileHubWideAt(`. My context held only `hubwide.go`, `clone.go` and `fabric.go`, not the walk or other sites.
3. Grepped the repo for "never fills" wording in Go, markdown and YAML files. This tests whether the F9 fix covered every site.
4. Read `internal/hubreconcile/doc.go:43-50`, grepped `docs/*.md` for `FollowTemplate` and `ErrInvalidBase`, and read `internal/configsync/configsync.go:455-460`. I needed the context of the stale-wording hits from call 3.
