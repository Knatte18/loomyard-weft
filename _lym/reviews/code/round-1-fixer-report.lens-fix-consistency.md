```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix change set through the fix-consistency lens.

Findings:

1. **LOW** — `internal/fabricengine/destroy_landedwork_integration_test.go:1-4`, the file header.
   - The header says the file "drives the pair-warp ownership kind and the unlanded-work dirtiness kind".
   - F1 added `TestOwnedUntrackedTrackedAt_RefusesATargetTheIndexTracksOrTheCommitDoesNot` to this file. That test drives a third kind, the untracked-tracked-at ownership.
   - A reader choosing a file by its header would not find it here.
   - The header should name the third kind.

2. **LOW** — `internal/fabricengine/droplocal_integration_test.go:23-26`, the `TestDropLocal` doc comment.
   - It lists the branch move, the removal of an untracked file origin tracks, and the dirty-path refusal.
   - It does not mention the new subtest. That subtest checks that an ignored file, a file origin does not track, and a code branch file are not removed.
   - The doc comment should include it.

3. **LOW** — `internal/adoptcli/deps_test.go:1-2`, the file header.
   - The header lists the hub-only pause reading, the check start's guard, the alive skip, the liveness reading and the gate tiers.
   - F2 added `TestReconcileOptionsWaitOnAPushLock`, which covers the reconcile's push-lock bound. The header omits it.

Checked and consistent:
- The new `doc.go` Step done sentence agrees with the Resume bullet near line 172.
- No other text matched "resumed under either", "standing record is resumed" or "Only the local-commit flag". A grep over `.go` and `.md` files outside `_lym` found nothing.
- The `overview.md` sentences agree with the `doc.go` and `worker.go` behaviour.
- `deps.go` has the new `reconcileOptions` helper, and the seam now calls it.

Tool calls:
1. Ran `grep` for the stale "resumed under either" phrases across the repo, to see whether a twin of the `doc.go` edit was left unchanged. My context held only the files the fixer read.
