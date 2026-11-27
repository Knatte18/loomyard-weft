```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. I checked what the diff's changes force outside the lines it touches.

**Per change, the set found and whether it is covered:**

1. **Finalize lock ordering: merge and gate unlocked, lock only to land.**
   - `contracts/orchrole/role-landing-halts.md`: covered.
   - `docs/overview.md`: covered.
   - `internal/landingshed/doc.go`: covered.
   - `finalize.go` file header, `Call` doc and step comments: covered.
   - `finalize.go:61` `Finalize` type doc: **not covered**.
   - `finalize.go:454-456` step 1c comment and `deps.go:261-262` `ConfigChanges` doc: **not covered**.
   - `finalize.go:540-544` `freeLock` doc: partly covered.
   - `contracts/recipes/{sampler,darn,tapestry,hardener}-recipe.yaml` Finalize comments: **not covered**.
   - `contracts/orchrole/orch-role.md:196-199`: still true, so covered.
   - `contracts/specs/landing-fix-review-spec.md`: no lock claim, so covered.
   - `docs/research/**`: measurement and report documents, exempt.
2. **Pre-push check exemption at Finalize, and `PATTERN-owner-machine`.**
   - `internal/loomcli/prepush.go:89,108` skips the check at Finalize "which judges origin and its owners itself". The diff now runs Finalize's commits, the unlocked merge and fixer, before its owner checks.
   - This is a behaviour finding, so it is the correctness and holistic lenses' call. I flag only the stale contract below.
3. **New key `landing.finalize_lost_races`.**
   - `config.go` field, `ConfigSpec` category, `LoadConfig` check, `keynames.go` constant, `template.yaml`, `config_test.go` row, `docs/overview.md`: covered.
   - Default-layer files: an unknown key only warns (`configengine/config.go:323-330`), so an older binary does not break. The summary's "two-way" call holds for the config.
4. **`mergeTip` moved to the shared `parentMergeTip(ctx, deps, guard, producer)`.**
   - The callers are Parent-Align (`parentalign.go:147`) and `gateUnlocked`. Both are updated.
   - `parentGuard` is passed by value, which is safe: it holds no mutable state (`parentguard.go:16-22`).
5. **`mergeParent(ctx, tip)`, `judgeParentMerge(..., tip)`, and `keepParentHead` moved into `mergeAndGate`.**
   - All callers are in `finalize.go` and updated.
   - The `mergeParent` doc moved the `keepParentHead` sentence to `mergeAndGate`: covered.
6. **`freeLock` keyed on `lockHeld`.** The only callers are `runFixer` and `judgeFixerChange`: covered.
7. **Commit messages.** There are no `Pattern-Departure:`, `Loosened-Check:` or `Root-Cause:` trailers, and none is needed: the change is an enhancement and loosens no assertion. No declared PATTERN changes are in the directive.

**Findings**

- **MEDIUM, consistency.** `contracts/recipes/darn-recipe.yaml:16,305`, `contracts/recipes/sampler-recipe.yaml:18,261`, `contracts/recipes/tapestry-recipe.yaml:52,528` and `contracts/recipes/hardener-recipe.yaml:176`.
  - The Finalize row comments still say "Finalize takes the parent lock first, merges the parent into the task branch, runs the landing gate and the merge fixer". The darn and sampler headers also say "Finalize merges the parent itself, under the parent lock it takes first".
  - After the diff, the merge, gate and fixer run with no lock held, and the lock is taken only to land.
  - Evidence: the grep output of those lines, against `finalize.go:302-326`.
- **LOW, consistency.** `internal/landingshed/finalize.go:61-62`.
  - The `Finalize` type doc still reads "the ShedProducer that merges the parent branch into the task worktree under the parent lock". The merge now runs unlocked first (`finalize.go:334-357`).
- **LOW, consistency.** `internal/landingshed/finalize.go:454-456` and `internal/landingshed/deps.go:261-262`.
  - Step 1c says `ConfigChanges` is read "so the diff is taken against the parent as the task forked from it". The `deps.go` doc says it is called "before the merge-in there, so before any parent-side mutation".
  - In the default path, `gateUnlocked` has already merged the parent tip into the task by then, the weft side included. `ReadConfigChanges` takes `merge-base(parentTip, tip)` (`fabricengine/configchanges.go:76-80`), so the fork point it reports is now the merged parent tip. Before the diff, a recipe with no Parent-Align read the original fork point.
  - The comments no longer describe what is diffed. The handler should judge whether this changes the notice's base values.
- **LOW, consistency.** `internal/landingshed/finalize.go:540-544`.
  - The `freeLock` doc says that after a free "the run takes the lock again at the front of the queue and runs its locked pass again". After the diff, `Call` runs the lock-free pass again first, while races remain (`finalize.go:310-316`).
- **LOW, consistency.** `internal/loomcli/prepush.go:89`.
  - The `skipsCheck` doc exempts Finalize "which judges origin and its owners itself". It now commits merge and fixer work for the whole unlocked pass before judging its owners (`finalize.go:312-316` runs before `ownerChecks` at `:418`).
  - The diff leaves this contract claim wrong. I left the behavioural severity to the correctness and holistic lenses.

**Tool calls**

1. grep for `finalize_fix_review_rounds` listings and for stencil and spec lock prose: to find docs that enumerate landing keys or describe Finalize's lock, which the manifest lacked.
2. grep of `contracts` for parent-lock prose: to find contract docs beyond the files the diff touched.
3. Read `orch-role.md:192-202` and the Finalize lines of `landing-fix-review-spec.md`: to judge whether those lock claims went stale.
4. grep of configengine for unknown-key handling: to judge whether the new key breaks older binaries (Merge danger).
5. Read `configengine/config.go:315-335` and `doc.go:68-82`: to confirm the default layer warns on an unknown key rather than refusing it.
