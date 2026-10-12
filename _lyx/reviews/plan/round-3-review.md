---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/16-sparse-migration.md (Intent, the walk's integration-test paragraph)
    summary: card 16's three Ensure integration assertions presuppose unmarked worktrees holding full files, but every hubforge fixture worktree arrives already marked and with no per-worktree file, so the stated assertions cannot hold
    fix: >-
      In card 16, add `internal/hubreconcile_test#newStaleHub` to `Edit:` and state in its Intent that the fixture, after CopyHub and AddPair, writes and commits a full pre-migration per-worktree file set (each module's template, plus one key holding a value that differs from the repo default) in the prime and in the pair, and deletes and commits `configengine.SparseMarkerRel()` in each, so the walk under test meets unmarked worktrees with something to strip. Have `TestEnsure_MergeBeginningBeforeCommitRestoresTheFiles` build its hub through that same fixture, so its no-marker-after-restore assertion starts from an unmarked pair.
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/16-sparse-migration.md (Edit list); _lyx/plan/15-configsync-layer-reconcile.md (CloneAndWire and hubforge paragraphs)
    summary: once card 16 lands, the clone's own Ensure walk commits the sparse marker in the prime, yet no card brings the hubforge seeding contract, CloneAndWire's mutation record or the clean-hub scenario's record-shape step in line with what a clone now produces
    fix: >-
      In card 16, add `internal/hubforge/doc.go`, `internal/fabriccli#CloneAndWire` and `internal/fabriccli_test#TestRunCLI_CleanHubScenario` to `Edit:`.
      The Intent then says that a clone's prime arrives with the sparse marker committed by the clone's walk, that CloneAndWire appends a `KindFileWritten` entry for the marker when it is absent before the `Ensure` call and present after it, and that the scenario asserts the committed marker.
      In card 15's Intent, say how the `CloneConfigCommit_MutationRecordShape` step of `TestRunCLI_CleanHubScenario` changes, namely that it no longer expects one prime-side `KindFileWritten` per per-worktree module followed by a weft commit, and that it expects one entry per seeded repo default.
---

## Focus verdicts

1. Seeding before card 9 and inside `Ensure`.
   No defect before card 15: no card before 15 seeds a per-worktree repo default.
   The loaders that cards 4–8 move to `fabricengine.ConfigDirs` all degrade through `LoadOrTemplate`, so an absent default falls back to the template as the base, with the worktree file stacked above it.
   Card 9 keeps loom's callers on `Dirs{Anchor: ...}`, and card 15 switches them in the same card that seeds the defaults.
   The seed stays inside `hubreconcile.Ensure`:
   card 15 gates the per-worktree seed on `HubWideOptions.SeedDefaults`, which only `reconcileHubWide` sets.
   `CloneAndWire` reaches the seed only by calling `Ensure`.
   `hubreconcile` imports nothing from `fabriccli` (`internal/hubreconcile/seam_enforcement_test.go`), so the new call adds no import cycle.
   Card 15 does edit `internal/hubforge/doc.go`.
   Card 16 then changes the clone's outcome again without following through; see F2.
2. Card 2's way forward for an absent per-worktree default.
   Not a defect.
   `lyx loom resume` calls `hubreconcile.Ensure` before `armAt` loads any module config (`internal/loomcli/arm.go:150-156`).
   Card 15 makes `Ensure` treat an absent per-worktree default as a stale stamp, and the resulting walk passes `SeedDefaults: true`, so the named step does create the file.
   The way forward rightly names neither `lyx fabric reconcile` nor `ReconcileAll`, since card 15 has both leave an absent default absent.
   The new `TestLoad_Refusals` row pins the start verb and the hub reconcile, which matches the text card 2 prescribes.
3. Where layers are stacked.
   Not a defect.
   Card 7's batcher reads `StackLayers`'s `Origins` and never picks the winning layer itself.
   Card 8's burler validates each `Layers` entry and decodes the stack's `Doc`.
   modelspec lays the stack's `Doc` over its built-ins, which replaces whole entries against a non-file base, not across layers.
   No card merges or orders layer files outside `configengine`.
   configsync, configcli and hubreconcile read layer files for reconcile, print and snapshot, never to resolve a value, and form their paths through `configengine`.
4. Test economy across cards 4, 5, 8, 9, 10 and 13.
   Not a defect.
   Each local-layer key named in the acceptance criteria is pinned once.
   Cards 4, 5, 8, 9 and 10 pin `reed.shell`, `shuttle.claude_deny_python`, `burler.lenses`, `loom.parent_review_wait_min`, `darn.verify` and `landing.publish_verify` through the real loaders.
   Card 13's table pins `darn.review_max_bounces`, `landing.require_pr_to_base`, `reed.mouse`, `loom.discussion_producer`, `loom.plan_fan` and the shuttle open-map entry.
   No key appears on both sides.
   Card 14's `TestPrepareRunSettings` reuses `darn.review_max_bounces` and the open-map entry, but it pins the separate `run_settings` surface that the acceptance criteria name apart from the local file.
5. Card 16's failed walk.
   The snapshot half is sound.
   Card 16 has `snapshotConfig` record the marker as nil when it is absent, and `restoreConfig` already removes every nil entry (`internal/hubreconcile/walk.go:383-397`).
   The card's assertion still cannot hold, because its fixture pair is marked before the walk starts; see F1.

### [BLOCKING] Card 16's walk assertions run on worktrees that are already migrated

**Location:** `_lyx/plan/16-sparse-migration.md`, the paragraph naming `TestEnsure_StaleBuildReconcilesAndCommitsEveryWorktree`, `TestEnsure_MidMergePairIsLeftAloneAndRetried` and `TestEnsure_MergeBeginningBeforeCommitRestoresTheFiles`.

**Issue:** Card 15 makes `CloneAndWire` end by calling `hubreconcile.Ensure`.
That walk runs `reconcileHubWide` first, which seeds every per-worktree default, and then runs `reconcileWorktree` on the prime (`internal/hubreconcile/walk.go:105-113`).
Card 16 has `reconcileWorktree` pass `Migrate: true`, and it gates the strip only on every per-worktree default existing, which that same walk has just ensured.
So the clone's walk writes and commits `_lyx/config/sparse` in the prime.
hubforge builds every template hub through `CloneAndWire`, and `AddPair` forks the pair from the prime, so by D6 the pair inherits the marker.
After card 15 the prime also holds no per-worktree config file, since `ReconcileAll` no longer materialises absent files in a hub, so there is nothing to strip.
Card 15's `cloneHub` removes only the build stamp.
All three tests start from `newStaleHub` (`internal/hubreconcile/ensure_integration_test.go:146-150`, which is `CopyHub` plus `AddPair`) or from the same template hub.
On that fixture none of card 16's assertions can hold:
- "each worktree is stripped and committed with its marker": every worktree is already marked, and a marked worktree is never stripped.
- "the skipped pair is stripped and marked by the retry": the pair is already marked.
- "has no marker on disk after the restore": the marker is committed, so the snapshot records its bytes and the restore puts it back.
These tests are what pins D6's resumability acceptance criteria ("a walk interrupted … is completed by the next walk"), so the card cannot be implemented green as written.

### [MEDIUM] Clone outcome changed by card 16 is not reflected in hubforge's contract, the clone's record or the clean-hub scenario

**Location:** `_lyx/plan/16-sparse-migration.md`, `Edit:`; `_lyx/plan/15-configsync-layer-reconcile.md`, the `CloneAndWire` and hubforge paragraphs.

**Issue:** Card 15 rewrites `internal/hubforge/doc.go`'s seeding contract to say a clone's prime carries "only the per-worktree config files the records repository already held".
Card 15 also has `CloneAndWire` record a `KindFileWritten` entry only for the seeded defaults.
From card 16 on, the same `Ensure` call also strips the prime and writes and commits the sparse marker there.
No card after 15 edits the hubforge contract, `CloneAndWire`'s record or `TestRunCLI_CleanHubScenario`, so all three describe a clone that card 16 no longer produces.
Card 15 also leaves the existing step `CloneConfigCommit_MutationRecordShape` unaddressed (`internal/fabriccli/cleanhub_integration_test.go:142-170`).
That step asserts one prime-side `KindFileWritten` per per-worktree module followed by a weft commit.
Card 15 lists the scenario under `Edit:`, but its Intent names only the new board-default assertion.
